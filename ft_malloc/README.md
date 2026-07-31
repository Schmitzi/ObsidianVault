---
tags: [42school, c, malloc, systems-programming, ft_malloc]
---

# ft_malloc — Project Notes

## Overview

Custom memory allocator in C replacing libc `malloc`/`free`/`realloc`, built on `mmap`/`munmap`. Compiles to a shared library (`libft_malloc_$HOSTTYPE.so`), loadable via `LD_PRELOAD` to transparently intercept a program's allocation calls.

**Core design:** three allocation categories based on requested size, each handled differently:

| Category | Size threshold | Zone behavior |
|---|---|---|
| `TINY` | ≤ 128 bytes | Multiple allocs share a zone; blocks split/coalesce within it |
| `SMALL` | ≤ 1024 bytes | Same as TINY, different size class |
| `LARGE` | > 1024 bytes | One `mmap` per allocation, no splitting, zone freed individually |

---

## Data Structures

```c
typedef enum e_type { TINY, SMALL, LARGE } t_type;

typedef struct s_block {
  size_t size;
  int free;
  struct s_block *next;
} t_block;   // 24 bytes

typedef struct s_zone {
  t_type type;
  size_t size;
  t_block *blocks;
  struct s_zone *next;
} t_zone;
```

- Global `t_zone *zones` — singly-linked list, head-insertion on creation.
- Each zone is laid out contiguously in the mmap'd region: `[t_zone header][t_block header][data][t_block header][data]...`
- `t_block.free` uses `ALLOC = 0` / `FREE = 1`.

### Memory layout of one zone

```
mmap'd region:
+----------+----------+--------------------+----------+-----------+
| t_zone   | t_block  |   data (size N)     | t_block  | data ...  |
| header   | header   |                      | header   |           |
+----------+----------+----------------------+----------+-----------+
zone->blocks points here ^
```

---

## Key Design Decisions

- **Separate `t_type` enum from byte thresholds** — `get_type(size)` maps a raw size to a category; `type_max_size(type)` maps back to the category's max size. Kept distinct so zone-sizing math (`calculate_pages`) doesn't have to duplicate threshold logic.
- **Zone sizing via ceiling-division page arithmetic:**
  ```c
  pages = (numerator + pagesize - 1) / pagesize;
  zone_size = pages * pagesize;
  ```
  This pattern showed up **three separate times** as a source of bugs — specifically, computing `pages` correctly via ceiling division, then *forgetting to multiply back by `pagesize`* before passing the byte count to `mmap`. Worth double-checking at every call site that does this arithmetic.
