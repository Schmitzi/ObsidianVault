Building a drop-in-place dynamic allocation memory management library

## Bemerkungen

- use the mmap(2) and munmap(2) system calls to request and return memory zones to the system
- must manage own memory allocations for the internal funtioning of the project without the libc malloc() function.
- Limit the number of calls to mmap() and munmap() by pre-allocating memory zones to store "small" and "medium" malloc.
- The size of these zones must be a multiple of sysconf(_SC_PAGESIZE)
- Each zone must contain at least 100 allocations.
	- "TINY" mallocs, from 1 to n bytes, will be stored in N bytes big zones.
	- "SMALL" mallocs, from (n+1) to m bytes, will be stored in M bytes big zones.
	- "LARGE" mallocs, from (m + 1) bytes, will be stored in M bytes big zones. This means they will be handled using`mmap()` , placing them in their own separate memory zone.
- The size of n, m, N and M so that you find a good compromise between speed (saving on system recall) and saving memory.
- Memory must be properly aligned.

## Extras

- Manage the use of ft_malloc in a multi-threaded program. Make it thread safe with the pthread library.
- Added functions such as:
	- Manage malloc debug environment variables. Imitate those from the system malloc or invent some.
	- Implement a `show_mem_alloc_ex()` function that provides extended details, such as a histroy of memory allocations or a hexadecimal dunp of the allocated memory zones.
	- Defragment the freed memory.


## malloc()

### Description

The **malloc**() function allocates _size_ bytes and returns a pointer to the allocated memory. The memory is not initialized.  If _size_ is 0, then **malloc**() returns a unique pointer value that can later be successfully passed to **free**().

### Return value

The **malloc**(), **calloc**(), **realloc**(), and **reallocarray**() functions return a pointer to the allocated memory, which is suitably aligned for any type that fits into the requested size or less. On error, these functions return NULL and set _[errno](https://man7.org/linux/man-pages/man3/errno.3.html)_.  Attempting
 to allocate more than **PTRDIFF_MAX** bytes is considered an error, as an object that large could cause later pointer subtraction to overflow.
### Errors
*ENOMEM*: Out of memory. Possibly, the application hit the RLIMIT_AS or RLIMIT_DATA limit described in getrlimit(2). Another reason could be that the number of mappings created by the caller process exceeded the limit specified by /proc/sys/vm/max_map_count.

## realloc()

### Description

The realloc() function changes the size of the memory block pointed to by p to size bytes.  The contents of the memory will be unchanged in the range from the start of the region up to the minimum of the old and new sizes.  If the new size is larger than
 the old size, the added memory will not be initialized.
 
 If p is NULL, then the call is equivalent to malloc(size), for all values of size.
 
 If size is equal to zero, and p is not NULL, then the call is
 equivalent to free(p) (but see "Nonportable behavior" for
 portability issues).
 
 Unless p is NULL, it must have been returned by an earlier call to
 malloc or related functions.  If the area pointed to was moved, a
 free(p) is done
### Return Value

The realloc**()function return a pointer to the allocated memory, which is suitably aligned for any type that fits into the requested size or less. On error, these functions return NULL and set _[errno](https://man7.org/linux/man-pages/man3/errno.3.html)_.  Attempting to allocate more than **PTRDIFF_MAX** bytes is considered an error, as an object that large could cause later pointer subtraction to overflow.

### Errors
*ENOMEM*: Out of memory. Possibly, the application hit the RLIMIT_AS or RLIMIT_DATA limit described in getrlimit(2). Another reason could be that the number of mappings created by the caller process exceeded the limit specified by /proc/sys/vm/max_map_count.
 

## free()

### Description

The free() function frees the memory space pointed to by p, which must have been returned by a previous call to malloc() or related functions.  Otherwise, or if p has already been freed, undefined behavior occurs.  If p is NULL, no operation is performed.

### Return Value:

The **free**() function returns no value, and preserves _[errno](https://man7.org/linux/man-pages/man3/errno.3.html)_.

### Errors

None.

## show_alloc_mem()
The show_alloc_mem() function allows visualization of the state of the allocated
memory zones.

```sh
TINY : 0xA0000
0xA0020 - 0xA004A : 42 bytes
0xA006A - 0xA00BE : 84 bytes
SMALL : 0xAD000
0xAD020 - 0xADEAD : 3725 bytes
LARGE : 0xB0000
0xB0020 - 0xBBEEF : 48847 bytes
Total : 52698 bytes
```


## mmap()

The mmap() function creates a new mapping in the `virtual address space` of the calling process. The starting address for the new mapping is specified in `addr`. The `length` argument specifies the length of the mapping (which must be greater than 0).

if `addr` is NULL then the kernel chooses the (page-aligned) address at which to create the mapping. If `addr` is not NULL, then the kernel takes it as a hint about where to place the mapping.

A call to mmap() might look like this:

```c
void *raw = mmap(NULL, size, PROT_READ | PROT_WRITE,
                  MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
```

mmap() hands back one flat chunk of `length bytes` which is just raw memory, no structure. We need to impose a structure on it by deciding "the first `sizeof(t_zone)` bytes of this chunk are the zone header, and everything after that is available for blocks"

```c
t_zone *zone = (t_zone *)raw;          // zone header lives at the very start
zone->type = type;
zone->size = zone_byte_size;
zone->next = NULL;

t_block *first_block = (t_block *)((char *)raw + sizeof(t_zone));
// first_block now points right after the zone header, still inside
// the same mmap'd chunk — same memory, just a different offset into it

first_block->size = zone_byte_size - sizeof(t_zone) - sizeof(t_block);
first_block->free = 1;
first_block->next = NULL;
```

And that's really it. One singular mmap() call gives a block of raw bytes and we need to carve it into pieces by casting different offsets of that same pointer to a different struct types. The zone header and the first block aren't two separate allocations that need to "know each other" through magic,they are neighbours in the same contiguous physical memory. To address one from the other, we can simply add `sizeof(t_zone)` to the base address.

This means we need to add  `t_block*` to the `t_zone` struct that holds a pointer to the first  block of the zone. That way we dont need to recompute `(char *)zone + sizeof(t_zone)` everytime.