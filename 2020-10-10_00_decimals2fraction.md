---
title: 'Python: Converting decimals to imperial fractions'
published: 2020-10-14T22:20:00+02:00
mainImg: null
tags:
  - programming
...
For my blog (the one that you're currently reading) I wanted to implement an
automatic conversion from metric to imperial for all measurements that I
mention in my posts.
This would help readers from the US to understand them more easily and
(probably the more relevant reason) be a nice programming task for me.
While I plan to publish the full implementation and write a post about it,
this post focuses on a small part of the task: converting decimal values
into nice fractions with a denominator that is a power of two.

Why powers of two?
Well, you better ask this question someone who actually uses the imperial
system, but my impression is that those fractions are most commonly used
for measurements (like FRAC(3/4)&Prime;).
So how can we solve this problem in Python?
Let's start with an example to better understand it:

Imagine that the input number we want to convert is $`1.26`$.
This number is larger than one, so our first step is to subtract the whole
part, leaving us with the fractional part $`0.26`$ and the whole part $`1`$,
which we remember for later.
We then want to express $`0.26`$ as a fraction $`a/b`$ where $`b`$ is a power
of two, i.e. 2, 4, 8, 16, 32, 64, and so on.
It's also a good idea to keep $`b`$ as low as possible---we don't want e.g.
FRAC(32/64) as our result instead of the much simpler and nicer
FRAC(1/2)---and therefore we start by assuming our denominator $`b = 2`$,
which means $`a/b = a/2 = 0.26`$ and, after multiplying both sides by two,
$`a = 0.52`$.
But of course we want a nice whole number for $`a`$, so by rounding we get
$`\hat{a} = 1`$ and a result of FRAC(1/2).
That's not a very good result, right?
If we convert FRAC(1/2) back to decimals we get $`0.5`$ which is pretty far
away from our original number $`0.26`$.
So let's see if using 4 as the denominator gives a better result:

```math
\begin{aligned}
a/b &= a/4 = 0.26 \\
  a &= 1.04 \approx 1
\end{aligned}
```

Our new result is FRAC(1/4), and if we convert that back we get $`0.25`$.
I think that's close enough.
But how can our algorithm know if we're close enough or not?
I decided to use a maximum relative error: if the difference between original
and converted value is less than UNIT(10,percent) of the original value
that's good enough for me.
After all our use case here is just to give people who aren't familiar with
the metric system a rough impression on what sizes we're talking about.

But wait, we're not done yet. We have to add the whole part that we
subtracted in the first step again, which gives us 1 FRAC(1/4) as the full
result. Now let's look at the code I came up with:

    #!python
    def decimal2fraction(number, maxRelativeError=.1, maxDenominator=64):
        # Convert relative error to absolute error:
        maxError = maxRelativeError * number

        # Split number into whole and fractional part:
        wholePart = int(number)
        fractionalPart = number - wholePart

        # Start with a denominator of 2:
        denominator = 2

        # Loop until the maximum denominator is reached or
        # a suitable fraction is found:
        while denominator <= maxDenominator:
            # Calculate the numerator for the denominator:
            numerator = round(fractionalPart * denominator)

            # Calculate the error:
            error = abs(fractionalPart - (numerator / denominator))

            # Check if we're below the error threshold:
            if error < maxError:
                # We are, so we found a suitable fraction.
                # Let's return it together with the whole part:
                return (wholePart, numerator, denominator)

            # We're not, try the next denominator:
            denominator *= 2

        # We couldn't find a matching fraction:
        return None

Yes, I know that the *return None* is not necessary, but I wanted to make
it clear that the function might not always return a result.
Now we can try it with some values:

    :::python
    >>> decimal2fraction(1.26)
    (1, 1, 4)
    # Nice, same result as we got with our manual calculation.

    >>> decimal2fraction(0.59)
    (0, 5, 8)
    # 5/8 = 0.625, that's around 6% error

    >>> decimal2fraction(0.99)
    (0, 2, 2)
    # Wait, isn't there an easier way to write 2/2?

    >>> decimal2fraction(1.01)
    (1, 0, 2)
    # Hmm, that also looks a bit weird...

Okay, as you can see from the last two examples we're not quite there yet.
We need to handle the two special cases where the numerator is equal to the
denominator (which means we can simply add one to the whole part and leave
out the fraction) and where the numerator is zero (which means we can simply
leave out the fraction).
I decided to return *None* for both the numerator and denominator if there's
no fractional part to return.
Here's the code (new parts marked in yellow):

    #!python hl_lines="24 25 26 28 29 30"
    def decimal2fraction(number, maxRelativeError=.1, maxDenominator=64):
        # Convert relative error to absolute error:
        maxError = maxRelativeError * number

        # Split number into whole and fractional part:
        wholePart = int(number)
        fractionalPart = number - wholePart

        # Start with a denominator of 2:
        denominator = 2

        # Loop until the maximum denominator is reached or
        # a suitable fraction is found:
        while denominator <= maxDenominator:
            # Calculate the numerator for the denominator:
            numerator = round(fractionalPart * denominator)

            # Calculate the error:
            error = abs(fractionalPart - (numerator / denominator))

            # Check if we're below the error threshold:
            if error < maxError:
                # We are, so we found a suitable fraction.
                # Check if the numerator is zero:
                if numerator == 0:
                    return (wholePart, None, None)

                # Check if fractional part simplifies to one:
                if numerator == denominator:
                    return (wholePart + 1, None, None)

                # Otherwise return the fraction together with the
                # whole part:
                return (wholePart, numerator, denominator)

            # We're not, try the next denominator:
            denominator *= 2

        # We couldn't find a matching fraction:
        return None

And now the results look better:

    :::python
    >>> decimal2fraction(0.99)
    (1, None, None)

    >>> decimal2fraction(1.01)
    (1, None, None)

That's it, but as I mentioned there will be a post about the complete unit
conversion sooner or later.
