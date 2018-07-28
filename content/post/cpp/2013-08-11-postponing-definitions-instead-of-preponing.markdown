+++
date = "2016-05-14T16:35:19+08:00"
draft = false
title = "Postponing definitions instead of preponing"
+++

Prepone definitions? Wrong!
------------------------------
Before C99, you must prepone all the variables' definitions before any
statements. But after C99 was released, you can put a variable's definition
in any place as long as before being used. I think this new feature is
introduced by learning from C++.

This makes programer write codes more convenient. But I used to insist the
old style. I even teach my students that they should use the old style when
I was a TA of the course *Programming with C*.

However, now I realize that I was wrong. **We should postpone variables'**
**definitions as long as possible**. It's not for coding style, but for the
effeciency of the program.

Consider this program.

    #include <cstdio>
    #include <iostream>
    
    void
    save_pwd(const std::string& pwd)
    {
        std::string salted_pwd("abc");
    
        if (pwd.size() < 6) {
    	    fprintf(stderr, "Password too short.");
    		return;
        }
        salted_pwd += pwd;
		...
    }
    
    int
    main(int argc, char **argv)
    {
        save_pwd("ddd");
        
        return 0;
    }
        
When running this program, it will print an error and exit the function.
The variable ``salted_pwd`` is not used. So, defining ``salted_pwd`` above
the ``if`` statement will waste the memory, the time to allocate memory and
the time to free memory. For the view of C++, you will pay for the cost of
construction and destruction of the object ``salted_pwd``.

Define and then assign? Wrong!
-------------------------------
When I first learned programming in C, my teacher says that

    int a;
	a = 1;

is the same as

    int a = 1;

Now I realize that they are not the same and the latter is more effecient.

For the former, ``int a;`` will allocate memory for the variable ``a`` and
then fill zero bytes in the memory. Then ``a = 1;`` will write value ``1``
in the memory.
For the latter, ``int a = 1;`` will directly allocate memory for the variable
``a`` and then write value ``1`` in the memory.
In other words, you will pay more cost when using the former style.

It seems that it makes little difference with the built-in type. When the
type of the variable ``a`` is user-defined type, great difference appears.

For example,

    std::string str;
	...
	std::string = "abc";

This will first call the default constructor of ``std::string`` and then call
the copy assignment operator. So the time and memory used in the default
constructor is wasted.

If we define an object in this way

    std::string str("abc");

or

    std::string str = "abc";

, it will only call the copy constructor. This is more effecient.

So now I prefer this way

    for (int i = 0; i < n; i++) {
	    ...
	}

instead of

    int i;

    for (i = 0; i < n; i++) {
	    ...
	}

Now there is a special case that is worth thinking.

    Person p;
	for (int i = 0; i < n; i++) {
	    p = persons[i];
		...
	}

and

    for (int i = 0; i < n; i++) {
	    Person p = persons[i];
		...
	}

, which is better?

Let's analysis the performance of them.
* The former need 1 construction, n assignments and 1 destruction. But it makes
the object ``p`` in a larger scope, which increase the comprehensibility and
maintainability of the program.
* The latter need n constructions and n destructions.

Which is more effecient? I think it all depends. If assignment is less expensive
than the total of construction and destruction, we should use the former style.
On the contray, we should use the latter style.

The author of **Effective C++** prefers the latter, but I prefer the former,
since I am sensitive with performance and I think memory allocation and free
will be more expensive than assignment.


