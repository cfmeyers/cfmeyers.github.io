---
layout: post
title: "Making Change With Dynamic Programming"
date: 2015-04-27T21:38:02-04:00
---

In brushing up on my Computer Science, I came across [Problem Solving with Algorithms and Data Structures](http://interactivepython.org/runestone/static/pythonds), a good high-level review of the topic using Python.

One problem, a [review of dynamic programming](http://interactivepython.org/runestone/static/pythonds/Recursion/DynamicProgramming.html), had me stepping through the solution very slowly.  

The problem is "what's the fewest number of coins it takes to make an arbitrary amount of change (given an arbitrary coin system)."  Obviously if we're talking about the American coinage system, the best way to find the fewest number of coins to make, say, $0.33, is to take the largest coin smaller than $0.33 (a quarter), add one to the count and subtract it from the amount, then next largest (a nickel), and so on, to get 1 quarter, 1 nickel, and 3 pennies.  This works well on a regular system of coins like America's, but fails when you have a "21 cent piece."  

One of the solutions the book gives uses Dynamic Programming.  I'd highly encourage you to take a look at their solution.  I'm re-writing it below more simply and with better variable names.

The dynamic programming strategy we'll employ is to compute the lowest possible change amount for all change-sizes up to the amount of change we want to make.  

~~~python
def dpMakeChange(coinValueList,change):
    minCoins = [0]*(change+1)
    for position in range(change+1):
        coinCount = position  #worst case scenario...can only make change with pennies
        usableCoinList = [c for c in coinValueList if c <= position]
        for coin in usableCoinList:
            if minCoins[position-coin] +1 < coinCount:
                coinCount = minCoins[position-coin] +1
        minCoins[position] = coinCount

    return minCoins[change]

print(dpMakeChange([1,5,10,25],63))
~~~

So we're calling the function with 2 arguments: `coinValueList`, a list of possible denominations (e.g. it would be `[1, 5, 10, 25, 50]` in the US), and `change`, the amount of change we want to make.


First we create an array (`minCoins`) for storing the fewest possible coins it would take to make a given change for each possible change from zero to `change`.

~~~python
    minCoins = [0]*(change+1)
~~~

Notice we initialize with zeros.  This will make the rest of the code cleaner.

The bulk of the problem is in the outer `for` loop:

~~~python
    for position in range(change+1):
        .
        .
        .
    return minCoins[change]
~~~
We're using this `for` loop to build our `minCoins` array.  Each element in the array gives you the minimum number of coins it would take to make change at that index.  For example, `minCoins[5]` would give you the fewest number of coins you could use to make 5 cents (in this case it would be just 1, a nickel).

Notice the `return` value: we're returning `minCoins[change]`, i.e. the fewest number of coins you could use to make `change` cents in change.

Next, notice `coinCount`.  We're going to use this variable to keep track of the best solution we know to the "fewest number of coins to make `position` cents."  Notice how we're initializing it to `position`:  that's going to be our worst case scenario, one in which we have to use all pennies to make change.  Using all pennies is going to be the way to make change that requires the _most_ coins (but sometimes, as in the case of `position` is 3 cents, it's also the fewest).

~~~python
    for position in range(change+1):
        coinCount = position  #worst case scenario...can only make change with pennies
        usableCoinList = [c for c in coinValueList if c <= position]
        .
        .
        .
~~~

`usableCoinList` is an expedient.  If `coinValueList` has pennies, nickels, dimes, and quarters, and we need to make 8 cents in change, we don't want to try to use quarters.  
The inner `for` loop is going to iterate over every coin value in in `usableCoinList` (e.g., if `position` is 17 and we have standard US coinage, we'll go through 1, 5, and 10).

~~~python
    for position in range(change+1):
        coinCount = position  #worst case scenario...can only make change with pennies
        usableCoinList = [c for c in coinValueList if c <= position]

        for coin in usableCoinList:
            .#possibly
            .#modify
            .#coinCount

        minCoins[position] = coinCount
~~~

As we go through each coin, we're going to try and make change with fewer coins thann `coinCount` (which we initialized to our worst case scenario, i.e. "all pennies").  If we find a way with fewer coins, we'll change `coinCount` to reflect that (greedy strategy).  Once we've exhausted all the possibilities, we'll set our `minCoins` array at that position to `coinCount`.  In this way we've calculated the fewest possible coins needed to make `position` cents in change, and saved it in our `minCoins` array for future reference.

~~~python
    for position in range(change+1):
        coinCount = position  #worst case scenario...can only make change with pennies
        usableCoinList = [c for c in coinValueList if c <= position]

        for coin in usableCoinList:
            if minCoins[position-coin] +1 < coinCount:
                coinCount = minCoins[position-coin] +1

        minCoins[position] = coinCount
~~~

The inner `for` loop works examining each `coin`, subtracting it from `position`,  and checking if the value at that difference in the `minCoins` array is (plus 1) is smaller than `coinCount`.  If it is, we've found a new "smallest # of coins".

Example:  suppose `position` is 7.  That means `usableCoinList` is going to be `[1, 5]`.  We start by initializing `coinCount` to our worst-case scenario, 7 ("all pennies").  Next we go through each `coin` in `usableCoinList`.

`coin` is 1 (penny).  `position-coin` is 7-1, 6.  So we check the 1 + minimum number of coins to make 6 cents in change (stored at `minCoins[6]`).  Since that is 3 (a penny and a nickel + 1), and since 3 < 7, we re-set `coinCount` to be 3 (a penny, a nickel, and another penny).  

We repeat for when `coin` is 5 (a nickel).  We happen to get the same results so, we don't reset the `coinCount`.

In this way we build a reference for all the ways to make change smaller than the amount we're actually interested in, then use that reference to calculate the amount we're interested in.




