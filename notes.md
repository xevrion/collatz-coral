# notes

learning log for collatz coral. first person, messy on purpose, this is for future me.

### what i built
one html file with three panels: trace one number and watch its hailstone path, scan every number up to some limit and find the champions, and draw thousands of paths together as a coral. no libraries, no build step, just canvas.

### measured numbers (real runs, node + headless chrome, not estimates)

| limit | memoized step scan | brute-force peak scan | numbers cached |
|---|---|---|---|
| 10,000 | 11 ms | 11 ms | 21,664 |
| 100,000 | 62 ms | 80 ms | 217,212 |
| 1,000,000 | 742 ms | 2,692 ms | 2,168,611 |

champions it found:

| range | longest path | highest climb |
|---|---|---|
| up to 10,000 | 6171, 261 steps | 9663 reaches 27,114,424 |
| up to 1,000,000 | 837799, 524 steps | 704511 reaches 56,991,483,520 |

837799 taking 524 steps under a million is the known record, so the memoizer is actually right, and 27 giving 111 steps with a peak of 9232 matches too. good, cuz i almost shipped it wrong (see the bug below).

### the cache is way bigger than the limit and that surprised me
scanning up to 10,000 cached 21,664 numbers, more than double. i assumed the cache would be roughly the size of the limit and that was just wrong. the paths wander WAY above the limit before coming back down (9663 goes past 27 million) and every number touched on the way gets cached, so the cache isn't "numbers i checked", it's "every number any path ever visited". at a million it's 2.1M entries. makes sense in hindsight but i didn't see it coming.

### assumption that was wrong: i thought the peak came free
i figured i could grab the peak from the same memoized walk as the step count. nope. the step count of a number only depends on where the path eventually lands, but the peak depends on the whole path, and a cached node has no idea how high its own descendants went. so the peak scan is brute force, and at a million it's like 3.6x slower than the step scan. i left it that way cuz memoizing peak properly (store the max-of-path per node) is a real change and it still finishes in under 3 seconds. it's in the try-later list.

### the bug that cost time: the memoizer returned the wrong number
symptom: the scan gave champions that were off by one or two steps sometimes, but 27 came out perfect so i didn't notice for a while and kept going. the cause was that the cache-fill loop walks the path backwards incrementing `s`, and i was returning `s` at the end, which is the count for the LAST element pushed, not the first one i actually asked about. fix was returning `stepCache.get(path[0])`. it was subtle cuz for short paths the two happen to agree, so the famous test number passed and hid it. lesson: test a number whose path doesn't immediately hit the cache, not just the one everybody uses.

### why the coral is drawn backwards from 1
if you draw each path forwards from its own starting number, every path begins somewhere different and you get spaghetti. reversing them so every sequence starts at 1 means they all share one root, and the shared prefixes overlap exactly on top of each other. that overlap is what makes the branches thick near the root and thin at the tips. it looks like a tree because it IS a tree, the collatz graph really is one (assuming the conjecture holds lol).

### why odd turns are sharper than even turns
i tried equal angles first and it came out as a boring fan. the classic look needs asymmetry, so odd steps turn about 1.9x harder the other way. i don't have a principled reason for 1.9, it just looked the most like the numberphile plot. this is a drawing choice not math, don't read into it.

### why log scale on the hailstone plot
27 goes from 27 up to 9232 and back. on a linear scale the interesting low part is a flat line squished at the bottom and you see nothing. log scale shows the actual bounces.

### why devicePixelRatio everywhere
without it the canvas is blurry on any hidpi screen, so every canvas gets resized to css-width times dpr and every line width is multiplied by dpr too. small thing, easy to forget, ugly when forgotten.

### to try later
- memoize peak properly (store the max along the path per cached node)
- BigInt mode so numbers past 2^53 stop lying
- animate the coral growing path by path instead of all at once, would be a great stream moment
- color paths by length so the long ones stand out
- draw the actual collatz tree as a graph instead of a turtle drawing
- go the other way: from 1, enumerate predecessors and grow the real tree upward
