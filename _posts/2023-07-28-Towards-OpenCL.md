---
layout: post
title: "Moving towards OpenCL"
subtitle: "Week 7 and 8"
date: 2023-07-28
background: '/img/posts/GSoC.png'
tags: gsoc
---

## Background

So far, all my work on GPUs has been using CUDA. But CUDA is proprietary to NVIDIA and only works on NVIDIA GPUs. So, I've been working on moving the code to OpenCL, which is an open standard for parallel programming on heterogeneous systems.

## OpenCL

[OpenCL](https://www.khronos.org/opencl/)(Open Computing Language) is an open standard for cross-platform, parallel programming of diverse accelerators(CPUs, GPUs, FPGAs, etc) found in supercomputers, cloud servers, personal computers, mobile devices and embedded platforms.
Note the 2 key points - 
- `open standard` : this means that the specification and documentation of the technology are publicly available and can be accessed by anyone. 
- `cross-platform` : this means that it can run on multiple operating systems and hardware architectures without requiring major modifications to the code.

This makes OpenCL a very attractive option for developers who want to write code that can run on a wide range of devices. From Gnuastro's perspective, this means that we can write code that can run on multiple GPU manufactureres, as well as CPUs and other accelerators. Our GPU kernels will be portable to any system, regardless of its configuration!


Next point to consider is OpenCL is a `standard`. It is different from CUDA in this regard. CUDA is a framework, whereas OpenCL is a standard. What does this mean?

- The OpenCL standard refers to the specification and guidelines set forth by the Khronos Group which is responsible for developing and maintaining the standard. The OpenCL standard defines the API, data types, functions, and programming model that developers must follow when writing code for OpenCL. It is a formal document that ensures uniformity and compatibility across different OpenCL implementations.

OpenCL is not an open-source library! It basically defines how the library should behave(big simplification!). 

So what can we do with the standard alone? Not much! We need an `implementation of the standard`.

This also reminds me of the question I once had - What do you need to create a new programming language? 
My first guess was a compiler! My thought process was if a program(compiler in this case) can understand my High level language and convert it to corresponding machine code, then I can write programs in that high level language for any task!
So all I'd need is a compiler for that language.
Its close, but not totally accurate. 

You dont actually need a compiler for a new programming language. You ONLY need a `specification` for it. The specification will define the syntax and semantics(rules) of the language.
You only need a compiler when you want to run programs using your language!(what good is a language if you cant run programs using it? haha)


Similaraly OpenCL defines a set of rules which specify how it will behave. But to use OpenCL we need an implementation of this standard.
- OpenCL implementations are software packages developed by hardware manufactureres that provide the necessary drivers and runtime libraries for running OpenCL applications on their specific hardware. Each hardware vendor is responsible for creating their own OpenCL implementation that conforms to the OpenCL standard. This means that each implementation may have its own unique features and quirks, but they all adhere to the same standard.

There are many different implementations available for it! (find the full list [here](https://www.khronos.org/conformance/adopters/conformant-products/opencl) or [here](https://www.iwocl.org/resources/opencl-implementations/)).

Basically each of the hardware manfacturers provide an implementation of the OpenCL standard for their hardware. This implementation is usually provided as a framework. Depending on what hardware you have on your system, you can choose the corresponding framework to use.

## How does OpenCL work?

Here's waht a typical OpenCL system looks like :

![opencl-sytem]({{site.baseurl}}/img/posts/opencl/opencl-system.png)


OpenCL programs consist of two parts: host code and device code. The host code is written in C or C++ and runs on the host, while the device code is written in OpenCL C and runs on the device. The host code is responsible for setting up the OpenCL environment, creating the context, compiling the device code, and executing the kernels on the device.

The device code is compiled at runtime by the host code. This means that the host code must be compiled first, and then the device code can be compiled. The host code is compiled using a standard C/C++ compiler, while the device code is compiled using the OpenCL compiler. The OpenCL compiler is provided by the OpenCL implementation and is responsible for compiling the device code into binary code that can be executed on the device.


How does the OpenCL library interact with the hardware? Its made possible through OpenCL-ICD.

OpenCL ICD stands for OpenCL Installable Client Driver. It is a component of the OpenCL

It enables multiple manufacturers OpenCL drivers to coexist on a single system. Instead of having a single monolithic OpenCL driver, an ICD allows different manufactureres (e.g., NVIDIA, AMD, Intel) to provide their own separate OpenCL implementation as dynamically loadable libraries. This means that developers can select the appropriate OpenCL driver at runtime without needing to modify their applications.

The ICD mechanism is crucial for achieving portability and flexibility in developing applications using computational power of various devices from different manufacturers.


## OpenCL Programming Model

The Programming Model of OpenCL is very similar to CUDA which I covered in my previous post. However CUDA has a lot of abstraction since it has its own runtime library which communicates with the driver.
In OpenCL there's direct communication with the drivers and the host code is responsible for setting up the environment so its a bit more lower level than CUDA.

Some of the key terms in OpenCL are :

- Work Item: Basic unit of work on a compute device
- Kernel: The code that runs on a work item (Basically a C function)
- Program: Collection of kernels and other functions
- Context: The environment where work-items execute (Devices, their memories and command queues)
- Command Queue: Queue used by the host to submit work (kernels, memory copies) to the device.


I'll cover the programming aspect of OpenCL in more detail in my next post.
