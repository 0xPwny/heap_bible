# heap_bible
## HEAP  (Notes / Techniques / Exploitation / Writeups)

# Heap Bins

## Fast bins 

There are 10 fast bins. Each of these bins maintains a single linked list. Addition and deletion happen from the front of this list (LIFO). No two adjacent free fast chunks consolidate together.

##  Small bins

There are 62 small bins. Small bins are faster than large bins but slower than fast bins. Each bin maintains a doubly-linked list. Insertions happen at the HEAD while removals happen at the TAIL (FIFO). Small chunks may be consolidated together before ending up in unsorted bins.

## Large bins 

There are 63 large bins. Each bin maintains a doubly-linked list. A particular large bin has chunks of different sizes, sorted in decreasing order (i.e. largest chunk at the HEAD and smallest chunk at the TAIL). Insertions and removals happen at any position within the list. Large chunks may be coalesced together before ending up in unsorted bins.

## Unsorted bin

There is only 1 unsorted bin. Small and large chunks, when freed, end up in this bin. The primary purpose of this bin is to act as a “cache layer” to speed up allocation and deallocation requests.

##  Top Chunk 

It is the chunk which borders the top of an arena. While servicing malloc requests, it is used as the last resort.


## Unlink Protection: 
At present day, unlink technique doesnt work since ‘glibc malloc’ has got hardened over the years. Below checks are added to prevent heap overflow using unlink technique.

**Double free:** Freeing a chunk which is already in free state is prohibited. When attacker overwrites ‘second’ chunk’s size with -4, its PREV_INUSE bit is unset which means ‘first’ is already in free state. Hence ‘glibc malloc’ throws up double free error.

```
   if (__glibc_unlikely (!prev_inuse(nextchunk)))
      {
        errstr = "double free or corruption (!prev)";
        goto errout;
      }
```

**Invalid next size:** Next chunk size should lie between 8 bytes to arena’s total system memory. When attacker overwrites ‘second’ chunk’s size with -4, ‘glibc malloc’ throws up invalid next size error.

```
if (__builtin_expect (nextchunk->size <= 2 * SIZE_SZ, 0)
        || __builtin_expect (nextsize >= av->system_mem, 0))
      {
        errstr = "free(): invalid next size (normal)";
        goto errout;
      }
```

**Courrupted Double Linked list:** Previous chunk’s fd and next chunk’s bk should point to currently unlinked chunk. When attacker overwrites fd and bk with free -12 and shellcode address, respectively, free and shellcode address + 8 doesnt point to currently unlinked chunk (‘second’). Hence ‘glibc malloc’ throws up corrupted double linked list error.


```
if (__builtin_expect (FD->bk != P || BK->fd != P, 0))                     
      malloc_printerr (check_action, "corrupted double-linked list", P);
```

