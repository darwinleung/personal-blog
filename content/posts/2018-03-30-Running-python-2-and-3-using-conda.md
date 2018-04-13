---
title: "Running python 2 and 3 using conda"
date: 2018-03-30T12:37:00-04:00
draft: false
tags: [ "python" , "mongodb", "conda" ]
categories: [ "programming" , "data science"]
---

I am taking the [M101P](https://university.mongodb.com/courses/M101P/about) course to brush up my Mongo skill, the course is kind of dated, it uses Python 2. However, I use Python 3 on my machine by default. I need a quick and easy way to run python 2 scripts without changing my default settings/path. I found conda offer a great out-of-box solution by using [environment](https://conda.io/docs/user-guide/concepts.html).

> A conda environment is a directory that contains a specific collection of conda packages that you have installed. For example, you may have one environment with NumPy 1.7 and its dependencies, and another environment with NumPy 1.6 for legacy testing. If you change one environment, your other environments are not affected. You can easily activate or deactivate environments, which is how you switch between them. You can also share your environment with someone by giving them a copy of your `environment.yaml` file.



It is perfect for my use case, I can simply create an env for python 2.7 and only use it within the course.

##  **Steps**

1. To create a new env for python 2.7 ([ref](https://conda.io/docs/user-guide/tasks/manage-python.html)):  
   `conda create -n py27 python=2.7 anaconda`  
   I got an error saying I already have an env created with the same prefix (py27), nice! it was created by default when I installed Python using Anaconda.
2. To get a list of env ([ref](https://conda.io/docs/user-guide/tasks/manage-environments.html#viewing-a-list-of-your-environments)), the one you are currently using is highlighted with an * :   
   `conda info --envs`
3. Next, to activate the py27 env:   
   `activate py27`
4. Notice the (py27) appear in the beginning of the command prompt, meaning I am currently in the environment, to verify python version:  
    `python -v`
5. To deactivate an environment:   
   `deactivate`
   If we list the env again, we are back to root.















