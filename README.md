# collatz coral

Take any number. If it is even, halve it. If it is odd, triple it and add one.
Repeat. Every number anyone has ever tried ends up at 1. Nobody can prove that
they all do. This page lets you play with that, and when you draw thousands of
the paths on top of each other, they grow into something that looks alive.

Live: <https://xevrion.github.io/collatz-coral/>

## How it works

The rule is the whole thing. Here it is, this is the entire algorithm:

```js
function collatz(n){
  const seq=[n];
  while(n!==1){ n = n%2===0 ? n/2 : 3*n+1; seq.push(n); }
  return seq;
}
```

That is it. There is no trick hiding underneath. The mystery is that a rule this
dumb produces paths nobody can predict. Start at 27 and it bounces around for
111 steps and climbs to 9,232 before it finally falls to 1. Start at 26, one
less, and it is done in 10 steps.

### the hailstone plot

Plot the value at each step and you get a jagged line that rises and crashes
like a hailstone bouncing in a cloud, which is where the name comes from. The
page draws it on a log scale so the big climbs and the long tails both fit.

### the champions

Check every number up to some limit and record two things: how many steps it
takes, and the highest value it touches on the way. The winners are surprising.
Under 10,000 the longest path belongs to 6171 (261 steps) and the highest climb
belongs to 9663, which reaches 27,114,424 before coming back down.

This scan is fast because of memoization. Once a path lands on a number whose
step count is already known, the rest is a lookup. Most paths hit a known number
within a few steps, so checking 10,000 numbers takes tens of milliseconds.

```js
while(!stepCache.has(n)){ path.push(n); n = n%2===0 ? n/2 : 3*n+1; }
```

### the coral

This is the reason the page exists. Reverse each sequence so it starts at 1,
then walk it: draw a short line, and at every step turn slightly left if the
number is even, slightly right if it is odd. Odd turns are a bit sharper than
even ones. Draw thousands of these paths from the same starting point, faint
and overlapping, and a branching organic shape appears. Nobody designed it. It
falls out of the parity pattern of the sequences.

```js
a += seq[i]%2===0 ? evenTurn : oddTurn;
x+=Math.cos(a)*stepLen; y+=Math.sin(a)*stepLen;
```

The angle sliders change the look completely. Small angles give tight seaweed,
big angles give wild coral.

## Usage

Open `index.html` in a browser. No server, no build, no dependencies.

```sh
# just open it
xdg-open index.html

# or serve it if your browser is fussy about file://
python3 -m http.server 8000
```

Type a number and hit trace. Set a limit and hit scan. Move the sliders and hit
grow coral. Save png downloads the coral.

## Limits

| thing | reality |
|---|---|
| numbers above 2^53 | JS numbers lose precision, results become wrong. Use BigInt if you want to go huge. |
| scan of 1,000,000+ | works, but the peak search is not memoized and takes a few seconds |
| the conjecture | this page verifies numbers, it proves nothing. Nobody has. |
| the coral | it is a drawing rule, not a mathematical object. Pretty, not deep. |
