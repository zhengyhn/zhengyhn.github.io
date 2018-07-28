+++
date = "2016-05-14T16:35:19+08:00"
draft = false
title = "Always pass parameters by reference-to-const"
+++

We all know that *pass by value* is not effecient.So we use *pass by pointer*
in C.However, in C++, we prefer **pass by referece-to-const**.

Here is an example.

    #include <iostream>
    #include <cstdio>
    
    class Dog {
    public:
        Dog()
        {
    	    printf("Calling Dog's constructor\n");
        }
        Dog(const Dog& d)
        {
    	    printf("Calling Dog's copy constructor\n");
    		this->name = d.name;
        }
        virtual ~Dog()
        {
    	    printf("Calling Dog's destructor\n");
        }
    private:
        std::string name;
    };
    
    void
    walk_the_dog(Dog d)
    {
    
    }
    
    int
    main(int argc, char **argv)
    {
        Dog dog;
        walk_the_dog(dog);
        
        return 0;
    }
        
The running result is as follows:

    Calling Dog's constructor
    Calling Dog's copy constructor
    Calling Dog's destructor
    Calling Dog's destructor

Now, let's analysis this *passing by value* process.
First, ``Dog dog;`` will call the constructor. Then, since the function
``walk_the_dog`` pass the parameter ``d`` by value, it will call the copy
constructor. Before exiting the function, it will call the destructor of ``Dog``
to destroy the parameter ``d``. In the end, before exiting the main function,
the destructor of ``Dog`` will be called again to destroy the ``dog`` object.

Now, it seems that *passing by value* will result in a call to copy constructor
and a call to destructor. But, in fact, it will result in two call to copy
constructor and two call to destructor. Note that there are a ``string`` object
as the member of the class ``Dog``, so there will be an extra call the copy
constructor and destructor of the ``string`` object.

Obviously, the cost of *passing by value* is very expensive.
How is *passing by reference-to-const*?

    void
    walk_the_dog(const Dog& d)
    {
    
    }
    
Just modify a bit and the result will be:

    Calling Dog's constructor
    Calling Dog's destructor

Note that the ``const`` is very important. If there is not a ``const``, the
compiler will put the object in the writable part of the memory so that it
cannot be shared by other functions. If declared as ``const``, the object
will be put into the readonly part of the memory and can be shared by lots
of functions so that the program need less memory.

The implementation of reference is using the pointer in C. Therefore, it may
be more effecient to pass the built-in type parameters by value than referece.
It's true. If the parameter is an object of type ``char``, which occupies
1 byte. But a pointer ocuppies 4 bytes in a 32-bit machine. So we should prefer
passing by value when the parameter is of built-in type? No, we should always
pass the parameters by reference-to-const, since **the compiler will always**
**put the pointer in the register!**

Finally, just remember, **always pass the parameters by reference-to-const**.

