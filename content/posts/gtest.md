---
title: "What I learned: Google Test"
date: 2018-10-27T13:12:36+02:00
tags: ["whatilearned"]
draft: false
---

I have tackled a small C++ project recently for an interview process and used
the chance to learn more about testing. I have mostly used Python `unittest` in
the past, both for Python and C code. Together with `ctypes` it is a nice
solution to test C against Python code.

Now I wanted to have a deeper look into Google Test in combination with CMake.

## Setting up Google Test
I compiled Google Test myself, which is pretty easy with the repo on
[Github](https://github.com/google/googletest). This will install headers and
libraries necessary for C++ testing in your projects.

An easy example of how to use Google Test can be found
[here](https://gist.github.com/mawenbao/9223908). I use one single source file
to run all my tests:

```c++
#include <gtest/gtest.h>

int main(int argc, char **argv) {
    ::testing::InitGoogleTest(&argc, argv);
    return RUN_ALL_TESTS();
}

const char *actualValTrue  = "hello gtest";
const char *expectVal      = "hello gtest";

TEST(StrCompare, CStrEqual) {
    EXPECT_STREQ(expectVal, actualValTrue);
}
```

## Integrate GTest in CMake

I wanted a solution to work together with CMake. When I run `make test` it
should run the GTest tests and report the results. Also, I wanted the tests to
be ignored if GTest is not available on a machine. I came up with this
CMakeLists.txt.

```cmake
cmake_minimum_required(VERSION 3.9)
project(test_with_gtest)
add_library(test_with_gtest SHARED code_to_be_tested.cpp)
find_package(GTest)
if(${GTEST_FOUND})
    message("GTest found, enabling testing")
    enable_testing()
    add_subdirectory(test)
endif(${GTEST_FOUND})
```

This will build a shared library with name `test_with_gtest` based on the
source file `code_to_be_tested.cpp`. If Google Test is installed/found,
additionally, the `test` directory is added to the project and testing is
enabled. The `CMakeLists.txt` of the `test` folder looks like this.

```cmake
include(GoogleTest)
add_executable(google_tests test_cases.cpp)
include_directories(${CMAKE_BINARY_DIR} ${gtest_SOURCE_DIR}/include ${gtest_SOURCE_DIR})
target_link_libraries(google_tests gtest gtest_main test_with_gtest)
gtest_add_tests(TARGET google_tests AUTO)
```
This creates a new executable target `google_tests` from source file
`test_cases.cpp` (see first code section). This code links against the library
which contains the code to be tested.

Now, if Google Test is found, I can compile my project and run tests like this.

```bash
$ cmake ..
$ make -j4
$ make test
```
However, this is just a CMake summary of the tests. If something goes wrong, it
is nice to have a more verbose output by directly running the executable
performing the tests.

```bash
$ ./test/google_tests
```
## Test Fixtures
For more extensive testing, it is preferrable to use reproducable sets of input
data. This is where fixtures come in handy. A fixture is a class providing
logic to prepare (`SetUp()`) and clean up (`TearDown()`) tests. This logic gets
called before each test and enables multiple tests on the same input state.
An example can be
```c++
#include <gtest/gtest.h>
#include "../code_to_be_tested.h"

int factor;

class rangeAllocatorTests: public ::testing::Test {
  protected:
    void SetUp() {
        factor = 7;
    }

    void TearDown() {
    }
};

int main(int argc, char **argv) {
  ::testing::InitGoogleTest(&argc, argv);
  return RUN_ALL_TESTS();
}

TEST_F(rangeAllocatorTests, testAddition) {
  int result = 4 * factor;
  EXPECT_EQ(result, 28);
}
```
With this code, `factor` will always be set to `7` before the actual test
happens. Of course, this is a very simple example and in most cases you want to
have a more complex `SetUp()` routine.
