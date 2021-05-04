---
layout: post
title:  "Curiously recurring template pattern"
date:   2021-05-04 14:15:12 -0400
categories: c++
description: Design pattern unique to C++
---
# Background
Design patterns are usually meant to be language agnostic. This makes designs easier to learn and more portable. However, each language has features that make them unique. In this post, we'll take a closer look at templates in C++ and the _Curiously Recurring Template Pattern._

# The problem
With every design pattern, there was a problem before it that needed to be resolved. So let's go over some code snippets and identify issues that could arise.

Often enough, we want to keep track of the objects we create. We'd also like to possible know more information about the usage of the class itself, not a specific object. To keep it simple for this example, let's say I want to keep track of how many `Orange`s we make. Here's one way to do it
{% highlight c++ %}
class Orange
{   private:
        static int count; // This is associated with the class, not the object itself
    public:
        Orange(/* some params go here */) { ++count;}
        // ...
        static int getCount() {return count; } 
};

Orange::count = 0; // Need to originally set the variable

Orange o1{...};
Orange o2{...};
Orange o3{...};

cout << Orange::getCount() << endl;  // Prints 3
{% endhighlight %}

This will certainly do the job for counting how many instances of `Orange` are created. However, it cannot be reused for other classes, and if we ever have additional statistics, we will have to keep updating the same code. We need a better way to do this.

# A solution
Here is where Curiously Recurring Template Pattern (CRTP) comes up. Templates are ubiquitous in C++, so we could try to template this idea into its own templated class.
{% highlight c++ %}
template<typename T> class StatCounter 
{
    static int count;
    Count() {++count;}
    Count(const Count&) {++count;}
    Count(Count &&) {++count;}
    ~Count() { --count; }
    static int getCount() {return count;}
};

template<typename T> int Count<T>::count = 0;
{% endhighlight %}

Now to extend this to other classes, and keep track of how many instances are created at a given time we can use inheritance.
{% highlight c++ %}
class Orange : StatsCounter<Orange>
{
    public:
        Orange(...) {...}
        using StatsCounter<Orange>::getCount; 
};
{% endhighlight %}
So how does this work? For any class `C` we make, we get that `StatCounter<C>` is a unique template instantion of `StatCounter` which will track every contruction/destruction of `C`. In general, CRTP essentially does two things:
1. Inheriting from a template class
2. Use the derived class itself as a template parameter of the base class

# What's the catch?
Every design comes with its own flaws and CRTP is no different. In this case, the derived classes don't share a common base class. This is more easily identified by the example below:
{% highlight c++ %}
template<typename T> class Shape
{
    ...
};

class Square : Shape<Square> {...};
class Circle : Shape<Circle> {...};
{% endhighlight %}
Unforturnately, `Square` and `Circle` cannot be stored homogenously. That is, that can't be stored in the same container. If I wanted to make a `std::vector` containing both `Square` and `Circle`, I won't be able to. The reason is that `Square` inherits from `Shape<Square>` and `Circle` inherits from `Shape<Circle>` hence the two classes share no common base class. 