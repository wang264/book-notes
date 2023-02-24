# Chapter 2: Consturctors, Destructors, and Assignment Operators

### **Item 5: Know what functions C++ silently writes and calls**
Empty class is not an "empty class" becasue compiler will generate default constructor, copy constructor, destructor, and copy assignment operator for you if you did not provide them. 
```cpp
class Empty {
    public:
        Empty() { ... }                             // default constructor
        Empty(const Empty& rhs) { ... }             // copy constructor
        ~Empty() { ... }                            // destructor — see below
        // for whether it’s virtual
        Empty& operator=(const Empty& rhs) { ... }  // copy assignment operator
};
Empty e1;       // will call default constructor, and destructor
Empty e2(e1);   // copy constructor
e2 = e1;        // copy assignment operator
``` 
If a constructor is declared in a class, compilers won’t generate a default constructor.

___

```cpp
template<typename T>
class NamedObject {
    public:
        NamedObject(std::string& name, const T& value);
        ... // as above, assume no operator= is declared
    private:
        std::string& nameValue; // this is now a REFERENCE
        const T objectValue; // this is now CONST
};
std::string newDog("Persephone");
std::string oldDog("Satch");
NamedObject<int> p(newDog, 2);
NamedObject<int> s(oldDog, 36); 

p = s; // what should happen to the data members in p?
```
Should the reference be modified?(Opps, C++ does not provide a way to make a reference refer to a different object.) Or the *string* object that `p.nameValue` refer be modified? NO! Also, it is not legal to modify *const* members.

C++ refuse to compile this code, you must define the copy assignment operator yourself if your class members contain *const* or reference. 

___
**Things to Remember**
* Compilers may implicitly generate a class’s default constructor, copy constructor, copy assignment operator, and destructor.
___

### **Item 6: Explicitly disallow the use of compiler-generated functions you do not want.** 
If you do not want the compiler-generated copy constructor, and copy assignmetn operator. For example, you want to following code to not compile. 
```cpp
class HomeForSale{...};
HomeForSale h1;
homeForSale h2;

HomeForSale h3(h1);     //attempy to copy h1, should not compile!
h1 = h2;                //attempt to copy h2, should not compile!
```

```cpp
class HomeForSale{
    public:
        ...
    private:
        ...
        // the name of the function's parameters are omitted, this is OK.
        HomeForSale(const HomeForSale&);        //declarations only
        HomeForSale& operator=(const HomeForSale&);
}
```
Compiler will complaint in the link time. 

```cpp
class Uncopyable {
    // allow construction and destruction of derived objects
    protected: 
        Uncopyable() {} 
        ~Uncopyable() {} 
    // but prevent copying
    private:
        Uncopyable(const Uncopyable&); 
        Uncopyable& operator=(const Uncopyable&);
};

// class no longer declars copy ctor or copy assign. operator
class HomeForSale: private Uncopyable {
...
};
```
Compiler will complaint in compile time. 
___

* By declaring a member function explicitly, you prevent compilers forom generating their own version. 
* By making the function *private*, you keep people from calling it. 
* Also, need to declaring member functions *private* and NOT implementing them. (So other member functions and friend functiosn can not call this private function)
___
* To disallowe functionality, automatically provided by compilers, declare the corresponding member functions private and give no implementations. Using a base class like *Uncopyable* is one way to do this.
___
### **Item 7: Declare destructors virtual in polymorphic base classes.** 
Example: TimeKeeper Class

```cpp
class TimeKeeper {
    public:
        TimeKeeper();
        ~TimeKeeper();
    ...
};
class AtomicClock: public TimeKeeper { ... };
class WaterClock: public TimeKeeper { ... };
class WristWatch: public TimeKeeper { ... };

// returns a poitner to a dynamically allocated object of a class derived from TimeKeeper
TimeKeeper* getTimeKeeper();
```

The function `getTimeKeeper()` is a **factory function** -- a function that returns a base class poitner to a newly created derived class object.
The object return by `getTimeKeeper()` is on the heap, so to avoid leaking memory, we need to delete that object. 

```cpp
TimeKeeper *ptk = getTimeKeeper();  //get dynamically allocated object from TimeKeeper hierarchy
... //use it
delete ptk; //release it to avoid resource leak
```

**The Problem**
* `getTimeKeeper()` returns a pointer to a derived class object (e.g., `AtomicClock`), that object is being deleted via a base class pointer (i.e., a `TimeKeeper* pointer`), and the base class (TimeKeeper) has a non-virtual destructor.
* When a derived class object is deleted through a pointer to a base class with a non-virtual destructor, results are undefined.
* What typically happens at runtime is that the derived part of the object is never destroyed. "partially destroyed" object. 

**The Fix**
give the base class a virtual destructor
```cpp
class TimeKeeper {
    public:
        TimeKeeper();
        virtual ~TimeKeeper();
        ...
};

TimeKeeper *ptk = getTimeKeeper();
...
delete ptk; // now behaves correctly
```

___

```cpp
// bad idea! std::string has a non-virtual destructor
class SpecialString: public std::string { ... };

SpecialString *pss =new SpecialString("Impending Doom");
std::string *ps;
...
ps = pss; // SpecialString* ⇒ std::string*
...
// undefined! In practice,  *ps’s SpecialString resources will be leaked, 
// because the SpecialString destructor won’t  be called.
delete ps; 
```
This might cause trouble if anywhere in an application you somehow convert a pointer to SpecialString into a pointer to *string* and you then use delte on string pointer. The same analysis applies to any class lacking a virtual destructor, including all the STL container types (e.g., vector, list, set, tr1::unordered_map (see Item 54), etc.).

If you’re ever tempted to inherit from a standard container or any other class with a non-virtual destructor, resist the temptation!
___
Sometimes, if you have a class that you would like to be abstract, and you don't have any pure virtual functions. **What to do?**

Declare a pure virtual destructor.
* Since abstruct class is intended to be used as a base class.
* And a base class should have a virtual destructor.
* And a pure virtual function yields an abstract class.
```cpp
class AWOV { // AWOV = “Abstract w/o Virtuals”
    public:
        virtual ~AWOV() = 0; // declare pure virtual destructor
};

AWOV::~AWOV() {} // definition of pure virtual dtor
```
you must provide a definition for the pure virtual destructor.
* The way destructors work is that the most derived class’s destructor is called first, then the destructor of each base class is called. 
* Compilers will generate a call to ~AWOV from its derived classes’ destructors, so you have to be sure to provide a body for the function. If you don’t, the linker will complain.

In summary, the rule for giving base classes virtual destructors applies only to **polymorphic base calsses** -- to base classes designed to allow the manipulation of derived class types through base class interfaces. `TimeKeeper` is a polynorphic base class, becasue we expect to be able to manipulate `AtomicClock` and `WaterClock` objects, even if we have only `TimeKeeper` pinters to them.  
___

* Polymorphic base classes should declare virtual destructors. If a class has any virtual functions, it should have a virtual destructor. 
* Classes not designed to be base classes or not designed to be used polymorphically should not declare virtual destructors. 

___
### **Item 8: Prevent exceptions from leaving destructors.** 

### **Item 9: Never call virtual functions during construction or destruction.** 

### **Item 10: Have assignment operators return a reference to *this.** 

### **Item 11: Handle assignment to self in operator=.**

### **Item 12: Copy all parts of an object.**