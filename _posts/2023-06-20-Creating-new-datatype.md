---
layout: post
title: "Creating a new Data Structure for pyGnuastro"
subtitle: "Week 3 and 4"
date: 2023-06-14
background: '/img/posts/GSoC.png'
tags: gsoc
---

## Background

[GnuAstro](https://www.gnu.org/savannah-checkouts/gnu/gnuastro/gnuastro.html) is a powerful and comprehensive library designed to handle various data formats(FITS/TIFF/TXT and more) and perform a wide range of operations, all while maintaining consistency across its entire codebase. 


This is done by representing all the data (acquired via input or created internally), regardless of its type, in a single data structure which encompasses the core data as well as metadata. This greatly assists in mainting uniformity.
Internally all the data is represented in the form of a C struct : `gal_data_t`
The following image describes how it keeps the core data as well as metadata : 
![Code-Block1](({{site.baseurl}}/img/posts/creating-data-structure/gal_data_t.png))

Explaining each attribute of this structure will require a seperate post of itself :). Instead I'll focus on the main topic here : Since Im creating a python package for Gnuastro, and the `gal_data_t` is at the heart of this library, How do I represent this complex type in Python?!


Normally we use Classes to define new and complex data types in Python, but hey.. I'm wrapping a C library in Python using the Python-C API. This means I write my wrappers in C!



So the question comes down to how do I create a new type in Python using C language?


## Creating New Data Types in Python Without Classes and Objects

Before I continue, I've to appreciate [Numpy](https://numpy.org/) for the incredible peice of software it is, the more I understand it, the more it amazes me.


C is not an Object Oriented Programming Language, but Python is.

In case you didn't know the most common implementation of Python (the one you most probably have) is written in C! It's called CPython.

This raises an obvious question, how does Python implement its whole OOP paradigm in C?

This question also answers our question of how to represent `gal_data_t` in Python, because essentially they're looking for the same thing.


`PyObject` is the answer! To the Python interpreter(written in C) all the data types(built in as well as user defined) are of this type!

and what is this `PyObject`? Its a simple struct in C.