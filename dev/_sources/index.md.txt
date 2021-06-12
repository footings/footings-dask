
# footings-dask

*A parallel back end for footings using dask.*

![tests](https://github.com/footings/footings-dask/workflows/tests/badge.svg)
[![gh-pages](https://github.com/footings/footings-dask/workflows/gh-pages/badge.svg)](https://footings.github.io/footings-dask/main/)
[![license](https://img.shields.io/badge/License-BSD%203--Clause-blue.svg)](https://opensource.org/licenses/BSD-3-Clause)

## Summary

Footings-dask is a parallel back end that can be used to parallelize computation for models built using the [footings](https://footings.github.io/footings/master/index.html) framework. It is built off the popular [dask](https://dask.org/) library which is used to parallelize computations.

## Purpose

The [footings](https://footings.github.io/footings/master/index.html) framework was developed with the intention of making it easier to develop actuarial models in Python. Often times the computation behind an actuarial model can be considered an embarrassingly parallel problem where we have to do the same computation for many policyholders. Thus, model computation time can be improved by distributing the workload and running computations in parallel. The `footings-dask` library was created to make it easier to parallelize a model built using the `footings` framework.

## Installation

This library is not yet on PyPI. Thus, the library needs to be installed directly from github.

```
pip install git+https://github.com/footings/footings-dask.git
```

## Learning

The best way to learn how to use footings-dask is to checkout the [example](example.md).


## License

See [license](license.md) file.

```{toctree}
:maxdepth: 2
:hidden:

example.md
api.md
license.md
changelog.md
```
