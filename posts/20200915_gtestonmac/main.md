---
Keywords: gtest, c++
Copyright: (C) 2020 Riki Otaki
---

# Installing google test on mac

Today I'm going to introduce how to install Google Testing Framework for GNU C++ especially on macOS Catalina (10.15.5).

## TL;DR

1. install cmake
2. build gtest with cmake
3. move headers and libraries to your path
4. `g++ test.cpp -pthread -lgtest_main -lgtest`


## Install and setup cmake

cmake is a tool for building packages written in C/C++. Install it by:

```sh
brew install cmake
```

```sh
| => cmake --version
cmake version 3.18.2
```

The default compiler that cmake uses to compile projects in macOS is **clang**. If you want to use gcc instead of clang you need to add some options.
Check your C/C++ compiler by

```sh
| => gcc --version
gcc (Homebrew GCC 10.2.0) 10.2.0
Copyright (C) 2020 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

| => g++ --version
g++ (Homebrew GCC 10.2.0) 10.2.0
Copyright (C) 2020 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

Also their locations

```sh
| => which g++
/usr/local/bin/g++
| => which gcc
/usr/local/bin/gcc
```

I am using GNU compiler so I need to add the following options when using cmake. 

```sh
alias cmake='cmake -DCMAKE_C_COMPILER=/usr/local/bin/gcc -DCMAKE_CXX_COMPILER=/usr/local/bin/g++'
```

I recommend you to add this to your `~/.bash_profile` if you are not using the default clang compiler. Don't forget to activate it by `source ~/.bash_profile`.

## Install gtest

Clone the googletest project from github and build it using cmake.
It is important to have set `alias cmake='cmake -DCMAKE_C_COMPILER=${YOURGCCPATH} -DCMAKE_CXX_COMPILER=${YOURG++PATH}'` when building otherwise googletest will only work for the default compiler **clang**.

```sh
git clone https://github.com/google/googletest.git
cd googletest
mkdir build
cd build
cmake .. # this is actually 'cmake -DCMAKE_C_COMPILER=/usr/local/bin/gcc -DCMAKE_CXX_COMPILER=/usr/local/bin/g++ ..'
make
```

Then install the headers and libraries to a local folder. I use `/usr/local/include` and `/usr/local/llb` respectively.

```sh
sudo cp -r ./googletest/googlemock/include/gmock /usr/local/include/gmock
sudo cp -r ./googletest/googletest/include/gtest /usr/local/include/gtest
sudo cp ./googletest/build/lib/*.a /usr/local/lib/
```

Check if your g++ compiler recognizes the path where you have put your include files.

```sh
echo | gcc -E -Wp,-v -
...
#include "..." search starts here:
#include <...> search starts here:
 /usr/local/Cellar/gcc/10.2.0/lib/gcc/10/gcc/x86_64-apple-darwin19/10.2.0/include
 /usr/local/Cellar/gcc/10.2.0/lib/gcc/10/gcc/x86_64-apple-darwin19/10.2.0/include-fixed
 /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include
 /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/System/Library/Frameworks
End of search list.
# 1 "<stdin>"
# 1 "<built-in>"
# 1 "<command-line>"
# 1 "<stdin>"
```

In my environment, `/usr/local/include` was not recognized. So, I added to the following lines to my `~/.bash_profile`.

```sh
export C_INCLUDE_PATH="/usr/local/include" 
export CPLUS_INCLUDE_PATH="/usr/local/include"
export LIBRARY_PATH="/usr/local/lib/"
```

Then activate the profile by `source ~/.bash_profile`.
Check again.

```sh
| => echo | gcc -E -Wp,-v -
...
#include "..." search starts here:
#include <...> search starts here:
 /usr/local/include # <- Added!
 /usr/local/Cellar/gcc/10.2.0/lib/gcc/10/gcc/x86_64-apple-darwin19/10.2.0/include
 /usr/local/Cellar/gcc/10.2.0/lib/gcc/10/gcc/x86_64-apple-darwin19/10.2.0/include-fixed
 /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include
 /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/System/Library/Frameworks
End of search list.
# 1 "<stdin>"
# 1 "<built-in>"
# 1 "<command-line>"
# 1 "<stdin>"
```

Now the environment setup is done and you can use google test.

## Use google test

Create `test.cpp` with the following contents;

```sh
#include <gtest/gtest.h>

TEST(TestCaseName, TestName){
    EXPECT_EQ(1, 1);
}
```

See if you can compile this code.

```sh
g++ test.cpp -pthread -lgtest_main -lgtest
```

Then

```sh
| => ./a.out
Running tests...
Test project XXX
    Start 1: test01
1/1 Test #1: test01 ...........................   Passed    0.47 sec

100% tests passed, 0 tests failed out of 1

Total Test time (real) =   0.48 sec
```

If you failed to compile, check if it is
- linking error 
  - check the path of headers and libraries
- not a linking error
  - check if `clang++ test.cpp -pthread -lgtest_main -lgtest` work. If this works, then reinstall google test but this time make sure that you have set `alias cmake='cmake -DCMAKE_C_COMPILER=${YOURGCCPATH} -DCMAKE_CXX_COMPILER=${YOURG++PATH}'` before building googletest.

Next post will be about **using** googletest in cmake.
