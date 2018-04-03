https://stackoverflow.com/questions/569775/smart-pointers-boost-explained

**1. What is the difference between the following set of pointers?
2. When do you use each pointer in production code, if at all?**

Examples would be appreciated!

..1. scoped_ptr
..2. shared_ptr
..3. weak_ptr
..4. intrusive_ptr

**3. Do you use boost in production code?**

----------------------------------------------------------------

# Basic properties of smart pointers

It's easy when you have **properties** that you can assign each smart pointer. There are three important properties.

**1) no ownership at all**

**2) transfer of ownership**

**3) share of ownership**

**The first** means that a 'smart pointer' cannot delete the object, because it doesn't own it.

**The second** means that only one 'smart pointer' can ever point to the same object at the same time. If the smart pointer is to be returned from functions, the ownership is transferred to the returned 'smart pointer', for example.

**The third** means that multiple 'smart pointers' can point to the 'same object' at the same time. This applies to a **raw pointer** too, however 'raw pointers' lack an important feature: *They do not define whether they are owning the object or not.* A share of 'ownership smart pointer' will delete the object, if every owner gives up the object. This behavior happens to be needed often, so 'shared owning smart pointers' are widely spread.

**The non-mentioned fourth type -** Some 'owning smart pointers' support neither the second nor the third. They can therefore not be returned from functions or passed somewhere else. Which is most suitable for RAII purposes, where the smart pointer is kept local and is just created so it frees an object after it goes out of scope.

# Implementing share of ownership in c++

1) Share of ownership can be implemented by having a copy constructor. This naturally copies a smart pointer and both the copy and the original will reference the same object. 
2) Transfer of ownership cannot really be implemented in C++ currently, because there are no means to transfer something from one object to another supported by the language: If you try to return an object from a function, what is happening is that the object is copied. So a smart pointer that implements transfer of ownership has to use the copy constructor to implement that transfer of ownership. However, this in turn breaks its usage in containers, because requirements state, a certain behavior of the copy constructor, of elements of containers, which is incompatible with this so-called "moving constructor" behavior of these smart pointers.
3) **[Advanced]** *C++1x* provides native support for transfer-of-ownership by introducing so-called **"move constructors"** and **"move assignment operators"**. It also comes with such a transfer-of-ownership smart pointer called **unique_ptr**.

# Categorizing smart pointers

**1) scoped_ptr** is a smart pointer that is neither transferable nor sharable. It's just usable if you locally need to allocate memory, but be sure it's freed again when it goes out of scope. That is, a scoped_ptr will be destroyed when it goes out of scope. But it can still be swapped with another scoped_ptr, if you wish to do so.

**2) shared_ptr** is a smart pointer that shares ownership (third kind above). It is reference counted so it can see when the last copy of it goes out of scope and then it frees the object managed.

**3) weak_ptr** is a non-owning smart pointer.**Normally, you would need to get the raw pointer out of the shared_ptr and copy that around.** But that would not be safe, as you would not have a way to check when the object was actually deleted. So, weak_ptr provides means by referencing an object managed by shared_ptr, without adding a reference count. If you need to access the object, you can lock the management of it (to avoid that in another thread a shared_ptr frees it while you use the object) and then use it. If the weak_ptr points to an object already deleted, it will notice you by throwing an exception. **Using weak_ptr is most beneficial when you have a cyclic reference**: Reference counting cannot easily cope with such a situation.

**4) intrusive_ptr** is like a shared_ptr, but it does not keep the reference count in a shared_ptr - but leaves incrementing/decrementing the count to some helper functions, that need to be defined by the object that is managed. This has the advantage that an already referenced object (which has a reference count incremented by an external reference counting mechanism) can be stuffed into an intrusive_ptr - because the reference count is not anymore internal to the smart pointer, but the smart pointer uses an existing reference counting mechanism.

**5) unique_ptr** is a transfer of ownership pointer. You cannot copy it, but you can move it by using C++1x's move constructors:

```c++
unique_ptr<type> p(new type);
unique_ptr<type> q(p); // not legal!
unique_ptr<type> r(move(p)); // legal. p is now empty, but r owns the object
unique_ptr<type> s(function_returning_a_unique_ptr()); // legal!
```

This is the semantic that **std::auto_ptr** obeys, but because of missing native support for moving, it fails to provide them without pitfalls. **unique_ptr** will automatically steal resources from a temporary other unique_ptr which is one of the key features of move semantics. auto_ptr will be deprecated in the next C++ Standard release in favor of unique_ptr. C++1x will also allow stuffing objects that are only movable but not copyable into containers. So you can stuff unique_ptr's into a vector for example. I'll stop here and reference you to a fine article about this if you want to read more about this.
