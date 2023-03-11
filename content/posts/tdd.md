---
title: "Test Driven Development in Rust and Python"
date: 2023-03-11T12:12:36+02:00
tags: ["whatilearned"]
draft: false
---

Test Driven Development (TDD) is a common best practice when developing
software. The idea behind it is to write the unit tests for the software before
and during development, instead of afterwards. It has empirically been observed
that this leads to shorter development times and less defects in the software.

A common method for TDD is "Red-Green-Refactor". This method consists of a
cycle of three steps:

1. Write a test that covers an aspect of the software you want to develop. If
   you run the test, it will fail (Red). Notice that also compilation and
   runtime errors should be considered failure.
2. Next, write enough business logic to make your test pass (green). Do not
   write anything else, focus on the small increments.
3. (If applicable) refactor and apply best practices to iterate to a mature
   solution.

Repeat until your feature set is completely tested and implemented.

## Test Driven Development in Rust

Rust has a convention that makes it particularly easy to practice TDD: The unit
tests are usually located in the same file as the business logic. This avoid
constant switching between files during TDD development, but also supports
during software maintenance, e.g. writing a regression test for a bug-fix.

For this example, I'm using the [bubble sort problem](https://www.hackerrank.com/challenges/ctci-bubble-sort/problem)
from Hackerrank.

The boilerplate code in Rust looks like this:

```rust
use std::io::{self, BufRead};

/*
 * Complete the 'count_swaps' function below.
 *
 * The function accepts INTEGER_ARRAY a as parameter.
 */

fn count_swaps(a: &[i32]) {

}

fn main() {
    let stdin = io::stdin();
    let mut stdin_iterator = stdin.lock().lines();

    let n = stdin_iterator.next().unwrap().unwrap().trim().parse::<i32>().unwrap();

    let a: Vec<i32> = stdin_iterator.next().unwrap().unwrap()
        .trim_end()
        .split(' ')
        .map(|s| s.to_string().parse::<i32>().unwrap())
        .collect();

    count_swaps(&a);
}
```

In the following, I will omit the `main` method, which just reads the `stdin`
input.

### Red Phase

To start with TDD, we need to write any test cases at all. For Hackerrank, I
usually use the provided examples and sample tests. The example tells, for a
given input of `[6,4,1]`, the output of the program shall be

```
Array is sorted in 3 swaps.
First Element: 1
Last Element: 6
```

So let's try to input that into an unit test. Note that due to the small scope
of Hackerrank exercises, this test already covers the whole business logic. In
a real world application, this can (and should) be a much smaller scope.

```rust
fn count_swaps(a: &[i32]) {

}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn example() {
        let output = count_swaps(&[6, 4, 1]);
    }
}
```

First we create a module named `test` for a test configuration, meaning it will
only be compiled when we explicitly run the tests via `cargo test`. Within the
module, all entities from the outer scope are imported. Lastly, a test function
`example` is created.

But wait, our program takes an array as input, but is required to print to
`stdout` as output. How to test that? The solution to this problem is to
refactor and extract the business logic to make it testable independently:

```rust

fn business_logic(a: &[i32]) -> (usize, i32, i32) {

}

fn count_swaps(a: &[i32]) {

}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn example() {
        assert_eq!(business_logic(&[6, 4, 1], (3, 1, 6));
    }
}
```

We created a testable method `business_logic` that returns the three values
that should be outputted. With that design, the logic becomes easily testable.
Also, we introduced an assertion into our test method, that check equality of
the output of `business_logic` and the provided example solution. Now we can
run `cargo test` and ... experience a compilation error: `error[E0308]:
mismatched types`. This is the "Red" phase of our TDD method, so we're on the
right way. Next, let's move to the Green phase and resolve our issue.

### Green Phase

To resolve the error we are facing, we need to provide a function body for
`business_logic` that returns a value matching the function signature. We can
easily do this:

```rust
fn business_logic(a: &[i32]) -> (usize, i32, i32) {
    (3, 1, 6)
}
```

Now, we can run `cargo test` and we can see that the test succeeds. Green phase
completed. At this point, we don't need to refactor and can directly move to
the next Red phase.

### Red Phase

Since it worked so well, we can now implement a second test that checks another
provided solution:

```rust
    #[test]
    fn example() {
        assert_eq!(business_logic(&[6, 4, 1], (3, 1, 6));
    }

    #[test]
    fn sample_test_case_0() {
        assert_eq!(business_logic(&[1, 2, 3]), (0, 1, 3));
    }
```

Running `cargo test` now shows us that something doesn't work as intended:
`thread 'tests::sample_test_case_0' panicked at 'assertion failed: (left ==
right)`. At this point, we should probably start thinking about a proper
business logic that solves our problem statement, as `(3, 1, 6)` will not
always be the answer we're searching.

### Green Phase

While repeating this pattern for a while, eventually we come up with the
following code:

```rust
fn business_logic(a: &[i32]) -> (usize, i32, i32) {
    let mut swaps = 0;
    let mut a_vec = a.to_vec();
    for _ in 0..a_vec.len() {
        for j in 0..a_vec.len() - 1 {
            if a_vec[j] > a_vec[j + 1] {
                a_vec.swap(j, j + 1);
                swaps += 1;
            }
        }
    }
    (swaps, *a_vec.first().unwrap(), *a_vec.last().unwrap())
}

fn count_swaps(a: &[i32]) {
    let (swaps, min, max) = business_logic(a);
    println!("Array is sorted in {swaps} swaps.");
    println!("First Element: {min}");
    println!("Last Element: {max}");
}
```

That is one of probably multiple solutions to the problem statement, and we
iterated to it using TDD! Also note that, in order to fulfill the requirements
of the task, the output should be printed to `stdout`, which is done in the
function `count_swaps` which serves as entry point for the main logic. That
function just calls our internal `business_logic` and prints the results.

This is a small example how to do TDD in Rust. Again, please note that the
scope was rather small and therefore almost no refactoring was necessary. In
bigger applications, it is usual to test different aspects of a program and
iterate and refactor until the requirements are met.

## Test Driven Development in Python

TDD in Python can be done quite similar as in Rust. It is even possible, though
not as much of a convention as in Rust, to write unit tests in the same file,
next to the business logic. As I mentioned above, I like that approach.

Unlike Rust, there is not one solution to implement and run tests provided by
the ecosystem (namely `cargo test`), though Python offers different modules
that can be used. Here, we're relying on `unittest`. An example how to write
unit tests could look like this:

```python
import unittest

def business_logic(a: List[int]) -> (int, int, int):
    [...]

class BubbleSortTest(unittest.TestCase):
    def test_example(self):
        self.assertEqual(business_logic([6, 4, 1]), (3, 1, 6))
```

Firstly, we have to import the `unittest` module to use it. Then, we declare a
class which inherits from `unittest.TestCase`. Within that class, we write
methods that serve as our unit tests. Note that test method names must always
start with `test`, else they won't get executed as part of the unit test. That
way, you can implement helpers in the test class that won't be executed
standalone. To run the unit tests, execute: `python3 -m unittest
bubble_sort.py`, which is the equivalent of `cargo test` in Rust.
