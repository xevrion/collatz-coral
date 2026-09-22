# collatz coral

so take any number, and if it's even you halve it, if it's odd you triple it and add one, and you just keep doing that. every single number anyone has ever tried ends up at 1, and the wild part is nobody on earth can prove they all do. it's been open since the 1930s and it's basically a meme in math circles cuz it looks like a homework problem and it eats careers.

this page lets you mess with it, and when you draw thousands of the paths on top of each other they grow into something that genuinely looks alive, like coral or seaweed. that's the whole reason it exists.

since the thing that comes out looks like algae, the page is built as one of anna atkins' cyanotypes. she made *photographs of british algae* in 1843, laying seaweed on light sensitive paper and letting the sun print around it, and it was the first book ever illustrated with photographs. so you get prussian blue paper, the specimen in white where it blocked the light, a latin binomial, and plates numbered I, II, III. the numbers are written into the specimen label instead of sitting in a stats bar.

live: <https://xevrion.github.io/collatz-coral/>

## how it works

the rule is the entire thing, there's nothing hiding underneath, this is literally the whole algorithm:

```js
function collatz(n){
  const seq=[n];
  while(n!==1){ n = n%2===0 ? n/2 : 3*n+1; seq.push(n); }
  return seq;
}
```

and that's what makes it so annoying (in a good way). a rule this dumb produces paths nobody can predict. start at 27 and it bounces around for 111 steps and climbs all the way up to 9,232 before it finally gives up and falls to 1. start at 26, literally one less, and it's done in 10 steps. there's no pattern you can see, and that's the mystery.

### the hailstone plot

if you plot the value at each step you get this jagged line that shoots up and crashes down over and over, like a hailstone bouncing around inside a cloud before it drops (thats actually where the name comes from). the page draws it on a log scale cuz otherwise the big climbs flatten everything else into a line at the bottom and you can't see the bounces.

### the champions

check every number up to some limit and keep track of two things, how many steps it takes and the highest point it touches on the way. the winners are kinda surprising. under 10,000 the longest path is 6171 (261 steps) and the biggest climb is 9663, which goes all the way up to 27,114,424 before coming back down, from a starting number under ten thousand lol.

the scan is fast because of memoization, and this is the one actual cs lesson in here. once a path lands on a number whose step count you already know, the rest is just a lookup, and most paths hit a known number within a few steps, so checking 10,000 numbers takes tens of milliseconds:

```js
while(!stepCache.has(n)){ path.push(n); n = n%2===0 ? n/2 : 3*n+1; }
```

### the coral

ok this is the fun part. you reverse each sequence so it starts at 1, then you walk it like a turtle: draw a short line, and at every step turn a little left if the number is even and a little right if it's odd (odd turns are a bit sharper). do that for thousands of numbers from the same starting point, faint and overlapping, and a branching organic shape just appears. nobody designed it, it falls straight out of the even/odd pattern of the sequences:

```js
a += seq[i]%2===0 ? evenTurn : oddTurn;
x+=Math.cos(a)*stepLen; y+=Math.sin(a)*stepLen;
```

play with the angle sliders, they change the look completely. small angles give you tight seaweed, big ones give you wild spiky coral.

## usage

just open `index.html` in a browser. no server, no build, no dependencies, nothing to install.

```sh
# just open it
xdg-open index.html

# or serve it if your browser is fussy about file://
python3 -m http.server 8000
```

three plates, switch between them with the roman numerals at the bottom right:

- **I** the coral. drag *specimens* for how many numbers to print and *turn* for the angle. save the print downloads it, blue and all.
- **II** one specimen. type a number, or hit take another for a random one.
- **III** the scattering. every number up to your limit, placed by how many steps it needs.

each plate deep links, so `#coral`, `#one` and `#scan` open straight to one.

## limits

| thing | reality |
|---|---|
| numbers above 2^53 | js numbers lose precision and the results become lies. use BigInt if you wanna go huge |
| scanning 1,000,000+ | works, but the peak search isn't memoized so it takes a few seconds |
| the conjecture itself | this page checks numbers, it proves nothing. nobody has |
| the coral | it's a drawing rule, not a real mathematical object. pretty, not deep |
