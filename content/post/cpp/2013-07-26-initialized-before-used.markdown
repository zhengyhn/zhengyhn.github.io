+++
date = "2016-05-14T16:35:19+08:00"
draft = false
title = "Initialized before used"
+++

For constructor
----------------
In the constructor, the statements in the body are assignments, not
initializations.

    #include <iostream>
    
    class Girl {
    public:
        Girl(std::string name, int age)
        {
    	    name_ = name;    // These are assignments, not initilizations.
    		age_ = age;
        }
    private:
        std::string name_;
        int age_;
    };

If you do this, the program will be very slow.When calling the constructor,
the program will call the **default construtors** to initialize the members,
and then enter the body of the constructor.Therefor, all the work performed
in the default constructors were wasted.

Using the initilization list instead of the assignment will be more efficient.

    #include <iostream>

    class Girl {
    public:
        Girl(std::string name, int age)
    	    :name_(name), age_(age)
        { }
    private:
        std::string name_;
        int age_;
    };
    
If the member is **const** or **reference**, you must use initialization list.

For non-local static objects defined in different translation units
--------------------------------------------------------------------

How long the title was!

**static objects** include *global objects*, *objects defined at namespace*,
*objects defined static inside classes*, *objects defined static inside
functions* and *objects declared static at file scope*.

*objects defined static inside functions* are called **non-local static objects**.

A **translation unit** is a single source file plus all of its ``#include``files.

Given an example.

main.h:

    #ifndef _MAIN_H_
    #define _MAIN_H_
    
    class Girl {
    public:
        Girl(std::string name, int age)
    	    :name_(name), age_(age)
        { }
        std::string
        get_name() const
        {
    	    return "Mary";
        }
    private:
        std::string name_;
        int age_;
    };
    
    #endif /* _MAIN_H_ */
    
main.c:

    #include <iostream>
    #include "main.h"
    
    extern Girl wife;
    
    class Man {
    public:
        Man(std::string name, std::string wife_name)
    	    :name_(name), wife_name_()
        {
    	    wife_name_ = wife.get_name();
        }
    private:
        std::string name_;
        std::string wife_name_;
    };
    
    int
    main(int argc, char **argv)
    {
        Man m("Tom", "");
    
        return 0;
    }
    
``wife`` is a non-local static object.In the constructor of class ``Man``,
``wife`` will be used before it is initialized since **the relative order**
**of initialization of non-local static objects defined in different **
**translation units is undefined**.

To let ``wife`` be initialized before used, change it to **local static**.

main.h:

    #ifndef _MAIN_H_
    #define _MAIN_H_
    
    class Girl {
    public:
        Girl()
        { }
        
        Girl(std::string name, int age)
    	    :name_(name), age_(age)
        { }
        
        std::string
        get_name() const
        {
    	    return "Mary";
        }
    private:
        std::string name_;
        int age_;
    };
    
    #endif /* _MAIN_H_ */
    
main.cpp

    #include <iostream>
    #include "main.h"
    
    class Man {
    public:
        Man(std::string name, std::string wife_name)
    	    :name_(name), wife_name_()
        {
    	    wife_name_ = wife().get_name();
        }
        Girl&
        wife()
        {
    	    static Girl w;
    
            return w;
        }
    private:
        std::string name_;
        std::string wife_name_;
    };
    
    int
    main(int argc, char **argv)
    {
        Man m("Tom", "");
    
        return 0;
    }
    
The so-call *Factory pattern* is just like the above code.

