- [ ] The library must be named `libft_malloc_$HOSTTYPE.so`
- [ ] Must have a Makefile containing `all`, `re`, `clean`, `fclean`.
- [ ] Makefile must also verify the existence of the environmental variable `$HOSTTYPE`. If this variable is empty or undefined, it should be assigned the following value:
```bash
ifeq ($(HOSTTYPE),)
	HOSTTYPE := $(shell uname -m)_$(shell uname -s)
endif
```

- [ ] The Makefile musr create a symbolic link libft_malloc.so pointing to libft_malloc_$HOSTTYPE.so. This means:
```
libft_malloc.so -> libft_malloc_inter-mac.so
```

- [ ] Include own libft including its Makefile in the root. Mallocs Makefile will have to complie the library, then compile the project and link the archive.
- [ ] One global variable is allowed to manage the allocations and one for theread safety
- [ ] Code must look nice

- [ ] Allowed functions in the mandatory are:
	- mmap(2)
	- munmap(2)
	- sysconf(_SC_PAGESIZE)
	- getrlimit(2)
	- functions already allowed from libft
	- functions already allowed fron libpthread

- [ ] For bonus, any function that can be justified is allowed

