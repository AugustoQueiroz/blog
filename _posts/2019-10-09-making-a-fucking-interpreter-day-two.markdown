---
layout: post
title: "Making a F*cking Interpreter - Day Two"
date: 2019-10-09 11:43:00 +0200
categories: dev project cpp
---

*You can find the code for this project (and the result as well) on [AugustoQueiroz/fucking-interpreter](https://github.com/AugustoQueiroz/fucking-interpreter)*

So, yesteday I got the baseline for the project working: the skeleton of the class `Interpreter` and the functions for moving around the tape, incrementing and decrementing the value at the current position, executing a single operation, and running a program. So today the first two things to be done are quite clear: print the current value at the current position of the tape through the `.` operator, and loop through the `[]` operators. After that there are two things I need to work on: getting a program to be passed to the interpreter through the command line (as you do when you `$ python your-program.py`), and creating the interpreter interface (which has a real name I can't remember)(as you do when you just `$ python`).

So, with goals defined we can go on to start...

## Day Two

First things first, and by first things I mean easiest things first, I started by the `.` operator. All it needed was a new private function, `void print()`, and in that function I write to the output stream by taking the value the head is currently pointing at, and casting it to a char (`(char) tape.at(head)`).

So, easiest out of the way, moving on to what is probably the most complex part of this language, the loop operator. And, even then, it's not that complex. To take care of the loops I added four function: `bool testLoop()`, which simply checks that the current position on the tape is, indeed, different than 0, `void skipLoop()`, which goes to the end of the loop without executing any of the commands inside, `void startLoop()` which saves the position of the start of the loop, and `void endLoop()` which moves to the position of the start of the loop. To save that position I've added a `std::vector<int>` to the class, which will be used as a pile: Whenever a loop is started it's start position is pushed into the pile, whenever it reaches the end of the block it's position is poped out of the pile so that the interpreter knows where to return to.

With that, the class has the following added to it so far:

```cpp
class Interpreter {
    private:
        ...
+       void print();
+       bool testLoop();
+       void skipLoop();
+       void startLoop();
+       void endLoop();
    
    public:
        ...
}
```

### A few notes on implementation so far

I have gone for a simple way of knowing where the loops are. It's slightly better than actually looking for the loop's start every time, but it could be improved upon: by making a reading of the program first, the loop's start and end pairs could be saved so that not only getting to the end of the loop will immediately take you back to the start, but skipping a loop will immediately take you to the end of it, rather than active look for it the way it is done now. This is a change that is worth making, maybe later today or tomorrow, when I start working on validating a program before running it (so far the interpreter is taking the program at face value, simply skipping invalid operators and whatnot).

One thing to pay attention to if going for the "look for the loop's end" when skipping is making sure that the interpreter won't execute the loop's end itself as an operation, otherwise the head will be taken back to the last loop start, but since the loop was skipped that won't be defined, which will cause problems in the execution, potentially (and probably) causing an eventual segfault or other problem related to access of forbidden memory.

### The Interpreter Interface

So far I had a line in my `main.cpp` that called the `run` function of the interpreter on a string that contained a piece of code that tested whatever operation I was currently testing out. This meant that to test different programs I had to recompile the code and run it once again. This process would have been faster if, instead, I had gone for an interpreter interface, such as the one Python has. This would allow me to more quickly test different inputs to my interpreter, seeing how it behaves. Not only that, but once I started working on this interface it also brought up some problems with the way my code had been organized so far, name ly the fact that the `run` function was doing more than simply run, but also loading the code into the interpreter's memory. This meant that I would have to change the code to run, otherwise it would not be possible to use it for the interpreter interface, as it would overload the program at every turn, which is not how this kind of interface is supposed to work.

To fix this I added a new function, `void loadProgram(std::string)`, which is the function responsible for taking a program and loading it into memory. This is also one of the places where program vetting should occur, though I haven't gotten to that yet. Then, I changed the `void run()` function, taking out it's parameters and making it so that it runs from the current state (that it to say, current program, current pc, current tape, and current head position).

With that done the interpreter interface is very simple: it's an infinite loop that promps the user for input. It exits only once the entered input is the command `"exit"`, otherwise it takes the input as a fragment of BrainF\*ck code, appends it to the program, and runs it. This is the second place where input vetting should take place.

The final changes to the `Interpreter` class for today are

```cpp
class Interpreter {
    private:
        ...

    public:
        ...
-       void run(std::string);
+       void loadProgram(std::string);
+       void run();
+       void enterInterpreterInterface();
}
```

### Next

To finish the basic functioning of this interpreter I need to add the capability of passing it a file containing some BrainF\*ck code and have it immediately interpret it. After doing that I need to work on what could well be the most interesting and challenging part of this project as a whole: vetting a BrainF\*ck program to find errors and creating good error messages. After that it's all about optimizations and bug fixes.