- **`split_block` takes a `t_type` parameter** so it can skip splitting entirely for `LARGE` (single mmap'd block, no internal structure to split).
- **Centralised logic preferred over many small helpers** — e.g. `realloc`'s LARGE-grow case and its general fallback-when-no-room-to-merge case turned out to be *the same code path* (`malloc` + copy + `free`), so no separate LARGE-specific function was needed.

---

## `malloc()`

Walks `zones` for a zone matching `get_type(size)`, then walks that zone's blocks for one that's `free` and large enough. Splits it via `split_block`. If no suitable zone/block exists, creates a new zone and splits its first block.

No major bugs surfaced here during this session — it was already working going in.

---

## `free()`

### LARGE branch — unlink zone from `zones`, then `munmap`

**Bug history (in order encountered):**

1. **Dead-code loop condition.** Original code had `while (curr != zone) { if (curr == zone) {...} }` — the inner check could never be true given the outer loop's condition, so the unlink/munmap never fired. Zone leaked silently.
2. **Use-after-`munmap`.** After fixing the loop, `munmap(zone, zone->size)` was called *before* `prev->next = curr->next` — but `curr == zone`, so reading `curr->next` after `munmap` is a read of already-unmapped memory (UB). **Fix: unlink first, munmap second.**
3. **Head-of-list case.** If the zone being freed is the current head (`prev == NULL`), the global `zones` pointer itself must be reassigned (`zones = curr->next`), not just some `prev->next`. Missing this left `zones` pointing at freed memory.

**Final correct shape:**
```c
while (curr != ptr_zone) {
  prev = curr;
  curr = curr->next;
}
if (prev != NULL)
  prev->next = curr->next;
else
  zones = curr->next;
munmap(ptr_zone, ptr_zone->size);
```

### Non-LARGE branch — mark free, coalesce adjacent free blocks

```c
while (curr) {
  if (curr->free && curr->next && curr->next->free) {
    curr->size += sizeof(t_block) + curr->next->size;
    curr->next = curr->next->next;
    // deliberately do NOT advance curr here
  } else {
    curr = curr->next;
  }
}
```

**Why `curr` isn't advanced after a merge:** after absorbing `curr->next`, the *new* `curr->next` might *also* be free. Not advancing lets the loop re-check the same `curr` against its new neighbour on the next iteration — this is what allows a run of N adjacent free blocks to collapse into one instead of stopping partway.

---

## `realloc()`

### Final design

```
ptr == NULL              → malloc(size)
size == 0                → free(ptr); return NULL
find_zone(ptr) == NULL   → return NULL (treated as UB / invalid input — no recovery attempted)

if block->size >= size:
  → split_block(block, size, zone->type)   // shrink in place, same pointer

else if block->next exists, is FREE, and (block->size + block->next->size + sizeof(t_block)) >= size:
  → merge forward:
      block->size += sizeof(t_block) + block->next->size;
      block->next = block->next->next;
    → split_block(block, size, zone->type)  // trim any excess back to a free block
    // same pointer returned, data never moved

else:
  → new_ptr = malloc(size)
  → ft_memcpy(new_ptr, ptr, block->size)   // copy only the OLD size, not the new size
  → free(ptr)
  → return new_ptr
```

### Bugs hit along the way

1. **NULL-zone "recovery" attempt.** Early draft tried to handle `find_zone(ptr) == NULL` by calling `create_zone` and grafting the unrecognized block pointer into the new zone's `blocks` field. This is actively dangerous: if `ptr` doesn't belong to any zone, it's not a pointer this allocator laid out — treating it as if it were a valid `t_block*` risks reading/writing through a garbage address. **Resolved by treating this case as undefined behavior and just returning NULL** — consistent with the C standard's stance on `realloc` given an invalid pointer.

2. **Merge-forward, wrong operator.** First attempt: `block->size = sizeof(t_block) + block->next->size;` — this *discards* `block`'s own original size instead of adding to it. Needed `+=`, not `=`.

3. **Wrong `t_type` passed to `split_block` post-merge.** Used `get_type(size)` (category of the *requested* size) instead of `zone->type` (the *actual* zone's category). These can differ in edge cases; `split_block` needs to know how the physical zone is laid out, not just what category the new request nominally falls into.

4. **Type confusion on `malloc`'s return value — twice, in two different disguises.**
   - First version: declared the result of `malloc(size)` as `t_block *new_block`, then wrote `new_block->size = size` — but `malloc` returns a pointer *past* the header, to the usable data. This overwrote the first 8 bytes of the caller's actual data, interpreting it as a size_t field. **Fix: declare it as `char *` (or `void *`) and never touch a `->size` field on it directly** — the real header's size is already set internally by `malloc`/`split_block`.
   - Second version (after fixing the type): `ft_memcpy(new_ptr, ptr, sizeof(ptr))` — `sizeof(ptr)` is the size of the *pointer variable* (always 8 on x86-64), not the size of the data it points to. Always copied exactly 8 bytes regardless of actual allocation size. **Fix: use `block->size`** (the old block's real data size) as the copy length — since this branch is only reached when `block->size < size`, copying `block->size` bytes is always safe and never over-reads.

5. **`shrink_case()` dead code.** An early standalone helper function duplicated the shrink logic that was also written inline in `realloc()` itself. Deleted in favor of the inline version to avoid divergence.

### Why LARGE didn't need a separate branch

`LARGE` zones hold exactly one block and never split, so `block->next` is always NULL for a LARGE allocation — meaning the merge-forward branch's guard (`block->next && ...`) naturally fails, and control falls straight through to the generic fallback (`malloc` + `memcpy` + `free`). That fallback *is* the mmap-new/copy-old/free-old behavior LARGE needs — no special-casing required. Shrinking a LARGE block goes through `split_block`'s own `type == LARGE` guard, which just marks it `ALLOC` and returns the same pointer without giving pages back — acceptable per the realloc contract (allowed to return a buffer ≥ requested size).

---

## libft integration

### Makefile changes required

1. **`-fPIC` override on the libft submake**, without editing libft's own Makefile:
   ```makefile
   $(LIBFT):
       @$(MAKE) -C $(LIBFT_DIR) CFLAGS="-Wall -Wextra -Werror -fPIC"
   ```
   Necessary because the final target is a `.so` — every `.o` linked into it, including libft's, must be position-independent code.

2. **Include path** so project `.c` files can `#include "libft.h"`:
   ```makefile
   CCFLAGS = -Wall -Werror -Wextra -fPIC -I$(LIBFT_DIR)
   ```
   Needed to add `#include "libft.h"` to `malloc.h` itself, since none of the `.c` files included it directly — caused an `implicit-function-declaration` error on `ft_memcpy` under `-Werror` until fixed.

3. **Link flags** on the final `.so` target:
   ```makefile
   $(NAME): $(OBJ) $(LIBFT)
       @$(CC) -shared $(OBJ) -o $(NAME) -L$(LIBFT_DIR) -lft -lpthread
   ```

4. **`clean`/`fclean` cascade** into `src/libft` so stale `.o`s don't linger between rebuilds with different flags.

5. A separate `bin:` target for compiling a standalone test executable (`main.c`) hit the same missing-link problem independently: it compiled `main.c` alone with `-I` flags but no `-L$(LIBFT_DIR) -lft`, causing `undefined reference to ft_memset` once `main.c` was switched to call libft functions directly instead of libc equivalents. **General lesson: `-I` (compiler, declarations) and `-L`/`-l` (linker, definitions) are separate concerns — an `implicit declaration` error and an `undefined reference` error point at two different missing flags.**

---

## Testing methodology

### `LD_PRELOAD` basics
```bash
LD_PRELOAD=./libft_malloc.so ./some_program
```
This makes the dynamic linker resolve `malloc`/`free`/`realloc` symbols from our `.so` *before* falling back to glibc — works for both explicit calls and libc-internal calls (e.g. `strdup` calling `malloc` internally).

**Important caveat:** under `LD_PRELOAD`, *everything* that calls `malloc` goes through our allocator — including the dynamic loader's own startup allocations and glibc internals — not just the explicit calls in the test program. This makes raw `mmap`/`munmap` call counts from a syscall trace hard to attribute precisely to specific lines of test code; don't over-interpret exact address/size correlation from `strace` output.

### Stress test coverage
Built a single test program exercising: TINY alloc+write, merge-forward realloc (freeing a neighbour then growing into its space), shrink-in-place realloc, `realloc(NULL, size)`, `realloc(ptr, 0)`, LARGE alloc, LARGE realloc-grow (fallback path), and explicit frees at the end. All paths confirmed correct — pointer identity preserved on in-place operations (shrink/merge), pointer moved (and old data preserved) on the LARGE fallback.

### Leak-checking without valgrind
`strace -e trace=mmap,munmap` on the test binary, filtered to `MAP_ANONYMOUS` (non-`MAP_FIXED`) entries to exclude the dynamic loader's own mappings. Confirms zone mmaps and munmaps roughly balance — though see caveat above re: exact attribution.

**Alternative to valgrind:** AddressSanitizer compiled directly into the test binary:
```bash
cc -fsanitize=address -g -o main src/main.c -Iinc -Isrc/libft -L$(LIBFT_DIR) -lft
LD_PRELOAD=./libft_malloc.so ./main
```
Catches use-after-free, buffer overflows, double-frees without any dependency on system debug-symbol infrastructure. Note: ASan and a custom `mmap`-based allocator both intercepting allocation can interact in surprising ways — worth being aware of if something behaves strangely under it.

---

## Open items / things to revisit before submission

- [ ] `show_alloc_mem()` is still an empty stub — subject likely requires it to print allocation state (zones, blocks, sizes, total).
- [ ] No handling for `mmap` failure inside `create_zone` bubbling up cleanly through `malloc`/`realloc` beyond returning NULL — worth double-checking every caller actually checks for NULL before dereferencing.
- [ ] TINY/SMALL zones are never returned to the OS even when fully free (by design) — confirm this is acceptable for the subject's requirements, or whether "free zone with no allocated blocks" should also trigger a `munmap`.
- [ ] Full `make` with real (non-stubbed) `libft` hasn't been confirmed end-to-end yet — only tested with a minimal reconstructed libft in a sandbox.
- [ ] Consider whether `realloc`'s "shrink" path should also attempt merging with a following free neighbour when shrinking (currently only splits off excess from `block` itself — doesn't look at `block->next` in the shrink branch).
- [ ] Double check thread-safety — `pthread.h` is included and linked (`-lpthread`) but no locking currently exists around `zones` list mutation. Worth deciding if the subject requires thread-safe `malloc`.

## Checklist
- [ ] malloc() allocates `size` bytes of memory and returns a `pointer` to the allocated memory.
- [ ] realloc() tries to change the size of the allocation pointed to by `ptr` to `size`, and returns `ptr`. 
	- [ ] if there is not enough room to enlarge the memory allocation pointed to by `ptr`, realloc() creates a new allocation, copies as much of the old data pointed to by `ptr` as will fit to the new allocation, frees the old allocation, and returns a pointer to the allocated memory.
- [ ] free() deallocates the memory allocation pointed to by `ptr`. If `ptr` is NULL, no operation is performed.
- [ ] If there is an error, the `malloc()` and `realloc()` functions return *NULL*.
- [ ] `mmap()` and `munmap()` must be used to request and return memory zones to the system.
- [ ] Use of glib-c malloc() is *STRICTLY FORBIDDEN*.
- [ ] Pre-allocate a number of zones for small (`TINY`) and medium (`SMALL`).
- [ ] Use `sysconf(_SC_PAGESIZE)` to obtain pages size.
- [ ] Each zone must contain at least 100 allocations.
	- [ ] `TINY` mallocs, from 1 to 𝑛 bytes, will be stored in 𝑁 bytes big zones.
	- [ ] `SMALL`mallocs, from ( 𝑛+1 ) to 𝑚 bytes, will be stored in 
	- [ ] `LARGE` mallocs, from ( m+1 )bytes and above, will be allocated outside the standard memory zones. this means they will be handled using `mmap()`, placing them in their own separate memory zone
	- [ ] Definitions:
		- [ ] n = 1-128
		- [ ] m = (n+1)-1024
		- [ ] m+1 = 1025
		- [ ] 𝑁 = 100 block large
		- [ ] 𝘔 = 100 blocks large