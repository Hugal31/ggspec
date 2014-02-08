# ggspec - lightweight unit testing library for Guile

Copyright (c) 2014 Yawar Amin

See LICENSE file for details

GitHub, Reddit, Twitter: yawaramin

ggspec is a _very_ lightweight unit testing framework for Guile. I may
port it to other Schemes in future; in principle it should be fairly
simple because it doesn't use very many Guile-specific features.

ggspec is self-testing. In fact, it was bootstrapped by the tests--the
tests were written _first,_ before there was a unit testing framework to
run them; and _then_ the framework was written around the tests.

## Installation

Assuming you have cloned the `ggspec` repository to a directory also
named `ggspec`, create a symbolic link to the `ggspec` directory inside
any directory in your `$GUILE_LOAD_PATH` (Windows: `%GUILE_LOAD_PATH%`).

E.g., if you have `~/guile` in your Guile load path, and have cloned the
repo into `~/code/ggspec`, then run:

```
cd ~/guile
ln -s ~/code/ggspec
```

## Verifying the framework itself

As I described earlier, the `ggspec` framework is tested using itself.
You can run the tests:

```
cd ~/code/ggspec
./ggspec .
```

## Minimal complete example

```
$ guile
guile> (use-modules (my ggspec))
guile> (define demo-suite (suite "Hello ggspec" end))
guile> (demo-suite)
  Suite: Hello ggspec
(0 0)
```

The last line above shows the number of passing tests and the number of
failing tests. Since we didn't write _any_ tests here, but are zero.

## Introduction

See the `ggspec.scm` file for a detailed reference. A brief introduction
to how it works:

  - The `suite` function is presented as the way to organise everything.
    This includes tests, any options you want to pass in to those tests,
    any setup variables you want to define before running each test in
    the suite, and any teardowns you want to do after running each test.

  - Inside the `suite` function, group of tests are organised together
    using the `tests` function. Each test is created using the `test`
    macro.

  - After the tests come the options, which are organised togerher using
    the `options` function. Each option is created using the `option`
    function.

  - After the options come the setups, which are organised together
    using the `setups` function. Each setup (which is a symbol name
    bound to an expression) is created using the `setup` macro. Each
    setup will be re-evaluated before each test is run. So generally you
    don't want to put expensive computations in these.

  - After the setups come the teardowns, which are organised together
    using the `teardowns` function. Each teardown (which is just a set
    of expressions) is created using the `teardown` macro. Each teardown
    will be re-evluated _after_ each test is run to ensure a clean
    'environment' for the next test run.

Options and setups are not required, but since setups come after
options, you have to specify options if you want to specify setups. You
can specify empty options, setups, and teardowns, in which case the
`suite` function doesn't try to do anything with them.

## Tutorial

A short tutorial (in the context of Test-Driven Development):

Suppose we want to develop a function `sqr` that should return the
square of its argument (a number). In this example I will show the
function and its test suite together, but usually you would have them in
separate files.

First you write a failing test:

```scheme
(use-modules (my ggspec))

(define sqr-suite ; Nothing significant about the name sqr-suite.
  (suite "The sqr function"
    (tests
      (test "Should square 1 correctly"
        ;; The 'e' argument below represents (and contains) all the
        ;; state that we're passing in to the test. All setups, options,
        ;; etc.
        e
        ((e 'assert-equal) 1 (sqr 1))))))

(sqr-suite) ; Run the suite.
```

This suite will fail because the `sqr` function does not exist:

```
  Suite: The sqr function
ERROR: Unbound variable: sqr
```

Now implement it to make the test pass:

```scheme
(define (sqr x) 1)
```

The test will now pass because we have 'cheated' to make it so:

```
  Suite: The sqr function
    Test: Should square 1 correctly
    [PASS]
```

Now we extend the suite to another 'example':

```scheme
(use-modules (my ggspec))

(define sqr-suite ; Nothing significant about the name sqr-suite.
  (suite "The sqr function"
    (tests
      (test "Should square 1 correctly"
        e
        ((e 'assert-equal) 1 (sqr 1)))
      (test "Should square 2 correctly"
        e
        ((e 'assert-equal) 4 (sqr 2))))))

(sqr-suite)
```

This will now fail, again, because the `sqr` function is hard-coded to
always return `1`:

```
  Suite: The sqr function
    Test: Should square 1 correctly
    [PASS]
    Test: Should square 2 correctly
      Expected: 4
      Got: 1
    [FAIL]
```

Now we fix that:

```scheme
(define (sqr x) (* x x))
```

This time the test will pass because `sqr` handles the general case:

```
  Suite: The sqr function
    Test: Should square 1 correctly
    [PASS]
    Test: Should square 2 correctly
    [PASS]
```

