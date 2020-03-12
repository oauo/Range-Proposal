# JavaScript Range Proposal

This is a proposal to add a way to define ranges of numbers in a straight forward manner without requiring a hacky or long winded solution.

## This feature exists in other languages

- Haskell `[a..b]` & `[a,n..b]`
- F# `[|a..b|]` & `[|a..n..b|]`
- PHP `range(a,b)` & `range(a,b,n)`
- R `a:b` & `seq(a,b,by=n)`

## Limited alternatives in other languages

- C# `Enumerable.Range(a, b)`
- Python `range(a, b+1)`
- Java `Range.between(a, b)`
- Coffeescript `[a..b]`

## Motivation

The current solution in JavaScript to get a range of numbersfrom a to b (inclusive) is to use a solution which applies the keys of an array into another array and then mapping to move to start at `a`

```js
[...new Array(b-a+1).keys()].map(x => x+a)
```

And if a step is required

```js
[...new Array(Math.floor((b-a)/n)+1).keys()].map(x => n*x+a)
```

I find myself needing to use an equivilant of `[0..b]` and `[0..n..b]` all the time but the solutions for those are bulky. Other languages have a way to do this an intended way and it would be a useful tool to have access to.

## Basic Idea

I propose that a syntax similar to F# should be used. `a` is first number (not necessarily smallest), `b` is last number (not necessarily the biggest), and `n` is the gap.

Ranges would be `a` to `b` by optional `n` (multiple) inclusive.

The example code below shows intended syntax, output, JS equivilant

## Example Usage

### a to b

#### Explicit bounds

```js
[1..5] = [1,2,3,4,5]   [...new Array(b-a+1).keys()].map(x => a+x)
[6..3] = [6,5,4,3]     [...new Array(a-b+1).keys()].map(x => a-x)
```

#### Implicit bounds

Missing bounds are implicitly `0`

```js
[..4] = [0,1,2,3,4]    [...new Array(b+1).keys()]
[2..] = [2,1,0]        [...new Array(a+1).keys()].map(x=>a-x)
```

### a to b by n

Ranges stepping by `n` will not go past `b`

#### Explicit bounds

```js
[1..2..5] = [1,3,5]    [...new Array(Math.floor((b-a)/n)+1).keys()].map(x => n*x+a)
[1..2..6] = [1,3,5]
[5..2..1] = [5,3,1]    [...new Array(Math.floor((a-b)/n)+1).keys()].map(x => a-n*x)
[6..2..1] = [6,4,2]
```

#### Implicit bounds

Missing bounds are implicitly `0`

```js
[..2..5] = [0,2,4]     [...new Array(Math.floor(b/n)+1).keys()].map(x => n*x)
[..2..6] = [0,2,4,6]
[5..2..] = [5,3,1]     [...new Array(Math.floor(a/n)+1).keys()].map(x => a-n*x)
[6..2..] = [6,4,2,0]
```

## Further Ideas

### Multiple gaps

These gaps would alternate applying, so gaps `1` and `2` would add `1` then `2` and stops before going over `b`

```js
[1..1..2..10] = [1,2,4,5,7,8,10]
[1..0..1..4]  = [1,1,2,2,3,3,4,4]

let abns = (a,b,...n) => {
    let arr = [], p = a, i = 0
    for(let p = a; p <= b; p+=n[i++%n.length])
        arr.push(p)
    return arr
}
```

### Float ranges

```js
[0..0.2..1] = [0,0.2,0.4,0.6,0.8,1]
[0...2..1]  = [0,0.2,0.4,0.6,0.8,1] //Without leading 0
[0.5..3]    = [0.5,1.5,2.5]
[0..1..2.2] = [0,1,2]
```

## Open Questions

- If `a` and `b` are missing, would it throw an error or return `[0]`

```js
[..] = [0..0] = [0]
[..2..] = [0..2..0] = [0]
```

- If `n` i missing, would it interpret it as `1`, this might be useful for [multiple gaps](#multiple-gaps) but could be confusing and doesn't present any benefit

```js
[2....5] = [2..1..5] = [2,3,4,5]
[1..0....4] = [1..0..1..4] = [1,1,2,2,3,3,4,4]
[1....0......3] = [1..1..0..1..1..7] = [1,2,2,3,4,5,5,6,7]
```

- Is `..` too ambigious in some situations and could be replaced with `:`, would also make it smaller

```js
[1:4]        = [1,2,3,4]
[3:2:8]      = [3,5,7]
[1.3:.2:2.5] = [1.3,1.5,1.7,1.9,2.1,2.3,2.5]
```

## Alternatives

- You can do everything with applying an array's keys to another array but it's unnecessarily complicated
- [Lodash](https://lodash.com/) does have [`_.range(a,b,n?)`](https://lodash.com/docs/4.17.15#range)
- [Coffeescript](https://coffeescript.org/) has [`[a..b]`](https://coffeescript.org/#loops) (scroll down to the section below)
