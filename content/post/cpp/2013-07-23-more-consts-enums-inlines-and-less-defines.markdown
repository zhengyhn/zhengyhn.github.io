+++
date = "2016-05-14T16:35:19+08:00"
draft = false
title = "More consts enums inlines and less #defines"
+++

This is an old topic.

For constants
================
Consider a macro below.

    #define PI 3.14

There are several drawbacks when using a macro.

* It's hard to debug.As we all know, the macros are resolved by the
preprocessor and the compiler know nothing about them.When you get an error
when compiling the program, the error message may refer to ``3.14`` but not
``PI`` because ``PI`` is not in the symbol table.

* It will result in more object codes.When the preprocessor replace ``PI``
with ``3.14``, there will be several copy of 3.14 in the object code.

If we use the **constant** instead of the macro,

    const double Pi = 3.14;

it has the following advantages, respectively.

* It's easy to debug, since the constant is in the symble table.

* It has only one copy in the object code.The other place it appears is just
its references.

But I think it has disadvantages too.

When replacing lots of **macros** with **constant**, there will be
lots of entries in the symbol table.And it will make the object files or
executable files larger.

When defining a constant string, we may want to do this.

    const char *Name = "Tom";

But this is not effecient.

If defined above, the value "Tom" cannot be changed, but what the pointer
``Name`` points to can be changed.So the compiler will allocate a piece of
memory for the pointer ``Name`` in the normal data segment.Moreover, when
the linker starts working, it need to perform some relocations for ``Name``.

So, it's better to be defined as follows.

    const char * const Name = "Tom";

or
    const char Name[] = "Tom";

Now the compiler will put ``Name`` in the read only data segment and the linker
needn't performing relocations.

Alternatively, we can use this.

    const std::string Name("Tom");

For class-specific constants
=============================
Sometimes we need a constant member in the class, for example.

    class Hand {
    private:
        static const int NumFingers = 5;
        int fingers[NumFingers];
    };

In order to ensure there is only one copy of the constant, it must be
``static``.

However, ``static const int NumFingers = 5;`` is just a **declaration** for
``NumFinger``.Why this can be compiled with no error message?Everything
should have its definition, does it?However, the class-specific constants
that are static and with integral type(``int``, ``char``, ``bool``,...) is
an exception.

You can even use the constant.

    #include <cstdio>
    
    class Hand {
     public:
        static const int NumFingers = 5;
        int fingers[NumFingers];
    };
    
    int main(int argc, char *argv[])
    {
        printf("%d\n", Hand::NumFingers);
        
        return 0;
    }

But when you want to use the address of ``NumFingers``, you must put the
definition of ``NumFingers`` in the implemetation file of the class.

    #include <cstdio>
    
    class Hand {
     public:
        static const int NumFingers = 5;
        int fingers[NumFingers];
    };
    
    const int Hand::NumFingers;
    
    int main(int argc, char *argv[])
    {
        printf("%p\n", &Hand::NumFingers);
        
        return 0;
    }

Remember, there is no need to put ``static`` at the begining of the definition.

We know that the size of the array ``finger`` is just the value of the static
constant ``NumFingers``.But there is another way to do that.

    class Hand {
     public:
        enum {NumFingers = 5};
        int fingers[NumFingers];
    };

That is fine, too.

For functions
==============
Macros like this

    #define MAX(a, b) ((a) > (b) ? (a) : (b))

can result in many painful problems.

* First, you have to remember parenthesizing all the arguments(We all know
why).

* There are problems when calling it like this ``MAX(++a, b)``(It's easy to
think about it).

If using ``inline`` and ``template``, we can solve all this problems.

    #include <cstdio>
    
    template<typename T>
    inline T
    max(const T& a, const T& b)
    {
        return a > b ? a : b;
    }
    
    int
	main(int argc, char *argv[])
    {
        char a = 'a', b = 'b';
        int c = 1, d = 2;
        
        printf("%c\n", max(a, b));
        printf("%d\n", max(c, d));
        
        return 0;
    }

However, I don't think macros are useless and should be replaced by const,
enum and inline.In a way, macro can decrease the time of compiling and
linking.And there are still lots of code that using macros.




