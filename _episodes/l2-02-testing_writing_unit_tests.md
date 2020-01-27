---
title: "Writing unit tests"
teaching: 40
exercises: 60
questions:
  - "What is a unit test?"
  - "How do I write and run unit tests?"
  - "How can I avoid test duplication and ensure isolation?"
  - "How can I run tests automatically and measure their coverage?"
objectives:
  - "Implement effective unit tests using pytest"
  - "Execute tests in Visual Studio Code"
  - "Explain the issues relating to non-deterministic code"
  - "Implement mocking, fixtures and test parametrisation using pytest decorators"
  - "Configure git hooks and pytest-coverage"
  - "Apply best-practices when setting up a new Python project"
  - "Recognise analogous tools for other programming languages"
  - "Apply testing to Jupyter notebooks"
keypoints:
  - "Python has a built-in testing framework, but pytest is not only more extensible, it's also more concise even for the simplest case"
  - "Testing with VS Code is fairly frictionless and encourages good habits (writing tests as you code, test-driven development)"
  - "Seeding random number generators and adding tolerances can help to test non-deterministic code (though presents challenges)"
  - "It is important to have a set-up you can use for every project - so that it becomes as routine in your workflow as version control itself"
  - "pytest has a myriad of extensions that are worthy of mention such as Jupyter, benchmark, Hypothesis, tox etc"
  - "Testing is not only standard practice in mainstream software engineering, it also provides distinct benefits for any non-trivial research software"
  - "Adding unit tests can verify the correct of software, but also its improve its structure: isolating logical distinct code for testing often involves untangling complex structures"
---

## Introduction

Unit testing validates, in isolation, functionally independent components of a program.

In this lesson we'll demonstrate how to write and execute unit tests for a simple scientific code.

This involves making some technical decisions...

### Test frameworks

We'll use `pytest` as our test framework. It's powerful but also user friendly.

For comparison: testing using `assert` statements:

```python
from temperature import to_fahrenheit

assert to_fahrenheit(30) == 86
```

Testing using the built-in `unittest` library:

```python
from temperature import to_fahrenheit
import unittest

class TestTemperature(unittest.TestCase):
    def test_to_farenheit(self):
        self.assertEqual(to_fahrenheit(30), 86)
```

Testing using `pytest`:

```python
from temperature import to_fahrenheit

def test_answer():
    assert to_fahrenheit(30) == 86
```

Why use a test framework?

- Avoid reinventing the wheel - frameworks such as pytest provide lots of convenient features
- Standardisation leads to better tooling, easier onboarding etc

Projects that use pytest:

