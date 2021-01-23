Recursion is often taught as something that goes along these lines:

- Base case (or termination case)
- Recursive step

Or really some variant of that. Now, to me, that's a bad way of thinking about it. It's a very accurate way of thinking about things, because it's incredibly general. So it's correct.

But it's not a **good** way of thinking about recursion. It hardly makes you **feel** the recursion. So I'm going to put my way of thinking about it down.

## Functions

Let's start with the humble function. A function, in its simplest form, is something that performs an action. One action. If you want to think about more complex functions, they may perform multiple actions.

But ultimately those functions can be decomposed into a set of functions that really do just one thing.

So let me define this: **A function is something that performs an action.**

## Recursive functions

Now the first thing to note is that recursion fundamentally takes the form of a **recursive function**. Know what that means? It means it's first and foremost a function. So a recursive function must perform an action.

Now let's take a couple of concrete examples.

- In **walking a tree**, perhaps you want to print every node in the tree, and you don't care about order, as long as every node is printed. You would meaningfully define a function, `printNode()` for instance, that prints the contents of a node. That's the action.

- In **building a railroad**, perhaps you want to lay the next track segment. You may define a function `layNext()`, that puts a track segment right in front of the current one. That's the action.

- In **filling the screen with red pixels**, you may define a function `fillNext()`, that changes the color of all adjacent pixels from the current one to red. That's the action.

So the point is, recursive functions must first define the action that it wants to perform. That's just a function.

Now, recursive functions also do one more thing: They know what to **do next**.

So they need to put that in as well. Going back to our examples:

- In **walking a tree**, with `printNode()`, you will also do: `for all children of current node, printNode()`. That's the next step.

- In **building a railroad**, with `layNext()`, you will also do: `for the newly laid track segment, layNext()`. That's the next step.

- In **filling the screen with red pixels**, with `fillNext()`, you will also do: `for all newly filled adjacent pixels, fillNext()`. That's the next step.

And that's (almost) all there is to thinking about recursion. Recursive functions basically have:

- The action
- The next step

Isn't it simple?

## A little bit more

If you're thinking, but something seems missing, you're right. Again, I'm going to ignore classic terminology here and use my own. It may not be the best, but I'm presenting to you **my** framework of thinking about recursion. So pinch of salt, please.

The above recursive functions are a bit special. Let's see why.

- **Walking a tree**. We are just printing the nodes of the tree. That's the action. Meaning we just need to touch, or visit, each node, and we're done. There's nothing more to do.

- **Building a railroad**. We are creating new items as we go along. That's the action. Meaning we just need to make a new thing, and we're done. There's nothing more to do.

- **Filling the screen**. We are creating new pixels as we go along. That's the action. Meaning it's exactly the same as the railroad.

All of the examples were carefully chosen so that the **action is self-contained in the function**. The information at a given level of recursion is fully sufficient to perform the action.

The **tree node** we are looking at is has enough information to print the node.

The **track** we are looking at has enough information for us to lay the next track.

The **current pixel** has enough information to color the adjacent pixels.

I call this "visitor-style" recursion.

There is another case where the information at a given level of recursion is not enough to perform the action. And this is often where the intuitive sense of recursion starts to fall apart.

But armed with the above intuitive grasp of recursion, let's take a look on how we can extend that to cover this harder (but generalized) case.

## Evaluator-style recursion

If the information at a given level of recursion is not enough, then it means that we need the result of further recursion to perform the action at the current level of recursion.

If you like textbook examples, the obvious one here is Fibonacci.

```
func fib(n) {
    // handle the base cases properly
    if n == 0 or 1 { return n }

    // recursion
    return fib(n-1) + fib(n-2)
}
```

It's very clear here. The action of `fib()` is to compute, and return, the nth fibonacci number. But it cannot do that just with the value of `n`. It needs more information, and that information is only available by further recursion.

Now let's do the **action** and **next step** modelling.

What's the action? To compute, and return, the Fibonacci number at index `n`.

What's the next step? For the two previous indices, `n-1` and `n-2`, compute, and return, their Fibonacci numbers.

So far so good. We got the action and the next step down. But, again, the action cannot be completed without the results of the next step. This puts us in a bit of a chicken-and-egg problem. Or does it?

This is exactly why we often feel that intuition breaks. But actually it doesn't.

The action defers its completion until the (just) next step is complete.

If we **know** the Fibonacci numbers at indices `n-1` and `n-2`, then the action can complete. If we don't know, then we again defer the completion of the action at `n`, `n-1`, and `n-2`, until the next steps (plural) are complete. We keep waiting, until we can actually complete the next step – the **last** next step.

And we know we can always complete the **last** next step, because the action on the **last** next step is stupidly trivial. In this case, `if n is 0 return 0`, `if n is 1 return 1`. Literature likes to call this the base case. Whatever.

And because we've been waiting, and we finally have a solution, now we can perform the action before the **last** next step, and the action before it, and all the way back to the front. Solving our problem.

## A mental parallel

Here's a mental parallel for recursion. When you start to piece together a jigsaw puzzle, you have a whole mess in front of you.

You can express the algorithm to solve a jigsaw, very loosely, as something like:

> Grab a piece, find the pieces that can join with that piece, and link them. For all the new pieces that were just linked, repeat the process.

Sounds about right, yea? Pretty much how you would do it?

But what's the catch here? You don't know where the first piece sits on the board. So you find a piece that you know has to be at a certain place on the board. That's your corner pieces. Once you find a corner piece, and put it in the correct place, then now everything starts to fall in place.

### Visitor-style mental model

That's the visitor-style recursion.

You started with something that you have – the corner piece – and then constructed everything around it by repeating the same action (find adjacent, put adjacent) until you solved the entire puzzle.

In code:

```
func nextPiece(p) {
    pieces := findPiecesThatCanLinkWith(p)
    layPiecesOnBoard(pieces)
    for each piece in pieces {
        nextPiece(piece)
    }
}
```

It's visitor-style because we assume that `findPiecesThatCanLinkWith()` can complete with only the current piece, `p`.

### Evaluator-style mental model

Or we can think of it the other way. We don't start with the base case – the corner piece. We start with any piece.

This is evaluator-style.

Why? Because we cannot perform our action (to put a given piece down) without more information – information that can only be found in a sub-recursion.

So how would you do that? You take an arbitrary piece, and you realize that if only, you know, **if only** you knew where its adjacent piece was, you could link the piece you're holding to it. And if you knew where the adjacent piece of that adjacent piece was, then you could like the adjacent piece to it.

And so on. So really, if you're thinking along this model, then you need the result of another (level of) recursion – finding the adjacent piece of the piece you're holding – to put your piece down on the board.

So we can keep going, until we hit the base case – we know where the corner piece is. With that, we can lock in the corner piece in it's spot, and now everything immediately falls into place, because you know which piece links to which piece. You just needed that corner piece to be solved.

The **last next step**.

Now, this is not a perfect analogy because:

- There are usually 4 corner pieces

- Corner pieces don't have a guaranteed location on the board (though humans can usually tell where they need to go)

- It's possible to hit the corner piece because the entire puzzle is solved

But it's meant to form an intuitive sense of recursion and base cases.

I'm sure a better analogy exists.
