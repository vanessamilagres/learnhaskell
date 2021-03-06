The list monad
==============

So far, we've seen how [code]Maybe[/code] values can be viewed as values with a failure context and how we can incorporate failure handling into our code by using [code]&gt;&gt;=[/code] to feed them to functions. In this section, we're going to take a look at how to use the monadic aspects of lists to bring non-determinism into our code in a clear and readable manner.

We've already talked about how lists represent non-deterministic values when they're used as applicatives. A value like [code]5[/code] is deterministic. It has only one result and we know exactly what it is. On the other hand, a value like [code][3,8,9][/code] contains several results, so we can view it as one value that is actually many values at the same time. Using lists as applicative functors showcases this non-determinism nicely:

All the possible combinations of multiplying elements from the left list with elements from the right list are included in the resulting list. When dealing with non-determinism, there are many choices that we can make, so we just try all of them, and so the result is a non-deterministic value as well, only it has many more results.

This context of non-determinism translates to monads very nicely. Let's go ahead and see what the [code]Monad[/code] instance for lists looks like:

[code]return[/code] does the same thing as [code]pure[/code], so we should already be familiar with [code]return[/code] for lists. It takes a value and puts it in a minimal default context that still yields that value. In other words, it makes a list that has only that one value as its result. This is useful for when we want to just wrap a normal value into a list so that it can interact with non-deterministic values.

To understand how [code]&gt;&gt;=[/code] works for lists, it's best if we take a look at it in action to gain some intuition first. [code]&gt;&gt;=[/code] is about taking a value with a context (a monadic value) and feeding it to a function that takes a normal value and returns one that has context. If that function just produced a normal value instead of one with a context, [code]&gt;&gt;=[/code] wouldn't be so useful because after one use, the context would be lost. Anyway, let's try feeding a non-deterministic value to a function:

When we used [code]&gt;&gt;=[/code] with [code]Maybe[/code], the monadic value was fed into the function while taking care of possible failures. Here, it takes care of non-determinism for us. [code][3,4,5][/code] is a non-deterministic value and we feed it into a function that returns a non-deterministic value as well. The result is also non-deterministic, and it features all the possible results of taking elements from the list [code][3,4,5][/code] and passing them to the function [code]\x -&gt; [x,-x][/code]. This function takes a number and produces two results: one negated and one that's unchanged. So when we use [code]&gt;&gt;=[/code] to feed this list to the function, every number is negated and also kept unchanged. The [code]x[/code] from the lambda takes on every value from the list that's fed to it.

To see how this is achieved, we can just follow the implementation. First, we start off with the list [code][3,4,5][/code]. Then, we map the lambda over it and the result is the following:

The lambda is applied to every element and we get a list of lists. Finally, we just flatten the list and voila! We've applied a non-deterministic function to a non-deterministic value!

Non-determinism also includes support for failure. The empty list [code][][/code] is pretty much the equivalent of [code]Nothing[/code], because it signifies the absence of a result. That's why failing is just defined as the empty list. The error message gets thrown away. Let's play around with lists that fail:

In the first line, an empty list is fed into the lambda. Because the list has no elements, none of them can be passed to the function and so the result is an empty list. This is similar to feeding [code]Nothing[/code] to a function. In the second line, each element gets passed to the function, but the element is ignored and the function just returns an empty list. Because the function fails for every element that goes in it, the result is a failure.

Just like with [code]Maybe[/code] values, we can chain several lists with [code]&gt;&gt;=[/code], propagating the non-determinism:

The list [code][1,2][/code] gets bound to [code]n[/code] and [code]['a','b'][/code] gets bound to [code]ch[/code]. Then, we do [code]return (n,ch)[/code] (or [code][(n,ch)][/code]), which means taking a pair of [code](n,ch)[/code] and putting it in a default minimal context. In this case, it's making the smallest possible list that still presents [code](n,ch)[/code] as the result and features as little non-determinism as possible. Its effect on the context is minimal. What we're saying here is this: for every element in [code][1,2][/code], go over every element in [code]['a','b'][/code] and produce a tuple of one element from each list.

Generally speaking, because [code]return[/code] takes a value and wraps it  in a minimal context, it doesn't have any extra effect (like failing in [code]Maybe[/code] or resulting in more non-determinism for lists) but it does present something as its result.

When you have non-deterministic values interacting, you can view their computation as a tree where every possible result in a list represents a separate branch.

Here's the previous expression rewritten in [code]do[/code] notation:

This makes it a bit more obvious that [code]n[/code] takes on every value from [code][1,2][/code] and [code]ch[/code] takes on every value from [code]['a','b'][/code]. Just like with [code]Maybe[/code], we're extracting the elements from the monadic values and treating them like normal values and [code]&gt;&gt;=[/code] takes care of the context for us. The context in this case is non-determinism.

Using lists with [code]do[/code] notation really reminds me of something we've seen before. Check out the following piece of code:

Yes! List comprehensions! In our [code]do[/code] notation example, [code]n[/code] became every result from [code][1,2][/code] and for every such result, [code]ch[/code] was assigned a result from [code]['a','b'][/code] and then the final line put [code](n,ch)[/code] into a default context (a singleton list) to present it as the result without introducing any additional non-determinism. In this list comprehension, the same thing happened, only we didn't have to write [code]return[/code] at the end to present [code](n,ch)[/code] as the result because the output part of a list comprehension did that for us.

