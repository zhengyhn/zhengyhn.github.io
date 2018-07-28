+++
date = "2016-05-14T16:35:19+08:00"
draft = false
title = "Understanding Encapsulation"
+++

Once I loved *C* very much and I thought it's the best programming language
in the world. I used to argue that *C* can be used to implement the
object-oriented design with ``struct``. I used to argue that ``private`` and
``public`` is useless in *C++*. However, now I know I was wrong.

Why we need private
------------------------

It's related to a very important concept in C++, that is, **encapsulation**.

**Encapsulation is not invisibility. It's maintainability.**

Suppose a class is very popular and is used by many projects. For example,
the ``string`` class. If every members in this class is ``public``, the clients
can use many interfaces to manipulate ``string``. One day, when the ``string``
class is going to be modified(some members' name change), then lots of clients
have to modify there code. This is painful! If we declare some of the members
``private`` and change the name of the private members, only the implementations
in the public member functions should be modified. None of the clients need to
change their codes.

Therefore, **public means unchangable and public means unencapsulated**.

The public member functions should not be changed in the future. Since you don't
know how your clients will use your class, don't expose the members that may be
changed in the future to the clients.

Prefer non-member non-friend functions to member functions
---------------------------------------------------------------

Consider the following class.

    class Person {
	public:
	    explicit Person()
		{ }
		~Person()
		{ }
	    void
		say_age() const
		{ printf("%d\n", age); }
		void
		say_name() const
		{ printf("%s\n", name.c_str()); }
	private:
	    Person(const Person&);
		const Person&
		operator=(const Person&);
	    int age;
		std::string name;
	};

Somebody may want to use the functions together, so we add another member
function.

    class Person {
	public:
	    explicit Person()
		{ }
		~Person()
		{ }
		void
		say_age() const
		{ printf("%d\n", age); }
		void
		say_name() const
		{ printf("%s\n", name.c_str()); }
		void
		say() const
		{
		    say_age();
			say_name();
		}
	private:
	    Person(const Person&);
		const Person&
		operator=(const Person&);
		int age;
		std::string name;
	};

This is straightforward, especially for those Java and C# programmers. Every
functions should be in a class and everything is object. It's obvious, right?

However, e have another option, that is, using the non-member function.

    class Person {
	public:
	    explicit Person()
		{ }
		~Person()
		{ }
		void
		say_age() const
		{ printf("%d\n", age); }
		void
		say_name() const
		{ printf("%s\n", name.c_str()); }
	private:
	    Person(const Person&);
		const Person&
		operator=(const Person&);
		int age;
		std::string name;
	};

    void
	person_say(const Person& p)
	{
	    p.say_age();
		p.say_name();
	}

Which is better? It seems that they are the same.

We should use the non-member function instead of the member function.

As we all know, **the less the public member functions and the friend **
 **functions, the greater encapsulation is**, since member functions and
friend functions are the only interface that can access the private members.

In this example, if we add another public member function, we decrease the
encapsulation of the class since this member function can access the private
members of the class. But if we use the non-member non-friend function, it
won't have impact with the class because it's the function outside.

Does it explain that *C++* is not so object-oriented? From Java or C#, those
programmers will say that everything is object and classes are everywhere.
However, they are wrong. Object-oriented princiles state that **data should be**
**encapsulated as possible**. Therefore, prefering non-member non-friend
functions is more *object-oriented*.

Until now, I can say that I really understand the meaning of encapsulation and
why it's so important in object-oriented programming.

