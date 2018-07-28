+++
date = "2016-05-14T16:35:19+08:00"
draft = false
title = "Ruby-getting started"
+++

Running ruby
===============

Interactive
------------
In the past, I always tested some ruby statements with ``irb``. Now I can
test a small ruby program in the following way.
    
    [monkey@itlodge octopress]$ ruby
    str = "abc"
    puts str
	Ctrl+d
    abc

After hit ``ruby``, it allows me to type as many code as possible.Finally,
press ``Ctrl+d`` will end the input and it will evaluate the code.

Programs
----------
When writing a script, we always use ``#!`` and specific which language used
to run the code.This is the Unix [Shebang](http://en.wikipedia.org/wiki/Shebang_%28Unix%29) notation.For example:

    #!/usr/bin/ruby -w
	puts "abc"

From the man page, I know that the ``-w`` option means *Enables verbose mode
without printing version message at the beginning*.

But, many guys like this style:

    #!/usr/bin/env ruby
	puts "abc"

From the man page, I know that the ``env`` is a program that *run a program in
a modified environment*.So it can search the ``$PATH`` and find the ``ruby``
program to run.

Ruby documentation
=====================
The site [ruby-doc.org](http://www.ruby-doc.org) contains a complete set of the
RDoc documentation for Ruby.

The ``ri`` tool is very useful for looking up documentation.If you want to find
the documentation for a class, just type ``ri ClassName``. For example:

    [monkey@itlodge ~]$ ri Vector
    = Vector < Object
    
    ------------------------------------------------------------------------------
    = Includes:
    Enumerable (from ruby core)
    
    (from ruby core)
    ------------------------------------------------------------------------------
    The Vector class represents a mathematical vector, which is useful in its own
    right, and also constitutes a row or column of a Matrix.
    
    == Method Catalogue
    
    To create a Vector:
    *   Vector.[](*array)                   
    *   Vector.elements(array, copy = true) 
    
    To access elements:
    *   [](i)                               
    
    To enumerate the elements:
    *  #each2(v)                            
    *  #collect2(v)                         
    :

If you want to find the documentation for a method, just type
``ri method's name``.For example:

    [monkey@itlodge ~]$ ri sleep
    = .sleep
    
    (from ruby core)
    === Implementation from Kernel
    ------------------------------------------------------------------------------
      sleep([duration])    -> fixnum
       
    
    ------------------------------------------------------------------------------
    
    Suspends the current thread for duration seconds (which may be
    any number, including a Float with fractional seconds). Returns the actual
    number of seconds slept (rounded), which may be less than that asked for if
    another thread calls Thread#run. Called without an argument, sleep() will
    sleep forever.
    
      Time.new    #=> 2008-03-08 19:56:19 +0900
      sleep 1.2   #=> 1
      Time.new    #=> 2008-03-08 19:56:20 +0900
      sleep 1.9   #=> 2
      Time.new    #=> 2008-03-08 19:56:22 +0900
    
    
    :

A method's name may be the same in different classes or modules.In this case,
``ri`` will list all of them.If you type ``ri ClassName.method's name``, it
will show only the documentation of the corresponding class's.

More contributions, more reputations.If a class or module hasn't yet documented
in RDoc, send an email to ``suggestions@ruby-doc.org`` to ask them to add.
