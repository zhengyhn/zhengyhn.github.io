+++
date = "2016-05-14T16:35:19+08:00"
draft = false
title = "My C++ coding style"
+++

Every programmer has his/her own coding style. Time goes by, and my coding
style is changing. There following is my note on c++ coding style. I will
explain why I want to coding in that style.

Why do I care about my coding style so much? Because I am a virgo.

* **Always add a ``m_`` prefix to the members and use ``this`` to refer to
other member functions in the member function.**

This seems ridiculous. But I prefer. In a member function, there may be
various kinds of variables, such as memeber variables, global variables
and local variables. If we don't add some flag to refer to the member
variables, we will confused when we met a variable. Is it a local variable
or a member variable? For example, there is a statement in a member function.

    size1 = size2 + 1;

How can you know what this statement exactly means? Which ``size`` is the
class's size? However, if we write this statement as this,

    m_size1 = size2 + 1;

we can know that we set the class's size be the value of local variable ``size``
plus one. This code is readable and it can relieve the code reader's headache.

For the member functions. When refering to a member function, always use
``this``.

    int *p = this->next();

Someone may argue that using ``this`` will let use write more code. My
experience told me that the whole world is balanced. You make yourself happy
and your code will make the code readers headache. Instead, you write readable
codes with some extra typing and you will make the code readers happy. This is
a balance and you can balance yourself. I am a so kind man and I always want
those who read my codes be happy. So I will use ``this`` always.

Note: In PHP, you must use ``$this`` to refer to members. In Python, you must
use ``self`` to refer to members. In most of the source code in **Android**,
members in C++ class is always with prefix ``m_`` and members in JAVA class is
always with prefix ``m``.

* **Always leave the return type name in a single line.**

I used to leave the return type name on the left of a function name. The first
time I saw some programmers put them in a single line is the
[Pintos](http://en.wikipedia.org/wiki/Pintos) project. I thought it's ugly.
But after I worked on the code of [Gold](http://en.wikipedia.org/wiki/Gold_(linker)),
I changed my mind. I followed the author's style, even in a main function.

    int
    main(int argc, char *argv[]) { }

Why do I leave the return value in a line? Because, in many large project, most
of the names of the return types are very long, most of the function names are
very long and most of functions has several parameters which have a long type
name. For example,

    std::map<size_t, LinkedList<T> *> *
    get_map(const std::string& name, bool is_global=true) const;

If you don't leave the return type name in a single line, the function
declaration will becomes

    std::map<size_t, LinkedList<T> *> * get_map(const std::string& name, bool is_global=true) const;

which is very very ugly!

If we leave the return type name in a single line, the code readers will see
the return type of the function more quickly. So the code will be more readable.

Someone may argue that it looks more beautiful to leave the return type name on
the left of the function name when the type name is short like this.

    int get(size_t index);

This may be right. But most of the return type name is long in a large project.
So for consistence, I always leave them in a single line.

* **Use ``return`` or ``continue`` to avoid indentation**

The code will look ugly when there are many indentations. For example,

    if (arr != NULL && begin < end - 1) {
        ...
        while (top > 0) {
            ...
            for (int i = begin; i < end - 1; ++i) {
                if (arr[i] < arr[pivot]) {
                    for (int j = i;  j < end; ++j) {
                        ...
                    }
                }
            }
        }
        ...
    }

There are **5** level of indentations, which is very ugly. However, we can use
``return`` and ``continue`` to make it look better.

    if (arr == NULL || begin >= end - 1) {
        return;
    }
    ...
    while (top > 0) {
        ...
        for (int i = begin; i < end - 1; ++i) {
            if (arr[i] >= arr[pivot]) {
                continue;
            }
            for (int j = i;  j < end; ++j) {
                ...
            }
        }
    }
    ...

Now, it's only **3** level of indentations, which looks more beautiful.
This is an important skills to make my code more beautiful.

