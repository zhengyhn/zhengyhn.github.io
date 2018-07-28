+++
date = "2016-05-14T16:35:19+08:00"
draft = false
title = "Some reviews about C plus plus"
+++

Declaration and Definition
===========================

I have already known the difference between **declaration** and **definion**,
but now I find that my understanding is not complete.

**Declaration** just tells the compiler about the name and the type of
something.It's very common about the declaration of *objects* and *functions*.

    extern int a;

    int max(int a, int b);

However, I never think about the declaration of *class*.Recently, I have to
view some codes in a large project and I have found many of these declarations.

    class Option;
	class Parameter;

Additionally, there are many declarations of the *template class*.

    template<typename T>
	class Swap;

Another interesting thing is that the official C++ definition of
*function signature* excludes the return type.I think this is strange and I
will consider it, anyway.

**Definition** gives the details of something to the compiler.In the past, I
hold the opinion that *definition* always be related to *memory*.In fact, it is
not allways.
- For object, the definition tell the compiler about the memory asigned for the
object.
- For function or function template, the definition provides the function body.
- For class or class template, the definition lists the members of the class.

For example:

    int a;

    int max(int a, int b)
	{
	    return a > b ? a : b;
	}

    class Option {
	public:
	    Option();
		~Option();
		...
	};

Defaul constructor
===================

I found I can't tell whether a constructor is a **default constructor** or not.
But now, I know.A **default constructor** is a constructor that can be called
without any arguments.That is, either no parameters or every parameter is
initialized.

The following are default constructors.

    Option();
	explicit Option(bool short = true);

And this one is not a default constructor.

    explicit Option(bool short);

What the hell **explicit** is doing here?I have never used this reserved word
even if I have learned C++ for several years.But this guy, *explicit*, is of
great importance.

Placing *explicit* before the constructor can prevent the objects of the class
being used to perform implicit type conversions.Obviously, they can be used
for explicit type conversions.

For example, if we have two classes and a function.

    class Dog {
	    explicit Dog();
		...
	}

    Dog aDog;
	Cat aCat;
	
    void kill(Dog d);
	
Now call the ``kill`` function passing ``aDog`` as parameter.
    kill(aDog);    // No problem.

If with the ``explicit`` reserved word and call the ``kill`` function passing
``aCat`` as parameter, it will be error since it prevents implicit type
conversions.

	kill(aCat);    // Error, prevent implicit conversion.

But if without the ``explicit`` reserved word and call the ``kill`` function
passing ``aCat`` as parameter, it will lead the compiler to perform unexpected
type conversions.So the policy is:

**Always declare the constructor as explicit.**


Copy constructor and copy assignment operator
==============================================

I have already known that this

    Parameter p1;

will invoke the *default constructor* and this

    Parameter p2(p1);

will invoke the *copy constructor* and this

    p1 = p2;

will invoke the *copy assignment operator*, but I have never known that this

    Parameter p3 = p2;

will not invoke the *copy assignment operator* but the *copy constructor*.

This amazes me very much.But now I can distinguish it easily.

If a new object is being defined, invoke the *copy constructor*.Otherwise,
invoke the *copy assignment operator* since it's just the **asignment**.

Some common sense
==================

* TR1, Technical Report 1, is a specification for new functionality in
C++ standard library.


This really deepens my understanding of C plus plus and now I am very
interested in this great programming language.