In fact, list comprehensions are just syntactic sugar for using lists as monads. In the end, list comprehensions and lists in [code]do[/code] notation translate to using [code]&gt;&gt;=[/code] to do computations that feature non-determinism.

List comprehensions allow us to filter our output. For instance, we can filter a list of numbers to search only for that numbers whose digits contain a [code]7[/code]:

We apply [code]show[/code] to [code]x[/code] to turn our number into a string and then we check if the character [code]'7'[/code] is part of that string. Pretty clever. To see how filtering in list comprehensions translates to the list monad, we have to check out the [code]guard[/code] function and the [code]MonadPlus[/code] type class. The [code]MonadPlus[/code] type class is for monads that can also act as monoids. Here's its definition:

[code]mzero[/code] is synonymous to [code]mempty[/code] from the [code]Monoid[/code] type class and [code]mplus[/code] corresponds to [code]mappend[/code]. Because lists are monoids as well as monads, they can be made an instance of this type class:

For lists [code]mzero[/code] represents a non-deterministic computation that has no results at all — a failed computation. [code]mplus[/code] joins two non-deterministic values into one. The [code]guard[/code] function is defined like this:

It takes a boolean value and if it's [code]True[/code], takes a [code]()[/code] and puts it in a minimal default context that still succeeds. Otherwise, it makes a failed monadic value. Here it is in action:

Looks interesting, but how is it useful? In the list monad, we use it to filter out non-deterministic computations. Observe:

The result here is the same as the result of our previous list comprehension. How does [code]guard[/code] achieve this? Let's first see how [code]guard[/code] functions in conjunction with [code]&gt;&gt;[/code]:

If [code]guard[/code] succeeds, the result contained within it is an empty tuple. So then, we use [code]&gt;&gt;[/code] to ignore that empty tuple and present something else as the result. However, if [code]guard[/code] fails, then so will the [code]return[/code] later on, because feeding an empty list to a function with [code]&gt;&gt;=[/code] always results in an empty list. A [code]guard[/code] basically says: if this boolean is [code]False[/code] then produce a failure right here, otherwise make a successful value that has a dummy result of [code]()[/code] inside it. All this does is to allow the computation to continue.

Here's the previous example rewritten in [code]do[/code] notation:

Had we forgotten to present [code]x[/code] as the final result by using [code]return[/code], the resulting list would just be a list of empty tuples. Here's this again in the form of a list comprehension:

So filtering in list comprehensions is the same as using [code]guard[/code].



A knight's quest

Here's a problem that really lends itself to being solved with non-determinism. Say you have a chess board and only one knight piece on it. We want to find out if the knight can reach a certain position in three moves. We'll just use a pair of numbers to represent the knight's position on the chess board. The first number will determine the column he's in and the second number will determine the row.

Let's make a type synonym for the knight's current position on the chess board:

So let's say that the knight starts at [code](6,2)[/code]. Can he get to [code](6,1)[/code] in exactly three moves? Let's see. If we start off at [code](6,2)[/code] what's the best move to make next? I know, how about all of them! We have non-determinism at our disposal, so instead of picking one move, let's just pick all of them at once. Here's a function that takes the knight's position and returns all of its next moves:

The knight can always take one step horizontally or vertically and two steps horizontally or vertically but its movement has to be both horizontal and vertical. [code](c',r')[/code] takes on every value from the list of movements and then [code]guard[/code] makes sure that the new move, [code](c',r')[/code] is still on the board. If it it's not, it produces an empty list, which causes a failure and [code]return (c',r')[/code] isn't carried out for that position.

This function can also be written without the use of lists as a monad, but we did it here just for kicks. Here is the same function done with [code]filter[/code]:

Both of these do the same thing, so pick one that you think looks nicer. Let's give it a whirl:

Works like a charm! We take one position and we just carry out all the possible moves at once, so to speak. So now that we have a non-deterministic next position, we just use [code]&gt;&gt;=[/code] to feed it to [code]moveKnight[/code]. Here's a function that takes a position and returns all the positions that you can reach from it in three moves:

If you pass it [code](6,2)[/code], the resulting list is quite big, because if there are several ways to reach some position in three moves, it crops up in the list several times. The above without [code]do[/code] notation:

Using [code]&gt;&gt;=[/code] once gives us all possible moves from the start and then when we use [code]&gt;&gt;=[/code] the second time, for every possible first move, every possible next move is computed, and the same goes for the last move.

Putting a value in a default context by applying [code]return[/code] to it and then feeding it to a function with [code]&gt;&gt;=[/code] is the same as just normally applying the function to that value, but we did it here anyway for style.

Now, let's make a function that takes two positions and tells us if you can get from one to the other in exactly three steps:

We generate all the possible positions in three steps and then we see if the position we're looking for is among them. So let's see if we can get from [code](6,2)[/code] to [code](6,1)[/code] in three moves:

Yes! How about from [code](6,2)[/code] to [code](7,3)[/code]?

No! As an exercise, you can change this function so that when you can reach one position from the other, it tells you which moves to take. Later on, we'll see how to modify this function so that we also pass it the number of moves to take instead of that number being hardcoded like it is now.