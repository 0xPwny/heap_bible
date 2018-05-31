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
