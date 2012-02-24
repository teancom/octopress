---
layout: post
title: "gem and npm are broken."
date: 2012-02-20 20:16
comments: false
published: false
categories: 
---

As the guy in charge of packaging up "everything" at my day job and some of my 
side-work, in both Centos and Debian environments, I have Opinions&trade; as 
to the right way to do things. But really, that opinion can be boiled down to 
this: package up *all* the things. It's that simple. The way to have 
reproducible environments, from development through continuous integration, QA, 
UAT, up to prod, is to have your application and all of it's dependencies in packages. 
Even if you don't have anything except dev and prod, it's still the best way. And 
by best, I mean simplest, most reliable, easiest to audit, and (maybe most 
importantly) the least work for the developer.

To start with, let's define our terms. By packages, I mean system-level packages
that integrate with the system-level packaging manager. I.e., debs on debian-derived
distributions, rpms on Redhat-derived distros, etc. These package formats do all 
the usual things that you'd want: determine dependencies, allow a given package to 
"provide" something other than just files (allowing for virtual dependencies), have 
built-in support for cryptographic signing and verification of those signatures, 
have the ability to run scripts before and after installation and removal, and much 
more. 

Language-level package management systems, like gem and npm
