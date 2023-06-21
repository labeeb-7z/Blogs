---
layout: post
title: "GSoC - its finally here"
subtitle: "Intro to my GSoC project"
date: 2023-06-05
background: '/img/posts/GSoC.png'
tags: gsoc
---

## What is Open-Source and Gsoc?
Open source software is software with source code that anyone can inspect, modify, and enhance. There are many institutions and individuals who write open software, mainly for research or free deployment purposes. Mostly these softwares, have only a few maintainers, and multiple people, writing and debugging the code, helps a lot. This is where Google Summer of Code `GSOC` comes into the picture. It is a global, online program focused on bringing new contributors into open source software development. Many organisations float projects for the developers to take over the summer and Google mediates in the process, while also paying the contributors for their work over the summer.


## What is my project about?

It has 2 main components : 
- Create a Python Library for Gnuastro
    - Design an error handling mechanism for Gnuastro
    - Design corresponding data structures of Gnuastro in Python
    - Write wrapper functions to be used in python
- Add CUDA support in Gnuastro
    - Integrate CUDA with Gnuastro's build system
    - Write GPU kernels for compute heavy and parallelizable operations.

## What have I completed till now?

- On the Python Library Part : 
    - Gnuastro now has an error handling mechanism!
    - Added error handling in Python package for the 2 existing modules.
    - Defined error types for each corresponding error type in C library.
    - Implemented Python wrappers for 2 of the C library modules
- On the CUDA support part : 
    - Gnuastro can now build with cuda! this means it already supports GPU computations.
    - Added docs for installing, configuring, and testing CUDA
    - Added test CUDA kernels and demo programs to test them.
    - Implementing CUDA kernel for Convolution operation.

## 
