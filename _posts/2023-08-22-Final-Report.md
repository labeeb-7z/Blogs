---
layout: post
title: "Moving towards OpenCL"
subtitle: "Week 7 and 8"
date: 2023-07-28
background: '/img/posts/GSoC.png'
tags: gsoc
---


I will be discussing the goals of my GSoC project, how I spent my time and what I learned during this period. I will also be discussing the future of my project and what I plan to do next.

# Goals of my GSoC

The [original](https://openastronomy.org/gsoc/gsoc2023/#/projects?project=gnuastro_library_in_python) Google Summer of Code project this was year was to : 
1. Redisgn the `error handling` inside Gnuastro C library.
2. Adding wrappers for Gnuastro library functions in `pyGnuastro`.

Prior to GSoC, my experience mostly consisted of Deep Learning and Computer Vision. I had a good high-level understanding of how GPUs were leveraged for the compute intensive tasks in various libraries and frameworks in these domains. I had started exploring the lower-level abstractions over GPUs using the CUDA framework.

In the early weeks of February, I delivered a [presentation](https://docs.google.com/presentation/d/1texW2MQJqjdbtPCuLULqXf8-1GuIrub4bJh_b_EffS4/edit?usp=sharing) to the Gnuastro development team. The point of this presentation was a proposal outlining the integration of GPU support into Gnuastro — an idea borrowed from the Machine Learning world but with huge advancement potential in the feild of Astronomy. Both of these domains process huge amounts of data. Both of these domains are characterized by the processing of substantial volumes of data.

My mentor [Mohammad Akhlaghi](https://akhlaghi.org/) was very supportive of this idea and gave me the go ahead to start working on it.

And so, we had a 3rd goal for this GSoC project : 

3. Adding GPU support to Gnuastro.


## Work Done Throughout the GSoC

### Error Handling
**Need** : All of Gnuastro's library functions  performed error handling using `error(EXIT_FAILURE, ....)`; thus exiting the program whenever an error was encountered with a detailed error message. This wasn't a problem for the Gnuastro programs however for other callers like pyGnuastro, this is problematic as it exits from the entire Python environment.

The new error handling mechanism defines a module `error.h`  new data structure `gal_error_t`. The exact contents of this structure have gone through multiple iterations but the final one is : 

![gal_error_t]({{site.baseurl}}/img/posts/final/gal_error.png)

The user should define a `gal_error_t` before the function call and pass it as an argument to the function(every function in Gnuastro will have an extra argument now).

During the function execution, if any error occurs, it will populate the `gal_error_t` with the error message and the error code. The user can then check the error code and the error message to determine what went wrong.

![new_error_handling]({{site.baseurl}}/img/posts/final/error_handling.png)

Corresponding functions are added in `error.h` for writing and managing the structure. Some methods are also provided for Python interface. 

After the module was finished, Mohammad implemented the new error mechanism inside the `cosmology.c` module, and then I used it to update the corresponding cosmology module in pyGnuastro. This solved the main the problem of python environment exiting on any error, instead errors were being reported inside the python shell.

![python_error]({{site.baseurl}}/img/posts/final/py-error.png)


This completed setting-up the low level infrastructure for the new error handling mechanism. This can be now used by other modules of Gnuastro to update what happens when an error occurs. Implementing the high level error function calls, deciding the exact error type and defining what message should be shown, would be best done by the original authors of the modules.


The new error handling mechanism currently lives at the [Gnuastro repository](https://gitlab.com/makhlaghi/gnuastro-dev/-/tree/error). 


### pyGnuastro

Apart from implementing the new error handling mechanism in existing modules of pyGnuastro, I worked on 2 major things
 - **Implemented speclines module in pyGnuastro** : this is a simple module without any complex data structures. I tried this first when I was learning about the C-Python API. It gave me a good grasp of how and what's going on in the existing pyGnuastro implementation.
 - **GAL_DATA_T for Python** :  The core data structure of Gnuastro - gal_data_t is a C struct. Any external data is represented using this structure. It was crucuial to had a similar structure in Python. Previously Jash had worked on loading and saving fits file made use of the Numpy-C API to to convert the raw data inside the gal_data_t to a Numpy array. This was an extremely clever and efficient idea, however it skipped all the other details inside gal_data_t. We had to find a way to represent the entire gal_data_t in Python. The normal way to create a new data structure in Python would be to create a new class. However, the wrappers are written in C language and we don't get access to the Python interpreter. I took some more inspiration from Numpy on how they [created a new Python](https://numpy.org/doc/stable/reference/c-api/index.html) - their core data structure : `numpy.ndarray` - using the C-Python API. I then discovered the API allows us to [define custom objects](https://docs.python.org/3/extending/newtypes_tutorial.html) which may be used a data type for the Python interpreter. I learnt and used them to have a corresponding `pygnuastro.data` for pyGnuastro. It basically acted as a new data type in python similar to `numpy.ndarray`, had other details of gal_data_t.After this we had details of gal_data_t in python but we were missing on Jash's idea of utilizing Numpy in pyGnuastro. I spent some time to make sure we can still utilize numpy's speed inside pyGnuastro, The C-Python API is versatile and it allows having complex objects as sub-objects to other objects. Eventually we had the array(raw data) being represented as a `numpy.ndarray`! This meant we had both the speed of numpy and the details of gal_data_t in pyGnuastro's `pygnuastro.data`. This was a major milestone in pyGnuastro.

 ![pyGnuastro.data]({{site.baseurl}}/img/posts/final/python-type.jpg)

### GPUs in Gnuastro

Gnuastro is an astronomical data analysis and manipulation library. Astronomical data is usually very large in size, and thus computationally intensive. If the operations performed on this data are parallelizable, then GPUs can significantly speed up the processing.

I started my work on GPUs right after Mohammad approved my initial idea. Here's a summary/story of all the work done for GPU support : 

1. **Learning about build systems** : After GPU support idea was accepted, my mentor suggested we should first setup the build system so CUDA modules can be integrated smoothly in the future. Gnuastro uses [Autotools](https://en.wikipedia.org/wiki/GNU_Autotools) for its build system. I started by learning about [autoconf](https://www.gnu.org/software/autoconf/), [automake](https://www.gnu.org/software/automake/) and [libtool](https://www.gnu.org/software/libtool/). 

2. **Linking Gnuastro with CUDA runtime** : CUDA SDK provides a [runtime library - `cudart`](https://nvidia.github.io/cuda-python/module/cudart.html) which the necessay component to initiate communication with the GPU drivers. The runtime library is distributed as both a static and shared object file. This made things easier as we could link the runtime library statically with the Gnuastro library, making `cudart` part of Gnuastro. I modified the configure script to link the runtime library statically with Gnuastro. This was also the time I learnt extensively about how low level system libraries are built, linked and distributed.

3. **Struggling with Libtool** : I then tried to implement some simple matrix functions in CUDA and integrate them with Gnuastro.  CUDA source code is compiled by `nvcc` compiler. However during linking, libtool assumes that all source files are compiled by `gcc`. It ignored all the CUDA source files. After writing dedicated rules for CUDA source compilation in the Makefile, the CUDA source was getting compiled, but not being linked to the Gnuastro. Libtool only links files having a corresponding libtool object(.lo files) and they're created by libtool for each source file handled by it(which in our case were gcc compiled files).

4. **AutoMake developers rescuing us** : After trying and struggling with libtool for a few days, my mentor suggested that I contact the AutoMake developers to seek some help. I [mailed](https://lists.gnu.org/archive/html/automake/2023-03/msg00036.html) them a [small demonstration](https://github.com/labeeb-7z/cuda-gnu/tree/main/shared-library) of what I was trying to do and waited for there response. After a few days, I received a reply from them. The fix was actually simple, automake had special variables(`LD_ADD`) which directly communicates with the GNU linker (ld) and I just had to add CUDA object files to this variable. It worked and we finally had a working CUDA module in Gnuastro which used GPU for execution!


It was around 1st week of April now, I made my final proposal submission and had fingers crossed for getting selected in GSoC.


As mentioned in the GSoC proposal, we had to first focus on the Error handling and Python wrappers, so I started working on these two goals (I was also indeed selected for GSoC in the meantime!).


5. **Convolutions on GPU** : After getting back to working with GPUs in around June-July, I started with implementing the [convolution function in CUDA]. Convolution is a direct operation as well as a subroutine to other operations in Gnuastro.

The results of CUDA convolution were remarkable. We got upto 400x speed up on convolution operation! My mentor then suggested me since the speedup is very significant, I should prioritise getting more of GPU work done.
Read more about Convolution on GPU in my blog [here](https://labeeb-7z.github.io/Blogs/2023/07/03/GPUs-and-Convolution.html).


6. **Adapting OpenCL** : CUDA is a proprietary framework by Nvidia. It only works on Nvidia GPUs. We wanted to make Gnuastro GPU support available to all users, irrespective of the GPU they have. This is where OpenCL comes in. OpenCL is an open standard for parallel programming of heterogeneous systems. It is supported by all major GPU vendors. I started learning about OpenCL and how it works at a low level. I also started learning about the OpenCL C99 programming standard. Read more about starting with OpenCL in my blog [here](https://labeeb-7z.github.io/Blogs/2023/07/28/Towards-OpenCL.html).

7. **Integrating OpenCL** : OpenCL was initially hard to learn, but I managed to integrate that with Gnuastro right before my GSoC's official timeline was about to end! I have a pretty detailed blog on the the entire integration process [here](https://labeeb-7z.github.io/Blogs/2023/08/12/Integrating-OpenCL.html).

8. **Same code on CPU and GPU** : After we had success with OpenCL, my mentor recommended we should try executing the `exact` same code on CPU and GPU - to show the concept of executing same instructions both processors and seeing the speed-up on GPUs. This was never done in the field of Astronomy so it'd have been a great demonstration. This was quite challenging as GPUs are programmed with different frameworks and have some extra components in code for management. Usually in Machine Learning frameworks, the GPU and CPU modules are generally written seperately(Infact Tensorflow used to have different package altogether for GPU until 2.0) 
However the good part is, most of the GPU frameworks are derived from C/C++ language and have  . I spent my last week of GSoC trying to implement the core logic in a Macro which will be shared by both OpenCL kernels and C library and had success, this can be accessed here. 

**Future** : The future of this project is very bright. I have set up the bare-bone GPU integration already, I'll continue to add GPU modules building upon it.. We have a working OpenCL integration. We have a working CUDA integration. We have a working CPU-GPU code sharing. I mentioned certain challenges we are currently facing in my [opencl_integration](https://labeeb-7z.github.io/Blogs/2023/08/12/Integrating-OpenCL.html) blog. I'll continue to figure out a solution for them and adding support for further modules on GPU.

## Acknowledgements

GSoC has been a great learning experience for me. I'm extremely grateful to everyone who was part of this journey.

I would like to thank my mentor [Mohammad Akhlaghi](https://akhlaghi.org/) for his constant support and guidance throughout the project. He has been very patient right from the beginning, beleived in me when I did not have a clear idea on how I'd approach all the goals. He allowed me work on my pace, explore and learn things as needed and has always pulled me out of the rabbit hole whenever I got stuck. Everytime I join a meeting with him, I learn something new. I'm very grateful to him for giving me this opportunity to work on this project.

I am Graciously thankful to Jash Shah for introducing me to the Gnuastro development team and walking me through the existing work on error handling and pyGnuastro. It provided me a huge boost was extremely valuable. He's always been attentive to my small queries and has supported me through multiple challenges. In general, Im very grateful to have him as a mentor and freind.

I would also like to thank the Gnuastro development team for their support and feedback throughout the project. Its been such a wonderful time working with them. I have learnt a ton from attending Pedram's work on adding Sql to Gnuastro, Fathma's work on Tiff files and Curl library, Faezeh's work on implementing Convolutional Neural Networks in Gnuastro.
They've always been crucial in providing feedback and suggestions on my work. I'm very grateful to them for their support. 
I am genuinely grateful for the opportunity to collaborate with such a talented and committed group, and I look forward to work and grow with them in the future.

I would also like to thank the Google Summer of Code team for taking the wonderful initiative and giving me this opportunity to work on this project.