- [numpy](https://github.com/numpy/numpy/blob/master/README.md) ([example](https://github.com/numpy/numpy/blob/master/numpy/core/tests/test_umath.py#L220))
- [pandas](https://github.com/pandas-dev/pandas)
- [SciPy](https://github.com/scipy/scipy)
- [Devito](https://github.com/devitocodes/devito/blob/master/README.md) ([tests](https://github.com/devitocodes/devito/tree/master/tests))

### Code editors

We've chosen [Visual Studio Code](https://code.visualstudio.com/) as our editor. It's free, open source, cross-platform and has excellent Python (and pytest) support. It also has built-in Git integration, can be used to edit files on remote systems (e.g. HPC), and handles Jupyter notebooks (plus many more formats).

> ## Demonstration of pytest + VS Code + coverage
>
> - Test annotations, status indicators and ability to run tests inline
> - Test perspective and Test Output
> - Increasing coverage (`assert recursive_fibonacci(7) == 13`)
> - Test-driven development: adding a new test (`test_negative_number_`) and fixing it (`test_negative_number`)
{: .callout}

## A tour of pytest

### Checking for exceptions

If a function invocation is expected to throw an exception it can be wrapped with a pytest `raises` block:

```python
def test_non_numeric_input():
    with pytest.raises(TypeError):
        recursive_fibonacci("foo")
```

### Parametrisation

Similar test invocations can be grouped together to avoid repetition. Note how the parameters are named and "injected" into the test function:

```python
@pytest.mark.parametrize("number,expected", [(0, 0), (1, 1), (2, 1), (3, 2)])
def test_initial_numbers(number, expected):
    assert recursive_fibonacci(number) == expected
```

This corresponds to running the _same_ test with _different_ parameters, and is our first example of a pytest decorator (`@pytest`).

### Skipping tests and ignoring failures

Sometimes it is useful to skip tests (conditionally or otherwise), or ignore failures (for example if you're in the middle of refactoring some code).

This can be achieved using other `@pytest.mark` annotations e.g.

```python
@pytest.mark.skipif(sys.platform == "win32", reason="does not run on windows")
def test_linux_only_features():
    ...

@pytest.mark.xfail
def test_unimplemented_code():
    ...
```

### Fixtures

If multiple tests require access to the same data, or a resource that is expensive to initialise, then it can be provided via a fixture. These can be cached by modifying the scope of the fixture. See this example from Devito:

```python
@pytest.fixture
def grid(shape=(11, 11)):
    return Grid(shape=shape)

def test_forward(grid):
    grid.data[0, :] = 1.
    ...

def test_backward(grid):
    grid.data[-1, :] = 7.
    ...
```

This corresponds to providing the _same_ parameters to _different_ tests.

### Tolerances

It's common for scientific codes to perform estimation by simulation or other means. pytest can check for approximate equality:

```python
from pytest import approx

def test_approximate_pi():
    assert 22/7 == approx(math.pi, abs=1e-2)
```

### doctest

pytest has automatic integration with the Python's standard [doctest](https://docs.python.org/3/library/doctest.html) module. This is a nice way to provide examples of how to use a library, via interactive examples in docstrings:

```python
def recursive_fibonacci(n):
    """Return the nth number of the fibonacci sequence

    >>> recursive_fibonacci(7)
    13
    """
    return n if n <=1 else recursive_fibonacci(n - 1) + recursive_fibonacci(n - 2)
```

_Hint: If your simulation or approximation technique depends on random numbers then consistently seeding your generator can help with testing. See [`random.seed()`](https://docs.python.org/3/library/random.html#random.seed) for an example._

## Hands-on unit testing

> ## Getting started
>
> ### Get our example code
>
> 1. Download and extract [this zip file](https://github.com/ImperialCollegeLondon/diffusion/archive/master.zip)
> 1. Make the folder your working directory and run the following commands:
>
> ```bash
> python3 -m venv .venv
> source .venv/bin/activate  # or `.venv\Scripts\activate.bat` if you're using Windows
> python -m pip install -U pip
> python -m pip install -U -r requirements-dev.txt
> ```
>
> ### Set up our editor
>
> 1. Download, install and run [Visual Studio Code](https://code.visualstudio.com/download)
> 1. Go to **File > Open Folder...** and find the files you just extracted
> 1. If you see an alert **This workspace has extension recommendations.** then click "Install All"
> 1. Open `test_diffusion.py`
> 1. You should now be able to click on **Run Test** above the `test_heat()` function and see a warning symbol appear, indicating that the test is currently failing
> 1. Switch to the **Test** perspective by clicking on the flask icon on the left-hand toolbar. From here you can **Run All Tests**, and **Show Test Output** to view the coverage report
{: .challenge}

### Introduction to your challenge

You have been lucky enough to inherit some code from a previous member of your group. They went to the trouble of writing a test but didn't have time to debug their program. Your job is to refactor the code and write some extra tests in order to identify and fix the problem, and make it more robust.

The code implements a numerical solver for the heat equation:

![\frac{\partial\phi}{\partial t}=\alpha\frac{\partial^2\phi}{\partial x^2},\;\;\;\;\;0\leq x\leq L,\;\;t\geq 0](https://latex.codecogs.com/png.latex?\frac{\partial\phi}{\partial&space;t}=\alpha\frac{\partial^2\phi}{\partial&space;x^2},\;\;\;\;\;0\leq&space;x\leq&space;L,\;\;t\geq&space;0)

A finite-difference approximation can be calculated as follows:

![\phi_{i}^{m+1}=r\phi_{i+1}^{m}+(1-2r)\phi_{i}^{m}+r\phi_{i-1}^{m}\;\;\;\;\;\;\;\;(1)](https://latex.codecogs.com/png.latex?\phi_{i}^{m&plus;1}=r\phi_{i&plus;1}^{m}&plus;(1-2r)\phi_{i}^{m}&plus;r\phi_{i-1}^{m}\;\;\;\;\;\;\;\;(1))

where `m` designates the time step and `r` is defined as follows: ![r=\frac{\alpha \Delta t}{\Delta x^2}](https://latex.codecogs.com/png.latex?r=\frac{\alpha\Delta&space;t}{\Delta&space;x^2})

The test provided (`test_heat`) compares the approximation with the exact solution for the relevant initial/boundary conditions. This test is correct but is failing - suggesting that there is a bug in the code.

> ## Testing (and fixing!) the code
>
> ### Refactoring
>
> First we'll simplify the code by modularising it. We'll extract the code that performs a single time step into a new function that can be verified in isolation via a new unit test:
>
> 1. In `diffusion.py` move the logic that updates `u` within the loop in the `heat()` function to a new top-level function `step(u, dx, dt, alpha)`. _Hint: the loop in `heat()` should now look like this:_
>
>    ```python
>    for t in range(nt - 1):
>        u = step(u, dx, dt, alpha)
>    ```
>
> 2. Run the existing test to ensure that it still executes - albeit with a failure.
> 3. Add a new (failing!) test called `test_step()` to `test_diffusion.py` in order to focus on our bug. It should call `step()` with some suitable arguments for `u`, `dx`, `dt` and `alpha` and compare the result with that suggested by equation (1) above. Use `approx` if necessary. _Hint: `step([0, 1, 0], 0.04, 0.02, 0.01)` is one suitable example_.
> 4. Assuming that this test fails, fix it by changing the code in the `step()` function to match the equation - correcting the original bug. Once you've done this all the tests should pass.
>
> ### More tests
>
> Now we'll add some further tests to ensure the code is more suitable for publication:
>
> 1. We want the `step()` function to raise an `Exception` when the following stability condition _isn't_ met: ![r\leq\frac{1}{2}](https://latex.codecogs.com/png.latex?r\leq\frac{1}{2}) Add a new test `test_step_unstable`, similar to `test_stable` but that invokes `step` with an `alpha` equal to `0.1` and expects an `Exception` to be raised.
> 1. Parametrise `test_heat()` to ensure the approximation is valid for some other combinations of `L` and `tmax` (ensuring that the stability condition remains true).
> 1. Review the Test Output and check the coverage of your tests. If you have completed the previous two steps it should be 100%.
>
> ### Bonus task(s)
>
> - TODO: write a doctest compatible docstring for `step()` or `heat()`?
> - TODO: use a mock to test `heat()` without testing `step()`?
{: .challenge}

## Advanced topics

### More pytest plugins

- [`pytest-benchmark`](https://pytest-benchmark.readthedocs.io/en/stable/) provides a fixture that can transparently measure _and track_ performance while running your tests:

```python
def test_fibonacci(benchmark):
    result = benchmark(fibonacci, 7)
    assert result == 13
```

> ## pytest-benchmark example
>
> Demonstration of performance regression via recursive and formulaic approaches to Fibonacci calculation
{: .callout}

- [`pytest-notebook`](https://pytest-notebook.readthedocs.io/en/latest/) can check for regressions in your Jupyter notebooks (see also [Jupyter CI](https://github.com/mwoodbri/jupyter-ci))

### Related tools

- [tox](https://tox.readthedocs.io/en/latest/) provides a means of testing code using different versions of Python and/or libraries. It essentially acts as wrapper for pytest (and alternative test frameworks)
- [Hypothesis](https://hypothesis.readthedocs.io/en/latest/) provides property-based testing, which is useful for verifying edge cases):

```python
from fibonacci import recursive_fibonacci
from hypothesis import given, strategies

@given(strategies.integers())
def test_recursive_fibonacci(n):
    phi = (5 ** 0.5 + 1) / 2
    assert recursive_fibonacci(n) == int((phi ** n - -phi ** -n) / 5 ** 0.5)
```

## Best practice, collaboration and automation

- Typing:
  - [Type hints](https://docs.python.org/3/library/typing.html)
  - Validation ([mypy](http://mypy-lang.org/))
- Version control and testing:
  - [`git bisect`](https://git-scm.com/docs/git-bisect)
  - [`pre-commit`](https://pre-commit.com/)
- CI:
  - GitHub Actions (see [ImperialCollegeLondon/fibonacci](https://github.com/imperialcollegelondon/fibonacci))

## Wrap-up

### What we've learnt

TODO

### Testing in other languages

- We've discussed tools and approaches for Python but analogues exist for other languages
- We (the RCS) have had success with:
  - C++: [Catch2](https://github.com/catchorg/Catch2)
  - Fortran: [pFUnit](https://github.com/Goddard-Fortran-Ecosystem/pFUnit)
  - R: [testthat](https://github.com/r-lib/testthat)
- See the Software Sustainability Institute's [Build and Test Examples](https://github.com/softwaresaved/build_and_test_examples) for many more

### Further resources

- [`ImperialCollegeLondon/pytest_template_application`](https://github.com/ImperialCollegeLondon/pytest_template_application)
- [A tried-and-tested workflow for software quality assurance](https://doi.org/10.5281/zenodo.1409199) ([repo](https://gitlab.com/mwoodbri/rse18))
- [Using Git to Code, Collaborate and Share](https://www.imperial.ac.uk/study/pg/graduate-school/students/doctoral/professional-development/research-computing-data-science/courses/git-to-code-callobrate-share/)
- RCS [courses](https://wiki.imperial.ac.uk/display/HPC/Courses) and [clinics](https://www.imperial.ac.uk/admin-services/ict/self-service/research-support/rcs/support/attend-a-clinic/)
- [Research Software Community](https://www.imperial.ac.uk/computational-methods/rse/)

{% include links.md %}