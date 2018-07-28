+++
date = "2016-05-14T16:35:19+08:00"
draft = false
title = "About Constructors Destructors and Assignment Operators"
+++

What functions C++ will silently write and call?
--------------------------------------------------

If we declare an empty class, the compiler will declare a constructor,
a copy constructor, a copy assignment operator and a destructor for us.

    class Girl {

    };

But the compiler is not foolish, it will only generate them when needed.

What do the generated functions do?

* For construtor, it will invoke the corresponding constructor and non-static
members of its base class.

* For destructor, it will invoke the corresponding destructor of its base class.
If its base class's destructor is virtual, the destructor will declare as
virtual.

* For copy constructor and copy assignment operator, they will copy each
non-static data member from the source object to the target object.

If the class has some members that are pointer, reference or constant, there
will be some trouble with the compiler.Therefore, always define the four
functions when we define our own class.

Disallow the use of generated functions you don't want
---------------------------------------------------------

Sometimes, we may not want the copy constructor or the copy assignment
operator because every member may be different with the other object's
members.Preventing the generation of these functions will make the
program more effecient.

If we don't declare these two functions, the compiler will generate them
for us.

So how can we do?

The prefer solution is **declare the copy constructor and copy assignment**
**operator as private and never implement them**.

If we implement them, the members and friends of the class can call them.But
if we don't implement them, the linker will complain at it.

In order to move the link-time error to the compile time, we should let the
members and the friends of the class cannot call them.One method is to define
a base class that can't be copied and inherited by the class that you don't
want it to be copyable.

    class Uncopyable {
	protected:
	    Uncopyable()
		{ }
		~Uncopyable()
		{ }
	private:
	    Uncopyable(const Uncopyable&);
		Uncopyable& operator=(const Uncopyable&);
	};

    class Person: private Uncopyable {

    };

The copy constructor and copy assignment operator of the class ``Uncopyable``
are declared as private, so the members and friends of class ``Person`` can't
call them.


Declare destructors virtual in polymorphic base classes
---------------------------------------------------------

I have met this problem in an interview.

Suppose the destructor of the base class is not virtual.

    class Dog {
	public:
	    Dog();
		~Dog();
	};

And there are two class inherited from it.

    class Whippet:public Dog {

    };

    class Spaniel:public Dog {

    };

We can use a base class pointer to handle the derived classes.

    Dog *aDog = new Whippet();
	...

Then you should delete the pointer when you want to quit.

    delete aDog;

Now, the problem appears.The C++ specifies that when a derived class
object is deleted through a pointer to a base class with a non-virtual
destructor, results are undefined.

That is, the destructor of the base class will be called typically, but
the destructor of the derived class may not be called.

I have written a simple program to test it.

    #include <cstdio>
    
    class Dog {
    public:
        Dog()
        { }
        ~Dog()
        {
    	    printf("Call Dog's destructor\n");
        }
    private:
        Dog(const Dog&);
        Dog&
        operator=(const Dog&);
    };
    
    class Whippet:public Dog {
    public:
        Whippet()
        { }
        ~Whippet()
        {
    	    printf("Call Whippet's destructor\n");
        }
    };
    
    int
    main(int argc, char **argv)
    {
        Dog *aDog = new Whippet();
        delete aDog;
    
        return 0;
    }
        
I run the program several times and the results are the same.

    ~/test $ ./test 
	Call Dog's destructor

Now, if I change the destrutor to ``virtual``, the result is:

    virtual ~Dog()
    {
	    printf("Call Dog's destructor\n");
    }

    ~/test $ ./test 
	Call Whippet's destructor
	Call Dog's destructor

Therefore, when the base class have virtual member functions, you should
always make the destructor virtual.

However, not every destructor of any class should be virtual.Making it
virtual will occupy some additional information(virtual table pointer)
that can increase the size of an object of that class.

**Prevent exceptions from leaving destructors**

**Never call virtual functions during construction or destruction**

I have seen the assignment operator in this form many times.

    Person& operator=(const Person& p)
	{
	    ...
		return *this;
	}

First, the parameter is passed by const-reference, which is more effecient.
Second, the function returns a reference of the object.This is more effecient
when doing this.

    p3 = p2 = p1;

Remember that always write assignment operator in this form.

This is also appropriate with ``+=``, ``*=`` and so on.

Sometimes we may assignment to the object itself.

    Person p;
	p = p;

It seems impossible, but how about this.

    persons[i] = persons[j];
	*p1 = *p2;

``persons[i]`` and ``persons[j]`` may be the same.``p1`` and ``p2`` may point
to the same object.

Some assignment operator may be like this one.

    Disk&
	Disk::operator=(const Disk& d)
	{
	    delete data;
		data = new Disk(*(d.data));

        return *this;
	}

It seems reasonable that delete the original data and then allocate a
new one using the data of ``d``.

However, this is very dangerous.What if ``this`` is the same as ``d``?
If that happened, the content of ``data`` and ``d.data`` is the same
thing.So the content of ``d.data`` have been delete before call the
copy constructor.

A direct solution to this problem is obvious.That is, just check if they
are the same.

    Disk&
	Disk::operator=(const Disk& d)
	{
	    if (this == &d) {
		    return *this;
		}
	    delete data;
		data = new Disk(*(d.data));

        return *this;
	}
    
I prefer the above solution.But there are another solution.

    Disk&
	Disk::operator=(const Disk& d)
	{
	    Disk *origin = data;
		data = new Disk(*(d.data));
		delete origin;
		
        return *this;
	}
    
This code just change the order of some statement, but it make great
difference.

**When adding a member to a class, remember to update the constructors,**
**destructor, copy constructor, copy assignment operator**.

**Don't miss anyone!**

When a class is a derived class, make sure to call the constructors,
copy constructor and copy assignment operator of the base class, respectively
when writing my own constructors, copy constructor, assignment operator.

**Do not call copy constructor in the copy assignment operator.**
**Do not call copy assignment operator in the copy constructor.**

That is all about constructors, copy constructor and copy assignment operator.
It really helps.





