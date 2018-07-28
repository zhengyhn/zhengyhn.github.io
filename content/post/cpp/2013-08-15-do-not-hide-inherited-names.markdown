+++
date = "2016-05-14T16:35:19+08:00"
draft = false
title = "Do not hide inherited names"
+++

In inheritance, there are some rules defined by C++, one of which is that
the virables or the functions in the base class will be hidden if there
are virables and functions with the same name.

It seems perfect, but it's a pitfall of C++. Consider the following example.

    #include <cstdio>
    #include <iostream>
    
    class Person {
    public:
        explicit Person()
        { }
		~Person()
		{ }
        void
        sleep() const
        { printf("Person sleep\n"); }
        void
        sleep(const int sec) const
        { printf("Person sleep %d s\n", sec); }
    private:
        Person(const Person&);
        const Person&
        operator=(const Person&);
    };
    
    class Student:public Person {
    public:
        explicit Student()
        { }
		~Person()
		{ }
        void
        sleep() const
        { printf("Student sleep\n"); }
    private:
        Student(const Student&);
        const Student&
        operator=(const Student&);
    };
    
    int
    main(int argc, char **argv)
    {
        Student stu;
    
        stu.sleep();
        stu.sleep(1);
        
        return 0;
    }

This program won't be compiled. It shows the following error:

    test.cpp:39:16: error: no matching function for call to ‘Student::sleep(int)’
         stu.sleep(1);
                    ^
    test.cpp:39:16: note: candidate is:
    test.cpp:25:5: note: void Student::sleep() const
         sleep() const
         ^
    test.cpp:25:5: note:   candidate expects 0 arguments, 1 provided

That is because C++ will hide all the names in the base class as long as there
are the same names in the derived class. In the ``Person`` class, ``sleep()``
and ``sleep(const int sec)`` have the same name. In the ``Student`` class, it
only want to override the ``sleep()`` function, but the overriding cause the
hiding of the ``sleep(const int sec)`` function!

So how to solve this problem? There are two methods so far.

1. Don't hide the names in the base class will using overloading functions.
2. Use ``using`` directive to make the certain methods visible.

        class Student:public Person {
		public:
            using Person::sleep;
			
			explicit Student()
			{ }
			~Person()
			{ }
		    void
			sleep() const
			{ printf("Student sleep\n"); }

Finally, another important thing should be stated. In the above example, I
implement ``sleep()`` function in the definition of the class. Why do that?
I have worked in some very large C++ projects and there are many implementation
in the definition. In fact, this is so-called **implicit inline**. The compiler
will inline the functions that are implemented in the class definition
automatically. Therefore, it's good to implement some small functions in the
definition of class. And let the large function implement in another file.
This will make my code more effecient and I will follow this guide in the
future.


    
