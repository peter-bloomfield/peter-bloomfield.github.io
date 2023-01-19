---
layout: post
title: Even Powers and Odd Numbers
date: '2009-02-08 02:04:51'
tags:
- maths
redirect_from:
- /even-powers-and-odd-numbers
- /even-powers-and-odd-numbers/
---

This is a very simple mathematical relationship I found out about recently and which I rather like. I don’t claim to be original here. I'm sure it’s been covered by many people before me.

Here’s an example:

1<sup>2</sup> = 1  
2<sup>2</sup> = 4 = 1 + 3  
3<sup>2</sup> = 8 = 1 + 3 + 5  
4<sup>2</sup> = 16 = 1 + 3 + 5 + 7  
5<sup>2</sup> = 25 = 1 + 3 + 5 + 7 + 9

You’ll notice that the square numbers are being formed by sums of consecutive odd numbers, starting at 1. I don’t have a mathematical proof for this (although I’m sure one exists), but it appears to work for any _even_ power. I wrote a quick computer program to try the same with the 10<sup>th</sup> power, and sure enough, it works:

1<sup>10</sup> = 1  
2<sup>10</sup> = 1024 = (first 32 odd numbers)  
3<sup>10</sup> = 59049 = (first 243 odd numbers)  
4<sup>10</sup> = 1048576 = (first 1024 odd numbers)  
5<sup>10</sup> = 9765625 = (first 3125 odd numbers)

Do you notice the curious relationship between the raised base and the number of summed odd numbers required to reach it? Here’s another way to write it:

1<sup>2</sup> = 1  
32<sup>2</sup> = 1024  
243<sup>2</sup> = 59049  
1024<sup>2</sup> = 1048576  
3215<sup>2</sup> = 9765625

So the raised base is the square of the number of summed odd numbers required to reach it. In other words, where “y” is any positive even number:

![Powers equation 1](/assets/img/migrated/powers_equation_1.png)

(You may recognise “2n – 1” as being the way you calculate the n<sup>th</sup> odd number starting from 1.)

We can actually simplify it a little however. Notice that we’re taking the square root of a power? A root is actually just a fraction of a power, i.e. a square root is the power 1/2, and a cube root is the power 1/3.

As such, we can simplify this by multiplying the powers together, which means simply dividing the power _y_ in half on our summation. But now we have uncovered a (very bizarrely convoluted) demonstration of why this doesn’t work for all odd powers — if you divide an odd number in half, you get a fraction of a number, so (generally speaking) you will end up trying to carry the summation on to an non-integer limit, which is not possible.

(It's worth noting that it does work with odd powers where the base is itself a square number, since that allows the lingering square root in the power to be kind of ‘cancelled’ out, giving an integer in the end.)

Anyway, here’s a slightly simplified form of the equation, where “y” is any positive integer:

![Powers equation 2](/assets/img/migrated/powers_equation_2.png)

You could also write the exponent over the summation as just “y/2”, but this communicates a little better that our final exponent ought to be even.

Now, I’m sure you’re wondering why on earth that is useful, given that you still need to figure out a power in order to calculate how many odd numbers you need to add up in order to calculate your first power. And you are probably right… it’s not much good… _unless_ the exponent you are dealing with is a power of 2… in which case you could conceivably perform some nested summations, where you calculate the upper limit of a given summation using another summation, and so on, all the way from 1 up to half of your original exponent.

The nested summation sounds complicated though! That will be a challenge for another day perhaps. Or at least, when I’ve had more caffeine.
