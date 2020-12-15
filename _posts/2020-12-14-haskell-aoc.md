---
title: 'Studying Haskell with the Advent of Code 2020'
subtitle: ''
category: tech
---

This year's [Advent of Code](https://adventofcode.com/2020/about) (AoC) is going on and like many other programmers I have been thinking about starting it (but not actually doing so) for the longest time. However, with a test approaching rapidly the urge to procrastinate became greater than ever. Wanting to make the procrastination at least a little bit productive, I decided to finally get started with the AoC using Haskell, since half of the test should be about functional programming, which was taught through Haskell. So, despite the fact that I wasn't actually *studying* I could see this as me getting even more familiarized with the language and the paradigm, which should help me move towards being better prepared for the test. My methodology was very simple: at first there was no methodology, but then I started about a methodology. The result was that at first I just wrote the answers to the puzzles in Haskell, and not much else. I was getting better acquainted with the language, but not much more. Then I decided to, well, write this post as way to think more critically about how this idea of studying functional programming and Haskell through the AoC was actually working. So I opened up the syllabus for the course, and started looking for places in my code where I could say I had made good use of the subjects that we had studied. Later I realized that, more than this, I could look at the subjects and see how I could use them to, perhaps, improve on my solutions to the puzzles.

*The full code for each of the puzzles I've done can be found on [GitHub](https://github.com/AugustoQueiroz/AoC20)*

## How this post is structured

