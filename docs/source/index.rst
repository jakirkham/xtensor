.. Copyright (c) 2016, Johan Mabille and Sylvain Corlay

   Distributed under the terms of the BSD 3-Clause License.

   The full license is in the file LICENSE, distributed with this software.

xtensor
=======

Multi-dimensional arrays with broadcasting and lazy computing.

Introduction
------------

`xtensor` is a C++ library meant for numerical analysis with multi-dimensional array expressions.

`xtensor` provides

 - an extensible expression system enabling **lazy broadcasting**.
 - an API following the idioms of the **C++ standard library**.
 - tools to manipulate array expressions and build upon `xtensor`.

The implementation of the containers of `xtensor` is inspired by `NumPy`_, the Python array programming library. **Adaptors** for existing data structures to be plugged into our expression system can easily be written. In fact, `xtensor` can be used to **process numpy data structures inplace** using Python's `buffer protocol`_.

`xtensor` requires a modern C++ compiler supporting C++14. The following C+ compilers are supported:

 - On Windows platforms, Visual C++ 2015 Update 2, or more recent
 - On Unix platforms, gcc 4.9 or a recent version of Clang

Licensing
---------

We use a shared copyright model that enables all contributors to maintain the
copyright on their contributions.

This software is licensed under the BSD-3-Clause license. See the LICENSE file for details.

Usage
-----

Basic Usage
~~~~~~~~~~~

**Initialize a 2-D array and compute the sum of one of its rows and a 1-D array.**

.. code::

    #include <iostream>
    #include "xtensor/xarray.hpp"
    #include "xtensor/xio.hpp"

    xt::xarray<double> arr1
      {{1.0, 2.0, 3.0},
       {2.0, 5.0, 7.0},
       {2.0, 5.0, 7.0}};

    xt::xarray<double> arr2
      {5.0, 6.0, 7.0};

    xt::xarray<double> res = xt::make_xview(arr1, 1) + arr2;

    std::cout << res;

Outputs:

.. code::

   {7, 11, 14}

**Initialize a 1-D array and reshape it inplace.**

.. code::

    #include <iostream>
    #include "xtensor/xarray.hpp"
    #include "xtensor/xio.hpp"

    xt::xarray<int> arr
      {1, 2, 3, 4, 5, 6, 7, 8, 9};

    arr.reshape({3, 3});

    std::cout << arr;

Outputs:

.. code::

    {{1, 2, 3},
     {4, 5, 6},
     {7, 8, 9}}

**Broadcasting the ``xt::pow`` universal functions.**

.. code::

    #include <iostream>
    #include "xtensor/xarray.hpp"
    #include "xtensor/xmath.hpp"
    #include "xtensor/xio.hpp"

    xt::xarray<double> arr1
      {1.0, 2.0, 3.0};

    xt::xarray<unsigned int> arr2
      {4, 5, 6, 7};

    arr2.reshape({4, 1});

    xt::xarray<double> res = xt::pow(arr1, arr2);

    std::cout << res;

Outputs:

.. code::

    {{1, 16, 81},
     {1, 32, 243},
     {1, 64, 729},
     {1, 128, 2187}}

Lazy Broadcasting with `xtensor`
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We can operate on arrays of different shapes of dimensions in an elementwise fashion. Broadcasting rules of xtensor are similar to those of NumPy_ and libdynd_.

Broadcasting rules
^^^^^^^^^^^^^^^^^^

In an operation involving two arrays of different dimensions, the array with the lesser dimensions is broadcast across the leading dimensions of the other.

For example, if ``A`` has shape ``(2, 3)``, and ``B`` has shape ``(4, 2, 3)``, the result of a broadcasted operation with ``A`` and ``B`` has shape ``(4, 2, 3)``. 

.. code::

       (2, 3) # A
    (4, 2, 3) # B
    ---------
    (4, 2, 3) # Result

The same rule holds for scalars, which are handled as 0-D expressions. If `A` is a scalar, the equation becomes:

.. code::

           () # A
    (4, 2, 3) # B
    ---------
    (4, 2, 3) # Result

If matched up dimensions of two input arrays are different, and one of them has size ``1``, it is broadcast to match the size of the other. Let's say B has the shape ``(4, 2, 1)`` in the previous example, so the broadcasting happens as follows:

.. code::

       (2, 3) # A
    (4, 2, 1) # B
    ---------
    (4, 2, 3) # Result

Universal functions, Laziness and Vectorization
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

With `xtensor`, if ``x``, ``y`` and ``z`` are arrays of *broadcastable shapes*, the return type of an expression such as ``x + y * sin(z)`` is **not an array**. It is an ``xexpression`` object offering the same interface as an N-dimensional array, which does not hold the result. **Values are only computed upon access or when the expression is assigned to an xarray object**. This allows to operate symbolically on very large arrays and only compute the result for the indices of interest.

We provide utilities to **vectorize any scalar function** (taking multiple scalar arguments) into a function that will perform on ``xexpression`` s, applying the lazy broadcasting rules which we just described. These functions are called *xfunction* s. They are ``xtensor``'s counterpart to numpy's universal functions.

In ``xtensor``, all arithmetic operations (``+``, ``-``, ``*``, ``/``) and all special functions are *xfunction* s.

Iterating over `xexpression`s and Broadcasting Iterators
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

All ``xexpression`` s offer two sets of functions to retrieve iterator pairs (and their ``const`` counterparts).

 - ``begin()`` and ``end()`` provide instances of ``xiterator`` s which can be used to iterate over all the elements of the expression. The order in which elements are listed is ``row-major`` in that the index of last dimension is incremented first.
 - ``xbegin(shape)`` and ``xend(shape)`` are similar but take a *broadcasting shape* as an argument. Elements are iterated upon in a row-major way, but certain dimensions are repeated to match the provided shape as per the rules described above. For an expression ``e``, ``e.xbegin(e.shape())`` and ``e.begin()`` are equivalent.

.. toctree::
   installation

.. toctree::

   api_reference

.. _NumPy: http://www.numpy.org
.. _Buffer Protocol: https://docs.python.org/3/c-api/buffer.html
.. _libdynd: http://libdynd.org
