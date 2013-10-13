---
layout: post
title: "A Brief Introduction to CUDA"
date: 2013-10-13 06:14
comments: true
categories: 
---

{% img center http://moss.csc.ncsu.edu/~mueller/cluster/nvidia/nvidia-cuda.jpg 333 202 %}

**CUDA**, or Compute Unified Device Architecture, is a parallel programming API for [Nvidia](http://www.nvidia.com/object/cuda_home_new.html) GPUs 
(also known as GPGPUs for general purpose computing on GPUs). What this means is that with CUDA, one can easily run general purpose CPU code on 
the GPU, without having to learn a whole new GPU programming language like OpenGL or DirectX, albeit with some framework code. 
The idea of GPGPU is not new; people have been using GPUs for compute tasks since the 1990's, but it's not until the recent five years that 
this type of compute has become mainstream, easily to learn, and affordable. CUDA is but NVIDIA's specific implementation of general purpose
GPU programming that runs only on recent Nvidia graphics cards, but there are other solutions such as [OpenCL](http://www.khronos.org/opencl/)
and [C++ AMP](http://msdn.microsoft.com/en-us/library/vstudio/hh265137.aspx) that target other hardware solutions and even CPUs. In this post
I'll be mainly talking about CUDA, but you will see that the core design patterns are pretty much the same across all of these implementations.

### The Hardware
GPUs are similar to CPUs in that they have many compute cores, each with their own cache, plus global memory. However the difference is that 
GPUs have hundreds, if not thousands, of these said cores. So we can see right away that GPUs probably won't be good at running serial code
very fast, but rather great at running thousands of threads in parallel. Indeed this is what GPUs are engineered to do: graphics processing
which involves computing on millions of triangles and pixels on the screen in parallel for displaying images, videos or playing games. GPUs
are really good at launching lots of threads, thousands of them, without much overhead. They are also good at running all those threads in 
parallel efficiently, applying `map()` operations from each of these threads. With hardware optimized for this kind of computing, we can't
think serial code; we need to organize our program to run massively parallel.

{% img center http://images.anandtech.com/doci/6973/GTX780BlockDiagram.png 826 452 GTX 780 block diagram %}

### GPU Parallelism
The design pattern for programming GPUs is quite simple: start on the CPU, launch some code on the GPU, and copy back the results to the CPU.
The CPU is often referred to as the "host", while the GPU is referred to as the "device". Of course the GPU is but one possible device; a compute
node such as Nvidia's [Tesla](http://www.nvidia.com/object/tesla-servers.html) or an FPGA can also be considered a "device" with respect to the "host".
The program runs on the GPU is called a "kernel", so when someone refers to "writing a kernel", what they mean is that they are writing the part
of the code that actually runs on the GPU. The kernel is a small block of code that runs on each thread of the GPU, often performing a `map()`
operation for each data element in the input stream. A stream is simply input data with similar computation to be performed.

Basic GPU programming steps:
	1. Allocate memory on the GPU. Zero-out memory if needed.
	2. Copy some data from CPU memory to GPU memory.
	3. Launch some kernels on the GPU to perform a compute task.
	4. Copy some data back from the GPU memory to the CPU memory.
	5. Free the allocated memory on the GPU.
	
{% img center http://upload.wikimedia.org/wikipedia/commons/5/59/CUDA_processing_flow_%28En%29.PNG 386 400 CUDA processing flow %}

Of course after step 4, you probably will do something with all that computed data coming back from the GPU, perhaps printing them out or writing
to a file. Let's see how CUDA implements these basic functions.

### The CUDA API
CUDA is built on top of C, and it's syntax is very similar. As mentioned above, the program initially starts on the CPU/host (you can think of it as
the "master thread"), kernels are launched on the GPU/device, and ends back on the host. Here are the CUDA-specific function calls for the above
5 steps:

1. **Allocating memory:** `cudaMalloc(pointerToMem, numBytes)`. 
	**Zeroing memory:** `cudaMemset(pointerToMem)`.
2. **Copying data:** `cudaMemcpy(dest, source, numBytes, directionType)`. 
	**Copying data to GPU:** `cudaMemcpyHostToDevice`.
3. **Launching kernels:** `kernelName<<<gridSize, threadSize>>>(outputData, inputData)`
4. **Copying data to CPU:** cudaMemcpy with `cudaMemcpyDeviceToHost` direction.
5. **Freeing memory:** `cudaFree()`
	
The `cudaMemcpy()` function takes different arguments depending on which direction we are copying the data. The 4th argument to the function
determines which direction the data is copied. For example, when copying data from the CPU to the GPU, we would use 
`cudaMemcpy(d_in, h_in, ARRAY_BYTES, cudaMemcpyHostToDevice)`, where the device is the destination and the host is the source.

A common error that beginner CUDA programmers face is trying to access invalid memory. For example, accessing host memory from the device or vice
versa. We generally use `h_` when naming variables to refer to any data or memory in the host and `d_` to refer to elements in the device. 
This is purely conventional, but helps differentiate where the data we are referring to resides, and makes the code cleaner. 

### The Kernel
The kernel is arguably the most important and most interesting part about GPU programming. It's the core piece that actually does all of the compute
work on the GPU. To understand the kernel, we have to first understand Nvidia's GPU hardware a little more.

We can imagine the entire GPU to be a grid. That is, all of the cores on the GPU sits on a massive 2D grid. Now, this grid is split into blocks,
which we can imagine when combined into NxN formation creates the entire grid. Each block is further subdivided into threads. We can again
imagine these an MxM array of threads which would when laid out would build a block. When launching a kernel on the GPU, the programmer must
explicitly specify the size of the grid and the block. In CUDA, the format for calling a kernel is: `kernel<<<gridOfBlocks, blockOfThreads>>>`.
In other words, one must specify `kernel<<<numBlocks, threadsPerBlock>>>`. When determining the number of blocks and the threads per block, there
are a couple things to consider:

* How big is the problem?
* How is the problem split up? (data parallelism)
* How many cores do we have?
* How many blocks per core can we support?
* What are the limits for the number of blocks and threads for the particular GPU we're working with?

The last point is simple to figure out. GPUs supporting CUDA version 2.0 (GTX 4XX and above) can support up to 1024 threads/block, 
while older GPUs supports 512 threads/block. However, the GPU will always launch a minimum of 32 threads at once, so the number specified here
should always be a multiple of 32, preferably above 128 threads. There is one more caveat to threads and blocks, and that is that the grid is
actually 3D! So when we specify an argument like `kernel<<<1,64>>>` what we're actually calling is `kernel<<<dim3(1,1,1), dim3(64,1,1)>>>`.
The limits of the grid depends on your particular target GPU, but for example a CUDA 3.5 compatible GPU like my GTX 780 can have a grid 
dimension of 65536 x 65536 x 65536 and a block dimension of 1024 x 1024 x 64. Check out the features and specifications for the kernel
[here](http://en.wikipedia.org/wiki/CUDA) for your particular GPU.

The following function calls can be used to get the threadId and blockId from any thread within a kernel during execution.
```
threadIdx.x
threadIdx.y
threadIdx.z
blockDim

blockIdx.x
blockIdx.y
blockIdx.z
gridDim
```

### A Note About Cache Memory
Since we're discussing about the kernel, I also want to mention something quick about the memory hierarchy of Nvidia GPUs. Each thread has
what's called **"local memory"**. This memory is about 512KiB in size per thread, and is used whenever the thread creates local variables during
kernel execution. A group of threads in a block have what's called **"shared memory"**. This memory is about 48KiB in size per block, and any
thread that's part of the same block can access this memory. Finally we have **"global memory"**, which is a large pool of memory in the GiB sizes 
available to any thread and any block. My GTX 780, for example, has 3GiB of GDDR5 of such memory. I also want to mention that these memories are
hierarchical; local memory is faster than shared memory, which in turn is much faster than global memory. When we call `cudaMemcpy`, we are
essentially copying data from the host to the "global memory" of the device.

{% img center http://media.hpcwire.com/images/nvidia_arc_big.png 528 480 Memory hierarchy %}

### Sample Program
Here's a full CUDA sample program. This program creates a 1D array of input values on the host, copies them to the device, launches a kernel
to cube each value, copies the resulting array back to the host, and finally prints the value of the array. Try it out! The command to compile
and run is: 
```
$ nvcc -o cube cube.cu
$ ./cube
```

Nvcc is Nvidia's specialized C compiler for CUDA.

```
#include <stdio.h>
#include "cuda_runtime.h"
#include "device_launch_parameters.h"

// The kernel
__global__ void cube(float * d_out, float * d_in){
	int idx = threadIdx.x;
	float f = d_in[idx];
	d_out[idx] = f*f*f;
}

// Main function
int main(int argc, char ** argv) {
	const int ARRAY_SIZE = 96;
	const int ARRAY_BYTES = ARRAY_SIZE * sizeof(float);

	// generate the input array on the host
	float h_in[ARRAY_SIZE];
	for (int i = 0; i < ARRAY_SIZE; i++) {
		h_in[i] = float(i);
	}
	float h_out[ARRAY_SIZE];

	// declare GPU memory pointers
	float * d_in;
	float * d_out;

	// allocate GPU memory
	cudaMalloc((void**) &d_in, ARRAY_BYTES);
	cudaMalloc((void**) &d_out, ARRAY_BYTES);

	// transfer the array to the GPU
	cudaMemcpy(d_in, h_in, ARRAY_BYTES, cudaMemcpyHostToDevice);

	// launch the kernel
	cube<<<1, ARRAY_SIZE>>>(d_out, d_in);

	// copy back the result array to the CPU
	cudaMemcpy(h_out, d_out, ARRAY_BYTES, cudaMemcpyDeviceToHost);

	// print out the resulting array
	for (int i =0; i < ARRAY_SIZE; i++) {
		printf("%f", h_out[i]);
		printf(((i % 4) != 3) ? "\t" : "\n");
	}

	cudaFree(d_in);
	cudaFree(d_out);

	return 0;
}
```

In a future post, I'll give a brief introduction to GPU programming with OpenCL.