# notes

learning log for collatz coral. first person, messy on purpose.

### what i built
one html file. three panels. trace one number and see its hailstone path, scan
every number up to a limit and find the champions, and draw thousands of paths
as a coral. no libraries, no build, canvas only.

### measured numbers (real runs, node + headless chrome)

| limit | memoized step scan | brute-force peak scan | numbers cached |
|---|---|---|---|
| 10,000 | 11 ms | 11 ms | 21,664 |
| 100,000 | 62 ms | 80 ms | 217,212 |
| 1,000,000 | 742 ms | 2,692 ms | 2,168,611 |

champions found:

| range | longest path | highest climb |
|---|---|---|
| up to 10,000 | 6171, 261 steps | 9663 reaches 27,114,424 |
| up to 1,000,000 | 837799, 524 steps | 704511 reaches 56,991,483,520 |

837799 taking 524 steps under a million matches the known record, so the
memoizer is right. 27 gives 111 steps and peak 9232, also the known values.

### the cache is bigger than the limit, and that surprised me
scanning up to 10,000 cached 21,664 numbers. more than double. because the
paths wander way above the limit before coming down (9663 goes to 27 million)
and every number touched on the way gets cached. so the cache is not "numbers
i checked", it is "numbers any path ever visited". at a million it is 2.1M
entries. i assumed the cache would be about the size of the limit. wrong.

### assumption that was wrong: the peak scan
i thought i could get the peak for free from the same memoized walk. no. the
step count of a number only depends on where the path lands, but the peak
depends on the whole path, and a cached node does not know the highest point
its own descendants reached. so the peak scan is brute force, and at a million
it is 3.6x slower than the memoized step scan. i left it brute force because
memoizing peak properly (store max-of-path per node) is a real change and the
scan still finishes in under 3 seconds. noted below to try later.

### the bug that cost time: the memoizer returned the wrong thing
symptom: scanning gave champions that were off by one or two steps in some
cases, but 27 came out right, so i did not notice for a while. cause: the
cache-fill loop walks the path backwards and increments `s`, and i originally
returned `s` at the end, which is the count for the LAST element pushed, not
the first. the fix was to return `stepCache.get(path[0])`. subtle, because for
short paths the two happened to agree. the lesson: test a number whose path
does not immediately hit the cache, not just the famous one.

### why draw the coral backwards from 1
if you draw each path forward from its own start, every path begins somewhere
different and you get spaghetti. reversing so every sequence starts at 1 means
they all share one root and the shared prefixes overlap exactly. that overlap is
what makes the branches thick near the root and thin at the tips. it looks like
a tree because it IS a tree, the collatz graph really is one (assuming the
conjecture).

### why odd turns are sharper than even turns
i tried equal angles first. it came out as a boring fan. the classic look
needs asymmetry, so odd steps turn about 1.9x harder the other way. i do not
have a principled reason for 1.9, it just looked the most like the numberphile
plot. this is a drawing choice, not math.

### why log scale on the hailstone plot
27 goes from 27 to 9232 and back. on a linear scale the interesting low part is
a flat line at the bottom. log scale shows the bounces.

### why devicePixelRatio everywhere
without it the canvas is blurry on any hidpi screen. so every canvas gets
resized to css-width times dpr and line widths are multiplied by dpr. small
thing, easy to forget, ugly when forgotten.

### to try later
- memoize peak properly (store the max along the path per cached node)
- BigInt mode so numbers past 2^53 stop lying
- animate the coral growing path by path instead of all at once
- color paths by length so long ones stand out
- the actual collatz tree as a graph, not a turtle drawing
- try the reverse: from 1, enumerate predecessors and grow the real tree
