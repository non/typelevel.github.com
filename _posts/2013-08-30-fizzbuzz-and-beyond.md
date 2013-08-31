---
layout: post
title: FizzBuzz... and Beyond!

meta:
  nav: blog
  author: d_m
  pygments: true
---

(This article makes use of [Spire](https://github.com/non/spire),
[Thyme](https://github.com/Ichoran/thyme), and other libraries.)

> Interviewer: "Now we have this problem we like to ask during interviews to
> test your problem solving and programming abilities. I'd like you to write
> a program that prints the numbers 1 to 100. But for multiples of 3, print
> 'fizz' instead, and for multiples of 5 print 'buzz'. For multiples of 3
> and 5, print 'fizzbuzz'."
>
> (Question taken from 
> [Why Can't Programmers... Program](http://www.codinghorror.com/blog/2007/02/why-cant-programmers-program.html)
> by Jeff Atwood)

Full-disclosure: I hate this question. I understand the problem it's trying to
address, but something about it just grates on me.

I'm not sure if I've ever been asked to do FizzBuzz in an interview. If it did
come up now I'd like to blow it out of the water. So, that's what we're going
to do here. I want to start at 11 and crank it up from there.

I'm going to change a few things about the problem. First, let's say 0 to 99
instead of 1 to 100 (we can always map to that later if we need to, and
1-based indexing is inconvenient). Second, let's return a `Seq[String]`
instead of printing out results. Side effects are for drug trials not
software!

The naive solution
------------------

Here's my first take. It's pretty naive, but it's already quite extensible:
you can provide any range you want instead of `(0 until 100)` and you can easily
expand the list of divisors from just 3 and 5 to others as well.

```scala
val mods = Seq(3 -> "fizz", 5 -> "buzz")

def test1(limit: Int): Seq[String] =
  (0 until limit).map { n =>
    val words = mods.collect {
      case (m, s) if n % m == 0 => s
    }
    val v = words.mkString
    if (v == "") n.toString else v
  }

test1(100)
```

It also compresses down into this tweetable 113 character version:

```scala
(0 to 99)map{n=>val v=Seq(3->"fizz",5->"buzz").collect{case(m,s) if n%m==0=>s}.mkString;if(v=="")n.toString else v}
```

But this version has some problems. For each possible divisor (e.g. 3) we do
`n` trial divisions (where `n` is the upper limit of our range). That's not
going to scale very well, especially when we want to print "baz" for 7 and
"qux" for 11.

Round Two
---------

We can do a lot better. Instead of dividing every single number in our range,
let's just start with 0 and increment by 3 until we leave our range. This way,
we'll mark all the multiples of 3 without needing to do *any* division.

(You'll notice that 0 is divisible by any number, which is by design.)

```scala
def test2(limit: Int): Seq[String] = {
  val words = Array.fill(limit)("")
  mods.foreach { case (m, s) =>
    for (i <- 0 until limit by m) words(i) += s
  }
  words.zipWithIndex.map { case (s, i) =>
    if (s == "") i.toString else s
  }
}

test2(100)
```

This uses a little bit of temporary local state, but since we don't expose it
to the world it's fine. If we benchmark `test1` and `test2` using
[Thyme](https://github.com/Ichoran/thyme)
we see that `test2` is about 2.5x faster than `test1`:

```
scala> val th = new ichi.bench.Thyme()
th: ichi.bench.Thyme = ichi.bench.Thyme@6942e786

scala> th.benchOffPair()(test1(100).length)(test2(100).length)
res1: (Int, ichi.bench.Thyme.Comparison) =
(100,Benchmark comparison (in 22.90 s)
Significantly different (p ~= 0)
  Time ratio:    0.43047   95% CI 0.42885 - 0.43209   (n=20)
    First     20.55 us   95% CI 20.52 us - 20.58 us
    Second    8.846 us   95% CI 8.816 us - 8.877 us)
```

So this is working pretty well. Let's keep going.

Moving the Goalposts
--------------------

TBD
