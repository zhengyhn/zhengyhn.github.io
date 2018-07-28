+++
date = "2016-05-14T16:35:19+08:00"
draft = false
title = "Const anyway!"
+++

Const pointer
--------------

``const`` is versatile.I am always confused by the following syntax.

    const char *str = "Hello";    // const data, non-const pointer
	char * const str = "Hello";    // const pointer, non-const data

Now there is a tip to remember it.

If ``const`` appears to the right of the asterisk, the pointer is
constant.If ``const`` appears to the left of the asterisk, the data
is constant.

So the following two statements are the same.

    const char *str = "Hello";
	char const *str = "Hello";

In STL, ``iterator`` is just like a ``T *`` pointer.
* ``const std::vector<int>::iterator iter`` is just like ``T * const iter``.
* ``std::vector<int>::const_iterator cIter`` is just like ``const T *iter``.

So, in a loop, if we don't want to modify the data, use ``const_iterator``.


Const member function
----------------------

Sometimes, we have two version member functions.One is const and the other is
not.

    #include <string>
    #include <iostream>
    
    class Str {
    public:
        Str(std::string str)
    	:data(str)
        { }
        
        const char&
        operator[](std::size_t pos) const    // Const objects use this
        {
    	    return data[pos];
        }
    
        char&
        operator[](std::size_t pos)    // Non-const objects use this
        {
    	    return data[pos];
        }
    
    private:
        std::string data;
    };
    
    void
    print(const Str& const_str)
    {
        std::cout << const_str[1] << std::endl;
    }
    
    int
    main(int argc, char **argv)
    {
        Str nonconst_str("abc");
        std::cout << nonconst_str[1] << std::endl;
        nonconst_str[1] = 'a';
        
        print(nonconst_str);
    
        return 0;
    }

In ``main``, ``nonconst_str`` is a non-const object and it can be modified
by ``[]``.In ``print``, ``const Str& const_str`` means
**pass parameters by reference-to-const**, so ``const_str`` is a const object.

We observe that the ``const`` keyword is after the closing parenthesis of the
argument list.This means the function is a const member function.

Reference to [MSDN](http://msdn.microsoft.com/en-us/library/6ke686zh.aspx),
**A constant member function cannot modify any non-static members or call any**
**member functions that aren't constant.**That is, the cons member function
can't modify the object that it is called.

The above philosophy is called **bitwise constness** or **physical constness**.
There are another philosophy called **logical constness**.Adherents to this
philosophy argue that **a const member function might modify some of the bits**
**in the object on which it's invoked, but only in ways that clients cannot**
**detect**.

For example, if we add a new member function ``length()`` to the above class.

        std::size_t
        length() const
        {
    	    len = data.length();
    
            return len;
        }
        
    private:
        std::string data;
        std::size_t len;

This will generate the following compiled error.

    test.cpp:25:6: error: assignment of member ‘Str::len’ in read-only object
      len = data.length();
          ^

Since member variable ``len`` has been modified and the compiler use the
**bitwise const**, error produces.However, return and object's length seems
not modify the object it is called.

To solved this problem, use the **mutable** keyword.It can free non-static
data members from bitwise constness constraints.

    mutable std::size_t len;

I beleive in code reusing forever!In the above example, the duplicate code
in the two ``operator[]`` functions can be merged into one.

Only then non-const function should be modified.

    char&
    operator[](std::size_t pos)    // Non-const objects use this
    {
	    return const_cast<char&>(static_cast<const Str&>(*this)[pos]);
    }

``static_cast<const Str&>``will make ``*this``const and then it can call
``operator[]``.Finally, ``const_cast`` is used the free the const constraint.

This is all about ``const``, it's an amazing keyword.For readable and efficient
code, use it anyway.
