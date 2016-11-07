gpyfft
======

A Python wrapper for the OpenCL FFT library clFFT.

## Introduction

### clFFT


The open source library [clFFT] (released under Apache 2.0 license) implements FFT for running on a GPU via OpenCL. Currently supported features are:
* batched 1D, 2D, and 3D transforms
* supports many transform sizes (any combinatation of powers of 2,3,5,7,11, and 13)
* single and double precisions
* complex and real-to-complex transforms
* supports injecting custom code for data pre- and post-processing

### gpyfft

This python wrapper is designed to tightly integrate with [PyOpenCL]. It
consists of a low-level Cython based wrapper with an interface similar to the
underlying C library. On top of that it offers a easier-to-use high-level
interface designed to work on data contained in instances of
`pyopencl.array.Array`, a numpy work-alike array class for
GPU computations. The high-level interface takes some inspiration from [pyFFTW]. For details of the high-level interface see [fft.py].

## Status

This wrapper is functional, the high-level interface is not completely settled.

## Basic usage

Here we describe a simple example of performing a batch of 2D complex-to-complex FFT transforms on the GPU, using the high-level interface of gpyfft. The full source code of this example ist contained in [simple\_example.py], which is the essence of [benchmark.py]

imports:

``` python
import numpy as np
import pyopencl as cl
import pyopencl.array as cla
from gpyfft.fft import FFT```

initialize GPU:

``` python
context = cl.create_some_context()
queue = cl.CommandQueue(context)
```

initialize memory (on host and GPU). In this example we want to perform in parallel four 2D FFTs for 1024x1024 single precision data.

``` python
data_host = np.zeros((4, 1024, 1024), dtype = np.complex64)
#data_host[:] = some_useful_data
data_gpu = cla.to_device(queue, data_host)```

create FFT transform plan for batched inline 2D transform along second two axes.

``` python
transform = FFT(context, queue, data_gpu, axes = (2, 1))
```

If you want an out-of-place transform, provide the output array as additional argument after the input data.

Start the work and wait until it is finished (Note that enqueu() returns a tuple of events)

``` python
event, = transform.enqueue()
event.wait()
```

Read back the data from the GPU to the host

``` python
result_host = data_gpu.get()
```

### work done

-   low level wrapper (mostly) completed
-   high level wrapper
   * complex and real<->complex transforms
   * single precision
   * interleaved data
   * in and out-of-place transforms


  [clFFT]: https://github.com/clMathLibraries/clFFT
  [pyFFTW]: https://github.com/hgomersall/pyFFTW
  [PyOpenCL]: https://mathema.tician.de/software/pyopencl
  [fft.py]: gpyfft/fft.py
  [pyfft]: http://github.com/Manticore/pyfft
  [simple\_example.py]: examples/simple_example.py
  [benchmark.py]: gpyfft/benchmark.py