After thinking about how to write this post I decided to do it day-by-day, as opposed to having one section for each subject on the course and then talking about how it was related to my solution in different days. While the latter would make more sense as a "didatic" device, the former allows readers who might be currently thinking about doing the AoC20 (let's be honest, anyone who is already doing it is waaay past me, probably) to not get any spoilers. Though, who am I kidding, if this post is ever read it's probably going to be way after the AoC20 is done. Still, I thought this way might be interesting.

## Disclaimer

### Not all days

Here I tried not to take on challenges that were too hard, because it didn't really seem to make sense. At least at first glance there are a good number of AoC20 puzzles that are not best fit to the functional programming paradigm. While they are certainly still doable, I decided to skip around and do first the ones where the paradigm seemed to fit best. Maybe afterwards I will come around and try to do even them in Haskell.

### Not the raw inputs

Because I had a really hard time even finding anything on the web about how to read a list of number, one per line, into Haskell, I ended up taking a short route and reformatting the input such that it is more easily readable by my program. This probably might mean that I'm not so strong on IO in Haskell, hopefully I will improve on that front and ultimately be able to take the raw input as it is given by the AoC puzzle.

#### Input reformatting with Vim

To reformat the inputs I just used Vim. I pasted the raw input given by AoC into vim and, where necessary, used `:%s/$/,` to add a coma after all lines, and `VGJ` to select all lines and collapse them into a single line. Finally, `A<backspace>]` and `I[` for adding brackets. When the input was a list of strings I would do `:%s/^\(.*\)$/"\1",` to add quotes around the lines as well as the comma. In this way I could get the input with a single `getLine` and parse it appropriately with `read input :: [Int]` or `read input :: [String]`.

## Day 1: Maybe

The [first puzzle](https://adventofcode.com/2020/day/1) asks us to find two integers in a list of integers that add up to a third integer (2020). My first impulse with this solution was to, for every integer `n` on the list, look to see if it's complement in regards to the expected sum `s` was also on the list (i.e. `s - n` is in the list). The coded for that ended up like this:

{% highlight hs linenos %}
twoSum :: [Int] -> Int -> Maybe (Int, Int)
twoSum [] _ = Nothing
twoSum (head : tail) sum
  | listHas tail (sum - head) = Just (head, sum - head)
  | otherwise = twoSum tail sum
{% endhighlight %}

Here we encounter already the first interesting piece of Haskell, the use of `Maybe` and `Nothing`. Altough for part 1 it could be argued that the result need not be a Maybe, it makes the code more elegant and "portable" (as in: if I ever need something like this again I can just copy-paste it). This will come in handy for part 2, as it asks us to find three numbers that sum up to 2020, and using `Maybe (Int, Int)` allows us to neatly reuse this function for defining `threeSum` without having to deal with awkward constant return values (such as `-1` to indicate that no such two values exist). With this the solution for part three looks like this:

{% highlight haskell linenos %}
threeSum :: [Int] -> Int -> Maybe (Int, Int, Int)
threeSum [] _ = Nothing
threeSum (head : tail) sum = do
  let twoSumResult = twoSum tail (sum - head)
  case twoSumResult of
    Just (n1, n2) -> Just (n1, n2, head)
    Nothing -> threeSum tail sum
{% endhighlight %}

<!--
### A better solution with List Comprehension & Filtering

- I actually have to solve the second solution so that it doesn't depend on no repeat values.
-->

## Day 2: Parsing Strings

On the [second day]() of AoC the goal is to, given a set of "passwords", along with their rules, count how many of the passwords follow the rules. Each element is given as a string of the form `i-j c: password`, where `i` and `j` are both integers, and `c` is a character. For part one we have to count `c` in `password` and make sure that the count is in `[i, j]`. For part two we have to make sure that *exactly one* of `password[i]` and `password[j]` is `c`. This gave me a good opportunity to work on string parsing. Although I definitely don't have the best solution, I think I arrived at an interesting one, as it's not complex and allows for readable code, imo. The idea here consists in getting a substring from a start index up to a separator char. For example: we can find `i` by getting the substring from index 0 up to '-', then we find `j` by getting the substring that starts at the index 1 after '-' and that goes until we find ' '. My solution was to make two functions: one that skips a given amount of chars and returns the string after that, and one that reads a string until a given char is found.

{% highlight haskell linenos %}
skip :: Int -> String -> String
skip 0 s = s
skip n (_ : tail) = skip (n-1) tail 

getStrUntil :: String -> Char -> String -> String
getStrUntil (head : tail) endChar numStr
  | head == endChar = numStr
  | otherwise = getStrUntil tail endChar (numStr ++ [head])
{% endhighlight %}

It feels like there should be a native function in haskell that does this, but I couldn't find it (??) so I wrote it like that to parallel haskell's native `take`. With these two I can parse one line of input as such:

{% highlight haskell linenos %}
parseInputLine :: String -> (Int, Int, Char, String)
parseInputLine s = do
  let minStr = getStrUntil s '-' ""
  let maxStr = getStrUntil (skip ((length minStr) + 1 s)) ' ' ""
  let char = head (skip ((length minStr) + (length maxStr) + 2) s)
  let password = skip ((length minStr) + (length maxStr) + 5) s
  (read minStr :: Int, read maxStr :: Int, char, password)
{% endhighlight %}

With this parsing done the other functions needed to solve the proposed problem are quite trivial.

## Days 3 & 4: Skipped

I skipped [day 3](https://adventofcode.com/2020/day/3) because it was a matrix problem, and afaik we didn't even see in the course how to deal with matrices with Haskell, so I decided to skip it, lest I try to do something too imperative in Haskell. [Day 4](https://adventofcode.com/2020/day/4) I skipped just because reading the input seemed like to much of a pita.

## Day 5: Nothing particularly of interest

[Day 5](https://adventofcode.com/2020/day/5) asks us to find a list of numbers that are somewhat complexly defined. Ultimately we need to find two numbers by executing a "binary-search-like" sequence of steps, then we perform some maths with those two numbers (`row * 8 + col`). Below is the code to find the two numbers (`row` and `col`).

{% highlight haskell linenos %}
binarySearch :: Int -> Int -> String -> Char -> Int
binarySearch lo _ [] _ = lo
binarySearch lo hi (head:tail) lowerChar
    | head == lowerChar = binarySearch lo ((hi + lo) `div` 2) tail lowerChar
    | otherwise         = binarySearch ((hi + lo + 1) `div` 2) hi tail lowerChar

findRow :: String -> Int
findRow seq = binarySearch 0 127 seq 'F'

findCol :: String  -> Int
findCol seq = binarySearch 0 8 seq 'L'
{% endhighlight %}

Once we have that list we have to find the missing number, as it should be a linear list. To do so we can sort the list, and then start counting up with the list. When the value on the list is not matched to our counter, the counter has the value missing.

{% highlight haskell linenos %}
findMissingSeat :: [Int] -> Int -> Int
findMissingSeat [] n = n
findMissingSeat (head:tail) n
    | head == n = findMissingSeat tail (n+1)
    | otherwise = n
{% endhighlight %}

The call to this function is done like so:

{% highlight haskell linenos %}
findMissingSeat ourList (head ourList)
{% endhighlight %}

## Days 6-8: Skipped

[Day 6](https://adventofcode.com/2020/day/6) I skipped doing in Haskell because, as for day 4, just reading the input seemed like it would be a pita. [Day 7](https://adventofcode.com/2020/day/7) as well. [Day 8](https://adventofcode.com/2020/day/8), although at first it seemed like it could be a very cool one to do in Haskell, I decided to skip because of the `jmp` instruction, which requires going back and forth on a list, and so I new I would end up doing something too imperative.

## Day 9: Nothing of particular interest

For [day 9](https://adventofcode.com/2020/day/9) the first challenge asks us to find the first number in a list that cannot be made up by a sum of two of the previous 25 numbers. Then we need to find a contiguous sublist that adds up to that number. For the first part we can reuse the code from day 1, but now we "move a window" through the list to limit where we can find the two numbers that add up:

{% highlight haskell linenos %}
findFirstInvalid :: [Int] -> [Int] -> Int
findFirstInvalid l (head:t)
    | twoSum l head == Nothing = head
    | otherwise = findFirstInvalid ((tail l) ++ [head]) t
{% endhighlight %}

We receive two lists, the first one is the 25 previous values, from the second one the head is the number we're currently considering, and the tail are the future numbers. We then see if there are two numbers in the first list that add up to the head of the second list. If so, that means it's valid, so we remove the head of the first list and append the head of the second, pass this new list along with the tail of the second list recursively to findFirstInvalid. For the second part the idea is that, again, we will roll a window through the list, we widen the window at one size when the sum is less than our target `s`, and we shorten it on the other side when the sum is greater than `s`. When the sum is exactly what we want, we check if the size of the window is greater than one (remember, the value `s` that we're checking for is, itself, on the list, so there is going to be a window of size 1 containing only it that will sum to it), if it is that is our value, otherwise we reset the window.

{% highlight haskell linenos %}
findContiguousWithSum :: [Int] -> Int -> [Int] -> [Int]
findContiguousWithSum (h:t) s l
    | sum l < s = findContiguousWithSum t s (l ++ [h])
    | sum l > s = findContiguousWithSum (h:t) s (tail l)
    | sum l == s && length l > 1 = l
    | otherwise = findContiguousWithSum (h:t) s []
{% endhighlight %}

## Day 10: Tuples and Pattern Matching

Like day 5, [day 10](https://adventofcode.com/2020/day/10) (part 1) didn't result in anything too interesting. That is because the bulk of the problem was in regards to having a small realization about what was being asked. The idea of the solution is to sort the input list, and then we just need to count how many adjacent numbers have difference 1, and how many have difference 3. The solution to this first part is so simple and small that I'll just put the whole thing below. Besides being a good way to see the value of tuples and pattern matching (which is something found in almost all of the solutions I coded up), it gets a shout for the import (though that, too, was already done for day 1).

{% highlight haskell linenos %}
import Data.List (sort)

day10 :: [Int] -> Int -> (Int, Int) -> (Int, Int)
day10 [] _ result = result
day10 (head:tail) lastValue (c1, c3)
    | head - lastValue == 1 = day10 tail head (c1+1, c3)
    | head - lastValue == 3 = day10 tail head (c1, c3+1)
    | otherwise = day10 tail head (c1, c3)

main :: IO ()
main = do
    input <- getLine
    let inputs = sort (read input :: [Int])
    let (c1, c3) = day10 inputs 0 (0, 0)
    print $ c1 * (c3+1)
{% endhighlight %}

## Day 11: Skipped

[Day 11](https://adventofcode.com/2020/day/11)'s puzzle is another that deals with matrices, so I skipped it as well.

## Day 12: Tuples & Partial Folding

The solution to [day 12]() is another great example of the use of tuples and pattern matching. Here we can encode the state of the ship (for part 1) as a 3-tuple `(x, y, rotationAngle)`. Then we can "perform" the movements by taking the ship's state, the movement we want to make, and calculating the changes. One really cool thing from this question relates to Partial Function Application (when you give only part of the parameters to a function, which "results" in a new function that can later have the missing parameters passed to): We get a list of movements that need to be done. Firstly we define a function to perform one movement, and then we write another function that goes through the input list calling out first function with each movement. Although I had written this by myself, my haskell extension on VS Code suggested a change that I just found so damn beautiful and elegant that it makes the whole of functional programming even more beautiful to me. The suggestion was:

{% highlight haskell linenos %}
performMovements :: (Int, Int, Int) -> [(Char, Int)] -> (Int, Int, Int)
performMovements = foldl performMovement
{% endhighlight %}

The idea is to use haskell's `foldl`, which starts of with a given value and a list, then "updates" the value by calling a given function on each element of the list, with the value as a parameter. Here "updates" is, of course, a figure os speach only, because immutability. In this case we create the function `performMovements` by saying that we will be a `foldl` operation, where the function we want to use for the folding is `performMovement`, but the start values and list will be given later.