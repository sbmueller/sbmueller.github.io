---
title: "What I Learned: Clang Tools"
date: 2020-06-10T11:28:09+02:00
tags: ["whatilearned"]
draft: false
---

I have recently pivoted professionally and became a C++ developer. Due to that
change, I wanted to revisit my tooling landscape that I use for everyday
development. In particular, I want to share how I keep my code quality up
without any manual effort and at zero dynamic cost by using static code
analysis.

# Static Code Analysis/Linting
To perform a static code analysis means to search for programming errors and
issues before any code is executed (like in a unit test). It's the first
measure that the computer itself can take to improve code quality. The best
thing is: static code analysis can run automatically once set up with your IDE
and therefore provide checks and guidance with no additional effort on the
programmer's side.

## Set-Up
There are various static code analyzing tools for all kinds of programming
languages, but I'm going to focus on `clang-tidy` here. It's part of the LLVM
toolchain and a standalone tool that you can use by just executing
```
clangtidy main.cpp
```
However, since manual checks are inconvenient and can be forgotten, it is
recommended to integrate `clang-tidy` into your IDE. I'm using vim as my IDE
and make use of a plugin called [ALE](https://github.com/dense-analysis/ale).
This plugin just makes sure to run any analyzing tools of my choice
asynchronously in the background while I'm coding and I'm sure there is similar
functionality in other IDEs as well. ALE automatically checks code on open and
save actions but can be configured more precisely.

As part of the LLVM toolchain, `clang-tidy` needs a compilation database to
work properly. This is a small JSON file that specifies which file is going to
be compiled with what compiler command, to identify the C++ standard used, the
directories included, etc. Only with that information, reliable linting is
possible. For CMake projects, the creation of such a `compile_commands.json`
file is straightforward, by setting the CMake variable
`CMAKE_CREATE_COMPILATION_DATABASE` to `ON`. The file will be created in the
build directory and I usually symlink it into the project root, since all clang
tools search for it in the working directory and its parent directories.

### CMake
It is possible to set up CMake to run `clang-tidy` during the build process with
CMake 3.7.2 or later. Just set the variable `CMAKE_CXX_CLANG_TIDY` to
`"clang-tidy;-checks=*"`.

## Example
Let's assume this very basic definition of a C++ class:
```cpp
class Base {
public:
  Base();
  ~Base();
  Base(const Base &other);
};
```
Let's execute `clang-tidy` in the terminal here. I enabled all checks for this
example, which normally will return a lot of output when executed on a bigger
source code file:
```
$ clang-tidy --checks="*" main.cpp
2 warnings generated.
/Users/senpo/src/test/main.cpp:1:7: warning: class 'Base' defines a non-default destructor and a copy constructor but does not define a copy assignment operator, a move constructor or a move assignment operator [hicpp-special-member-functions]
class Base {
      ^
```
`clang-tidy` is warning us that we are violating the
[Rule of Three](https://en.cppreference.com/w/cpp/language/rule_of_three),
which tells us if a class has a user-defined copy constructor, copy assignment
operator or destructor, all three of them shall be implemented. The rule of
five extends this statement for the move constructor and the move assignment
operator. This issue will probably not raise a compilation or runtime error,
but as best practice can avoid exhausting bug searches in the future. So even
after compilation and unit-testing, this kind of issue would have remained
undetected without linting!

This is great and we can improve our code from here. With IDE integration, you
even get these messages directly while writing the code, without running
`clang-tidy` manually:

![clang tools in vim](/images/clang.png)

# Code Formatting
Besides linting, clang also offers a tool to properly format source code files,
named `clang-format`. By running it on a file, it will output a formatted
version of that source code file. To directly change the content of the file,
the `-i` flag can be used.
Let's look at this example:

```cpp
#include<iostream>

int main(){

    
    std::cout << "Hello, "
      << "World!" << std::endl;
    return
      0; }
```
That's a badly formatted code. Let's see what `clang-format` suggests:
```
$ clang-format main.cpp
#include <iostream>

int main() {

  std::cout << "Hello, " << "World!" << std::endl;
  return 0;
}
```
Various coding styles can be set via the --style flag, including own
definitions that are usually stored in a `.clang-format` file in the project
root. For example, have a look at the
[GNU Radio clang-format file](https://github.com/gnuradio/gnuradio/blob/master/.clang-format).

As with linting, this tool can be directly integrated into your IDE, for
instance, to format with a hotkey or to format before each save. This will make
sure you always stay compliant with your coding guidelines without any effort
from your side!
