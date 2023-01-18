---
layout: post
title: Decimal places in a floating point number
date: '2015-10-27 19:25:48'
math: true
tags:
- cpp
redirect_from:
- /decimal-places-in-a-floating-point-number
---

Floating point numbers are a mainstay of programming in areas such as games, graphics, and simulation. On the whole, they are easy and intuitive to use. However, they have certain quirks and issues to be aware of. For example, their representation is inherently flexible and often approximate. This means there isn't a simple answer to the question of how many decimal places they can hold.

By looking at the way floating point numbers are stored, it’s possible to understand why this happens and what precision is likely to be available. Hypothetically, it’s possible under certain circumstances to get up to about 45 decimal places in a C++ float, and 324 in a double. However, as we’ll see in this post, it depends on context.

## Floating point format

To begin with, it’s important to have some understanding of how floating point numbers are represented by a computer. If you’re already comfortable with it then you can skip this section. Otherwise, the [Floating-Point Guide](http://floating-point-gui.de/formats/fp/) is a helpful resource, and the [Wikipedia article on floating point](https://en.wikipedia.org/wiki/Floating_point) goes into a lot of technical detail if you’re feeling brave.

In summary, floating point numbers consist of two main parts: a **mantissa** (aka significand) and an **exponent**.

The mantissa contains the digits of the number. The exponent effectively says how far the point is moved left or right from the start of the mantissa. Changing the exponent moves the point, hence the term “floating” point.

Splitting up these two parts of the number format means there are two competing attributes: magnitude and precision. A large positive exponent means you can represent a very large number, running into billions and even trillions. However, doing so means you have fewer mantissa digits (if any) available for less significant digits, such as those after the point. This results in a big number that isn’t very precise.

At the other extreme, a large negative exponent means you can represent extremely small numbers, but only the last few digits of them. Any implicit digits between the point and the start of the mantissa are considered to be 0.

## Significant figures

The size of the mantissa (aka significand) effectively determines how many [significant figures](https://en.wikipedia.org/wiki/Significant_figures) the number can contain.

In C++, this information can be found by using the [`std::numeric_limits`](http://www.cplusplus.com/reference/limits/numeric_limits/) class from the standard library. It’s declared in the `<limits>` header (which is notably different from the C library’s [climits or limits.h](http://www.cplusplus.com/reference/climits/) header).

The **digits** member gives the number of digits in the mantissa. For the standard types, this should always be the number of binary bits. However, there is the possibility for a different base to be used. If so, it’s specified in the **radix** member. The number of significant decimal figures can be calculated using this formula:

$$ digits ∗ ln(2) / ln(10) $$

The result is not likely to be a whole number, so it must be rounded up to find the answer. `ln()` is the natural logarithm, which is called `std::log()` in C++. The 2 refers to the radix of the original number (binary in this case), and 10 refers to the radix of the desired result (decimal in this case). It’s useful to know that this formula can be more generally applied to calculate how many digits of any radix number would be required to represent a number in another radix.

Here is the C++ code to calculate this:

```cpp
int figures = static_cast<int>( std::ceil(
    numeric_limits::digits *
    std::log(numeric_limits::radix) /
    std::log(10.0)
));
```

[**Click here** to see this in an example program on ideone](https://ideone.com/Dyx23W).

For a float, you’ll typically see that it has 24 bits in the mantissa, which should allow **up to 8 significant figures in decimal** with reasonable accuracy. Remember though that we rounded up the final number in the calculation above, so there may be cases where only 7 significant figures are practically usable.

## Significant figures != decimal places

It’s easy to make the mistake of thinking that the code above answers our decimal places question. However, consider the following numbers:

- 0.12345123451234512345
- 0.00000000000000012345

Both of them are written with 20 decimal places. However, the second one would only require 5 significant figures (or the equivalent of 5 decimal digits in the mantissa). The extra leading zeroes after the decimal point can be represented by simply decreasing the exponent (i.e. making it more negative), leaving the entire mantissa available for precision.

This program shows an example of this in action:

```cpp
#include <iostream>
#include <iomanip>
using namespace std;
int main() {
    float f1 = 0.12345123451234512345f;
    float f2 = 0.00000000000000012345f;
    cout << fixed << setprecision(20);
    cout << "f1 = " << f1 << endl;
    cout << "f2 = " << f2;
    return 0;
}
```

[**Click here** to run this program at ideone](https://ideone.com/j3xLPI).

The output from the above program should be something like:

```
f1 = 0.12345123291015625000
f2 = 0.00000000000000012345
```

As you can see, only the first 8 digits of f1 are correct, which corresponds to our finding above that float should support **7 or 8 significant figures in decimal**.

In contrast, all 20 digits of f2 are displayed correctly. The reason is that all the zeroes immediately after the decimal point are _not_ significant figures. In fact, it’s only using 5 significant figures in the mantissa. All those extra zeroes are the result of the exponent shifting the point left.

## Finite representation

It’s worth highlighting at this stage that the exponent is of course finite. This means there is a limited distance by which the point can be shifted left or right. If it gets moved too far left then the mantissa starts to contain leading zeroes too in order to compensate. Such values are referred to as denormal or sub-normal because part of the mantissa has been wasted on zero digits, compromising precision.

Eventually, if the number gets small enough (or big enough), both the mantissa and exponent will be exhausted. This leads to the hard limit of what the floating point representation can accommodate. Dealing with numbers of extremely large or small magnitude is not the only problem with a finite number representation. A far more common and relevant problem is accuracy.

Even when dealing with values which are well within the magnitude limits of typical floating point types, it’s simply not possible to represent all numbers precisely in all radices (or bases). For example, one third (1/3) cannot be represented precisely in decimal (base 10); it is approximately 0.3333333…, but the 3’s go on forever. In contrast, one third in [ternary](https://en.wikipedia.org/wiki/Ternary_numeral_system) (base 3) is precisely 0.1. No need for any recurring digits there.

Similar issues exist in all bases, including binary. Perhaps a surprising example is one tenth (1/10). In decimal, it is simply 0.1, but in binary the digits would go on forever. This isn’t a problem for whole numbers, but it can introduce very significant relative errors where fractional numbers are concerned. The problem is compounded if repeated computations are involved, such as updating positions and velocities every frame in a game or simulation. This issue is sometimes called **floating point drift** , and it means that you basically cannot trust the accuracy of floating point numbers beyond a certain (rather fuzzy) threshold.

Sometimes, the only solution is to try to ensure you can do calculations to an acceptable level of precision, and compensate for the drift afterwards. Sometimes this simply means rounding the results carefully. Another approach common in physical modelling is “damping”, which deliberately _underestimates_ the results of calculations to ensure they don’t spiral off into infinity.

What does this mean for our mission to count available decimal places? Unfortunately, it means that knowing the limit on decimal places isn’t necessarily very helpful if we’re not careful of context. The inaccuracies of the representation could very easily make our calculations meaningless before we reach it.

## Maximum number of decimal places

We’ve seen that there’s no consistent answer to how many decimal places a floating point number will contain as it depends too much on the magnitude. We’ve also seen how significant figures make more sense (though admittedly they are by no means the perfect measure of precision).

However, there are situations where it would be useful to know the **maximum** number of useful decimal places a floating point value _could_ represent with reasonable accuracy under ideal circumstances. For example, maybe you’re developing a [fixed point](https://en.wikipedia.org/wiki/Fixed-point_arithmetic) data type, and you want to know how big it needs to be to represent the full range of numbers that could usefully be stored in a floating point type.

Fortunately, this is possible. All we need to do is check how far the exponent can shift the point, and add it to the number of significant figures the mantissa can represent.

To do this using C++, we turn once again to the `numeric_limits` traits. The `min_exponent` member tells you how far left it can shift the point, effectively indicating the number of additional zeroes it can represent immediately after the point.

The following function calculates the hypothetical maximum number of places a number could represent after the point. For flexibility, it allows it to be calculated in any radix (base). Note that it deliberately returns 0 for integer types because they don’t contain a point.

```cpp
template <typename T_type, int T_radix>
int getMaxPlaces()
{
    static_assert(
        T_radix > 1,
        "Radix must be at least 2."
    );
    static_assert(
        std::numeric_limits<T_type>::is_specialized,
        "Numeric limits traits info not found."
    );
    // Integers contain no point.
    if (std::numeric_limits<T_type>::is_integer)
        return 0;
    // Maximum number of places in the native radix.
    const int places =
        std::abs(std::numeric_limits<T_type>::min_exponent) +
        std::numeric_limits<T_type>::digits;
    // Special case: If the requested radix matches the underlying radix
    // specified in traits then no base conversion is necessary.
    if (T_radix == std::numeric_limits<T_type>::radix)
        return places;
    // Convert the number of places to the requested radix.
    return static_cast<int>( std::ceil(
        places *
        std::log(std::numeric_limits<T_type>::radix) /
        std::log(T_radix)
    ));
}
```

[**Click here** to see this in action on ideone](https://ideone.com/NNsHwn).

Using the function above, you’ll typically find it saying that a float could usefully represent 45 decimal places, and double up to 324. Here’s a very brief snippet of code you could try running to see this happening:

```cpp
float f = 0.0000000000000000000000000000000000000000123456789f;
std::cout << std::fixed << std::setprecision(50) << f << std::endl;
```

The floating point literal has 50 decimal places. However, running this on Visual Studio 2015, it’s only printed correctly up to the 5 (i.e. the first 45 digits). We proved earlier that the mantissa is capable of holding more than 5 digits, so this is an example of a sub-normal.

## Conclusion

We found that in C++, you could *hypothetically* reach 45 decimal places for float, and 324 decimal places for double. Unfortunately though, this is only possible in fairly specific cases where the majority of decimal places are actually zeroes. In practice, most numbers will be very inaccurate by the time you reach that many digits. Hopefully, these values at least provide a useful guideline for what is potentially representable with reasonable accuracy.

An interesting final side note is that floating point numbers representing more decimal places [have actually been observed](https://randomascii.wordpress.com/2012/03/08/float-precisionfrom-zero-to-100-digits-2/). These are exceptional cases though, and are not likely to be mathematically useful.
