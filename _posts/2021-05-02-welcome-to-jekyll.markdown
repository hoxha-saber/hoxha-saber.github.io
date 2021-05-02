---
layout: post
title:  "Smart pointers in C++"
date:   2021-05-02 15:42:12 -0400
categories: jekyll update
description: A tour of smart pointers in C++14
---
Since C++ is has inherited alot of its features from C, people seem to have this misconception that modern C++ is hard to use due to pointers and memory management. In this post, I'll go over some useful features that are included in the standard library (C++11 and ownward).


# unique_ptr
First lets go over the dangers of using raw pointers. Consider the code snippet below:

{% highlight c++ %}
void f()
{
  int *p = new int {3};
  if (some_condition)
  {
    throw SomeException{};
  }
  delete p;
}
{% endhighlight %}

When the function `f()` is called, we allocate some memory on the heap to store an `int`. If the condition `some_condition` is false, then the function will simple reclaim that memory and exit (what an intersting function!). But, bad things happen when `some_condition` is true. When `bad_condition` is true, the function throws an execption. Upon throwing an exception, the stack unwinding process begins. In this case, the stack frame belonging to the function `f()` gets destroyed, along with it the pointer `p`. We lost our only reference to the block of memory we just allocated on the heap! We therefore get a memory leak.

Our goal is to make sure that whatever happens in the function, the pointer `p` will be properly deleted. To do this, lets create a class that will encapsulate the idea of a pointer. Since pointers can be of almost any type, we should make sure this class is templated.

{% highlight c++ %}
template <typename T> class unique_ptr
{
  private:
    T *p;
  public:
    unique_ptr(T *p) : p{p} {} // constructor
    ~unique_ptr() {delete p; } // destructor
    T *get() const {return p; }
    T *release()
    {
      T *q = p;
      p = nullptr;
      return q;
    }
};
{% endhighlight %}

Now using the `unique_ptr` is the previous code snippet:

{% highlight c++ %}
void f()
{
  unique_ptr<int> p {new int {3}};
  if (some_condition)
  {
    throw SomeException{};
  }
}
{% endhighlight %}

That's it! Whenever an exception is ever called, the stack will be popped and the destructor of every object will also be called. But in this case, the desctructor of `unique_ptr` deletes the pointer is owns. As result, no memory leaks! There are a few problems with our current implementation however.

First of all, we are missing critical pointer features like dereferencing and `->` support. Second of all, what happens if we make many `unique_ptr` point to the same block of memory (shown below)?

{% highlight c++ %}
void f()
{
  unique_ptr<int> p {new int {3}};
  unique_ptr<int> q = p;
  if (some_condition)
  {
    throw SomeException{};
  }
}
{% endhighlight %}

When one of `p, q` gets deleted first, the destruction of the other one will result in deleting a pointer that has already been deleted. This is undefined behaviour!

To add the critical pointer features is merely some syntatic sugar:

{% highlight c++ %}
template <typename T> class unique_ptr
{
  private:
    T *p;
  public:
    ...
    T &operator*() const {return *p; }

    T *operator->() const return {p; }
    ...
};
{% endhighlight %}

Now what about the copying issue. Well, we simply need to disallow the copying (moving will be fine):

{% highlight c++ %}
template <typename T> class unique_ptr
{
  private:
    T *p;
  public:
    ...
    unique_ptr(const unique_ptr &other) = delete; // copy ctor
    unique_ptr &operator=(const unique_ptr &other) = delete; // =operator
    unique_ptr (unique_ptr &&other) : p{other.p} {other.p = nullptr}; // move ctor

    unique_ptr &operator=(unique_ptr &&other) // move =operator
    {
      swap(p, other.p);
      return *this
    }

    ...
};
{% endhighlight %}

And that's it! Since we added `=delete`, whenever someone tries to copy the unique_ptr, a compiler error will be raised. The `unique_ptr` should be unique after all (it's in the name).

---

# shared_ptr

In the previous section, we made a cool class that allowed us to use pointers and memory management without a big hassle. While `unique_ptr` has many practical uses, one big drawback is that we can't make copies of it. In some applications like graph theory, there will of course be many references to the same node (through neighbours). So, we're gonna need to circumvent this drawback. I'm glad to present you, the `shared_ptr`!.

With `unique_ptr`, the problem was we risk deleting the same ptr twice when we had copies of it. We need a way to ensure that we only delete the pointer when the last instance of the `shared_ptr` containing it is destroyed. To accomplish this, we will have a reference counter that counts how many current instances of this `shared_ptr` exist. Whenever a new one is created, we increment this counter, and whenever an instance is destroyed we will decrement the counter. Whichever instance has reference count of zero upon destruction will be the one to call `delete` on the underlying pointer. Onto the implementation!