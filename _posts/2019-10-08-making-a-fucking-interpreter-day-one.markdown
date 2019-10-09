---
layout: page
title: "Making a F*cking Interpreter - Day One"
date: 2019-10-08 23:35:00 +0200
categories: dev project cpp
---

This is a project that I've wanted to do for a very long time now, and the first in a three part series of possibly the dumbest possible project ideas. Still, it is interesting, and now I have the excuse of using it to better my C++ as I study it more deeply in class.

## The Project

To create an interpreter for the exoteric language BrainF\*ck. The name for such an interpreter? A f\*cking interpreter. So, before I begin, a little background on how BrainFuck works:

It's a very simple language that is composed of few instructions. You have your `>` and `<`'s, `+` and `-`'s, and `[]`'s and `.`. What each of does is rather evident, but before explaining it I need to mention that all the operations are done on a "tape", very much like in a Turing Machine. Thus the operators do the following:

* `>` and `<` : Move right and left on the tape

* `+` and `-` : Increment or decrement the value currently at that position on the tape

* `[]` : This is more interesting. The way this works is that when the program finds a `[` it will do a check to see if the current position has a 0. If so, it will skip whatever is inside the `[]`, otherwise it will execute it and, upon reaching the `]`, it will return to the `[` and do the check once again (but in the now current position in the tape)

* `.` : This prints the value at the current position of the tape as an ASCII character

So a sample program would be

```
++++>++>++<+
```

Which simply puts a 4 on the first position of the tape, then a 2 on the second, then a two on the third, then returns to the second to turn it into a three.

With that understood I can move on to actually creating the interpreter...

## Day One

On the first day of this project I created the basis: The class interpreter, it's components, and the basic functions for the first four operations. This is enough to do something, but not yet enough to have fun. Still, I decided to start this project far too late at night, and I have class in the morning, so this is what I'll do.

Firstly, the interpreter is going to be made up of 4 main components:

* The `tape`, which is an `std::vector<int>`
* The `head`, an `int` that tells it it's current position on the tape
* The `program`, which is a `std::string` with the set of instructions that the interpreter must follow. This is important to keep so that we can use it later when we start to do the loops.
* The `pc`, or "Program Counter", an `int` which tells us the current position in the program.

With these components decided, we are going to have the private functions that perform each of the operations for the interpreter:

* `moveRight()` and `moveLeft()`
* `increment()` and `decrement()`

And we'll also need a function to run a piece of code, which I called `run(std::string)`. I also defined a function `execute(char)`, which is the function responsible for calling each of the individual functions for each operation. For now I left it as public, as it helps in testing, but later it should probably be moved to private, if it is going to be kept at all.

So, finally, the class header looks like this:

```cpp
class Interpreter {
    private:
        std::vector<int> tape;
        int head;

        std::string program;
        int pc;

        void moveRight();
        void moveLeft();
        void increment();
        void decrement();

    public:
        void execute(char);
        void run(std::string);

        void printTape(std::ostream&);
};
```

The `printTape(std::ostream&)` function was one I created for testing, and it takes an `std::ostream&` because I intended to use it to overwrite the << operator for the class Interpreter, but I kept getting a weird error and, again, it's quite late, so I decided it was a better plan to just leave it for later.

### A few notes on implementation

Finally, a few things about the implementation of those functions that I think are worthy to mention:

When moving through the tape it's possible to eventually go out of bounds, and there are a couple of different ways to approach this: you can either make changes to the tape right away, so that the head is always pointing to a position that actually exists, or you can wait to only make the changes when necessary. That is, if the person goes 10 cases in, then comes back 10 cases, there might be no need to actually create those cases on the tape, so you can have only the cases that have actually been interacted with (incremented or decremented).

Another important detail is that even after deciding the things above, it's always a good idea to make sure that the tape has something at the position you're trying to increment/decrement. Even if you think that you have guaranteed that through the movement functions or whatever, there can always be an edge case that will cause a segfault on your project, which won't be fun at all.
