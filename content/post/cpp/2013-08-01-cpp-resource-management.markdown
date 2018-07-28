+++
date = "2016-05-14T16:35:19+08:00"
draft = false
title = "Cpp Resource Management"
+++


I always forget freeing a memory that allocated from the heap.And I always
forget closing the file handler.

Resources include file descriptors, mutex locks, fonts, brushes, database
connections and network sockets.

I think less programmers can remember closing a file handler, especially
in a large project.

Nowadays, many programming languages have the garbage collection function.
In C++, since the destructor will be automatically called, we can release
the resources automatically if we put the resource into an object.

How can do that?Use the *smart pointer*:``std::auto_ptr``.

    #include <cstdio>
    #include <memory>
    
    int
    main(int argc, char **argv)
    {
        int *num = NULL;
        {
    	    num = new int();
    		std::auto_ptr<int> ap(num);
        }
        // delete num;
        
        return 0;
    }
        
If trying to delete num after the block, that will be a double free error.

Reference from [cplusplus.com](http://www.cplusplus.com/reference/memory/auto_ptr/), ``auto_ptr`` is deprecated in C++11 and is replaced by ``unique_ptr``.
But they are similar.

However, multiple ``auto_ptr`` cannot contain the same object.

Here is the test program.

    #include <cstdio>
    #include <memory>
    
    int
    main(int argc, char **argv)
    {
        int *num = NULL;
        {
    	    num = new int();
    		std::auto_ptr<int> ap1(num);
    		std::auto_ptr<int> ap2(ap1);
    
            printf("%p\n", &ap1);
    		ap1 = ap2;
    		printf("%p\n", &ap2);
        }
        // delete num;
        
        return 0;
    }

And here is the gdb debug output information.

    Temporary breakpoint 1, main (argc=1, argv=0x7fffffffe6e8) at test.cpp:7
    7	    int *num = NULL;
    (gdb) n
    9		num = new int();
    (gdb) n
    10		std::auto_ptr<int> ap1(num);
    (gdb) n
    11		std::auto_ptr<int> ap2(ap1);
    (gdb) p ap1
    $1 = {_M_ptr = 0x601010}
    (gdb) p ap2
    $2 = {_M_ptr = 0x7fff00000001}
    (gdb) n
    13		printf("%p\n", &ap1);
    (gdb) p ap1
    $3 = {_M_ptr = 0x0}
    (gdb) p ap2
    $4 = {_M_ptr = 0x601010}
    (gdb) n
    0x7fffffffe5e0
    14		ap1 = ap2;
    (gdb) p ap1
    $5 = {_M_ptr = 0x0}
    (gdb) p ap2
    $6 = {_M_ptr = 0x601010}
    (gdb) n
    15		printf("%p\n", &ap2);
    (gdb) p ap1
    $7 = {_M_ptr = 0x601010}
    (gdb) p ap2
    $8 = {_M_ptr = 0x0}
    (gdb) 

But there are another smart pointer that can do this.It's the
``std::tr1::shared_ptr``.Mutiple shared_ptr can contain the same object.

The testing program is as follow.

    #include <cstdio>
    #include <tr1/memory>
    
    int
    main(int argc, char **argv)
    {
        int *num = NULL;
        {
    	    num = new int();
    		std::tr1::shared_ptr<int> ap1(num);
    		std::tr1::shared_ptr<int> ap2(ap1);
    
    	    printf("%p\n", &ap1);
    		ap1 = ap2;
    		printf("%p\n", &ap2);
        }
        
        return 0;
    }
        
And the gdb debug information is:

    Temporary breakpoint 1, main (argc=1, argv=0x7fffffffe6e8) at test.cpp:7
    7	    int *num = NULL;
    (gdb) n
    9		num = new int();
    (gdb) n
    10		std::tr1::shared_ptr<int> ap1(num);
    (gdb) n
    11		std::tr1::shared_ptr<int> ap2(ap1);
    (gdb) n
    13		printf("%p\n", &ap1);
    (gdb) p ap1
    $1 = std::tr1::shared_ptr (count 2, weak 0) 0x602010
    (gdb) p ap2
    $2 = std::tr1::shared_ptr (count 2, weak 0) 0x602010
    (gdb) n
    0x7fffffffe5d0
    14		ap1 = ap2;
    (gdb) n
    15		printf("%p\n", &ap2);
    (gdb) p ap1
    $3 = std::tr1::shared_ptr (count 2, weak 0) 0x602010
    (gdb) p ap2
    $4 = std::tr1::shared_ptr (count 2, weak 0) 0x602010

We can see that both of ``ap1`` and ``ap2`` point to the same object.

The ``auto_ptr`` and ``shared_ptr`` use ``delete`` but not ``delete[]``,
so they don't support array.But the ``unique_ptr`` support.

Sometimes, the resource may be a mutex, and we don't want to remember
unlocking the mutex every time we lock it.So a resource management object
may be:

    #include <cstdio>
    
    typedef int Mutex;
    
    void
    lock(Mutex *p)
    {
        printf("Locking...\n");
    }
    
    void
    unlock(Mutex *p)
    {
        printf("Unlocked\n");
    }
    
    class Lock {
    public:
        explicit Lock(Mutex *p)
    	    :pMutex(p)
        {
    	    lock(pMutex);
        }
        ~Lock()
        {
    	    unlock(pMutex);
        }
    private:
        Mutex *pMutex;
    };
    
    int
    main(int argc, char **argv)
    {
        Mutex m;
        {
    	    Lock ml(&m);
        }
        
        return 0;
    }

However, when we copy the ``Lock`` object, problems appear.

    Lock ml2(ml);

The default copy constructor will directly copy the pointer ``pMutex`` to
the target object.So we should let the object uncopyable.

    private:
    Lock(const Lock&);
    Lock&
    operator=(const Lock&);
    
    Mutex *pMutex;

Multiple objects can use the same resource, so the resource may be existed
until the last object has been destroyed.The ``tr1::shared_ptr`` provide a
``deleter`` and its **shared ability** to solve this problem.

    class Lock {
    public:
        explicit Lock(Mutex *p)
    	    :pMutex(p, unlock)
        {
    	    lock(pMutex.get());
        }
    private:
        Lock(const Lock&);
        Lock&
        operator=(const Lock&);
        
        std::tr1::shared_ptr<Mutex> pMutex;
    };
    
Provide access to raw resources
---------------------------------

Sometimes we may only want to access the raw resource but the object that
contains the resource.For example:

    #include <cstdio>
    #include <tr1/memory>
    
    typedef int Mutex;
    
    static Mutex *
    createMutex()
    {
        static Mutex *p = new Mutex();
    
        return p;
    }
    
    int
    main(int argc, char **argv)
    {
        std::tr1::shared_ptr<Mutex> pMutex;
    
        printf("%d\n", pMutex);
        
        return 0;
    }
        
Therefore, we must provide a method to access the raw resource.Like this.

        printf("%d\n", pMutex.get());

There are two ways, one is explicit and the other is implicit.

    class MutexManager {
    public:
        explicit MutexManager(Mutex* p)
    	    :p_(p)
        { }
        ~MutexManager()
        {
    	    delete p_;
        }
        Mutex
        get() const
        {
    	    return *p_;
        }
    private:
        Mutex *p_;
    };
    
    int
    main(int argc, char **argv)
    {
        MutexManager mm(createMutex());
        
        printf("%d\n", mm.get());
        
        return 0;
    }
        
This one above is exciplit conversion.And this one below is implicit.

    class MutexManager {
    public:
        explicit MutexManager(Mutex* p)
    	    :p_(p)
        { }
        ~MutexManager()
        {
    	    delete p_;
        }
        operator Mutex() const
        {
    	    return *p_;
        }
    private:
        Mutex *p_;
    };
    
    void
    print(Mutex m)
    {
        printf("%d\n", m);
    }
    
    int
    main(int argc, char **argv)
    {
        MutexManager mm(createMutex());
        
        print(mm);
        
        return 0;
    }
        
It's obviously that the explicit method is safer and the implicit method is
more convenient for clients.

Keep new and delete in the same form
----------------------------------------

We all know that the following code is wrong.

    std::string *str = new std::string[10];
	delete str;

We should use ``delete []str;`` instead of ``delete str;``.

But how about this?

    #include <iostream>
    
    typedef std::string Lines[4];
    
    int
    main(int argc, char **argv)
    {
        std::string *p = new Lines;
        delete p;
    
        return 0;
    }
    
If let me delete the pointer p, I will use this ``delete p;``.But I am wrong.
I shouldn't look at the left of the pointer, that is, ``std::string``.I should
look at the right of the ``new``, that is, ``Lines``.So, in this case, we
should use ``delete []p;``.

Avoid typedef for array types, use ``vector`` instead.




