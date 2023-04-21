# Chapter 2: Consturctors, Destructors, and Assignment Operators


<br/>
<br/>


## **Item 5: Know what functions C++ silently writes and calls**
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

C++ refuse to compile this code, you must define the copy assignment operator yourself if your class members contain **_const_** or **reference**.

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


<br/>
<br/>



## **Item 6: Explicitly disallow the use of compiler-generated functions you do not want.** 
If you do not want the compiler-generated copy constructor, and copy assignmetn operator. For example, you want to following code to not compile because you believe every house is different, like your real estate agent said. 
```cpp
class HomeForSale{...};
HomeForSale h1;
homeForSale h2;

HomeForSale h3(h1);     //attempy to copy h1, should not compile!
h1 = h2;                //attempt to copy h2, should not compile!
```

What you can do is declare them explicitly and make them priviate so client code can not call it. 
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
In C++ 11, you can leave it as public but put `HomeForSale(const HomeForSale&) = delete` instead.
```cpp
class Uncopyable {
  // allow construction and destruction of derived objects but prevent copying
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

* By declaring a member function explicitly, you prevent compilers from generating their own version. 
* By making the function *private*, you keep people from calling it. 
* Also, need to declaring member functions *private* and NOT implementing them. (So other member functions and friend functiosn can not call this private function)
* To disallowe functionality, automatically provided by compilers, declare the corresponding member functions private and give no implementations. Using a base class like *Uncopyable* is one way to do this.
___

<br/>
<br/>



## **Item 7: Declare destructors virtual in polymorphic base classes.** 
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

It is possible to get bitten by the non-virtual destructor problem even in the complete absence of virtual functions.


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
This might cause trouble if anywhere in an application you somehow convert a pointer to SpecialString into a pointer to *string* and you then use delete on string pointer. The same analysis applies to any class lacking a virtual destructor, including all the STL container types (e.g., vector, list, set, tr1::unordered_map (see Item 54), etc.).

**If you’re ever tempted to inherit from a standard container or any other class with a non-virtual destructor, resist the temptation!**
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




<br/>
<br/>



## **Item 8: Prevent exceptions from leaving destructors.** 
C++ does not prohibit destructors from emitting exceptions, but it certainly discourages that practice. With good reason.
```cpp
class Widget {
  public:
    ...
    ~Widget() { ... } // assume this might emit an exception
};
void doSomething()
{
std::vector<Widget> v;
...
} // v is automatically destroyed here
```
If `v` has ten `Widgets` in it, and execption is thrown when trying to delete the first one. Among the destructors call of the other nine Widgets, another call throw exception. Now there are two simultaneously active exceptions, and that’s one too many for C++. Depending on the precise conditions under which such pairs of simultaneously active exceptions arise, program execution either terminates or yields undefined behavior.

**What should we do** if our destructor needs to perform an operation that may fail by throwing an exception?

For example, suppose you’re working with a class for database connections:
```cpp
class DBConnection {
  public:
    ...
    // function to return DBConnection objects; params omitted for simplicity
    static DBConnection create(); 
    void close(); // close connection; throw an exception if closing fails
};
```
For destructor to not throw, we have two options.

**Option 1: Terminate the program**
```cpp
DBConn::~DBConn()
{
  try { db.close(); }
  catch (...) {
    make log entry that the call to close failed;
    std::abort();
  }
}
```
**Option 2: Swallow the exception**
```cpp
DBConn::~DBConn()
{
  try { db.close(); }
  catch (...) {
    make log entry that the call to close failed;
  }
}
```
Neither of these approaches is especially appealing. The **problem** with both is that the program has no way to react to the condition that led to close throwing an exception in the first place. A better strategy is to design DBConn’s interface so that its **clients have an opportunity to react to problems that may arise**.
```cpp
class DBConn {
  public:
    ...
    void close() // new function for client use
    { 
      db.close();
      closed = true;
    }
    ~DBConn()
    {
      if (!closed) {
        try { // close the connection if the client didn’t
          db.close(); 
        }
        catch (...) { // if closing fails, note that and terminate or swallow
          make log entry that call to close failed; 
          ... 
        }
      }
    }
  private:
    DBConnection db;
    bool closed;
};
```
In this example, telling clients to call close themselves doesn’t impose a burden on them; it gives them an opportunity to deal with errors they would otherwise have no chance to react to.
___
**Things to Remember**
* Destructors should never emit exceptions. If functions called in a destructor may throw, the destructor should catch any exceptions, then swallow them or terminate the program.
* If class clients need to be able to react to exceptions thrown during an operation, the class should provide a regular (i.e., non-destructor) function that performs the operation.


<br/>
<br/>




## **Item 9: Never call virtual functions during construction or destruction.** 

Example, you got a class hierarchy for modeling stock transactions, and need the transactions be auditable. 

```cpp
// base class for all transactions
class Transaction { 
  public: 
    Transaction();
    // make type-dependent log entry
    virtual void logTransaction() const = 0; 
    ...
};
// implementation of base class ctor
Transaction::Transaction() { 
  ...
  logTransaction(); // as final action, log this transaction
} 
class BuyTransaction: public Transaction { // derived class
  public: // how to log transactions of this type
    virtual void logTransaction() const; 
    ...
};
class SellTransaction: public Transaction { // derived class
  public:// how to log transactions of this type
    virtual void logTransaction() const; 
    ...
};
```
Consider what happen when this code is executed: `BuyTransaction b;` 
* `BuyTransaction` constructor will be called but after the `Transaction` constructor call. 
    * Base class parts of derived class objects are constructed before derived class parts are. 
* The version of logTransaction that’s called is the one in `Transaction`, not the one in `BuyTransaction` even though the type of object being created is `BuyTransaction`. 
* During base class construction, virtual functions never go down into derived classes. Instead, the object behaves as if it were of the base type. 

The **same reasoning applies during destruction**. Once a derived class
destructor has run, the object’s derived class data members assume
undefined values, so C++ treats them as if they no longer exist. Upon
entry to the base class destructor, the object becomes a base class
object, and all parts of C++ — virtual functions, dynamic_casts, etc., —
treat it that way.

It’s not always so easy to detect calls to virtual functions during construction
or destruction. The below code is conceptually the same as the earlier version but more insidious. Because `logTransaction()` is wraped inside `init()`. 
```cpp
class Transaction {
  public:
    Transaction()
    { init(); } // call to non-virtual...
    virtual void logTransaction() const = 0;
    ...
  private:
    void init(){
        ...
        logTransaction(); // ...that calls a virtual!
    }
};
```
The only way to avoid this problem is to make sure that none of your constructors or destructors call virtual functions on the object being created or destroyed and that all the functions they call obey the same constraint.

One of the way to deal with this is to **turn `logTransaction` into a non-virtual function**yes in `Transaction`, then require that derived class constructors pass the necessary log information to the `Transaction` constructor.
```cpp
class Transaction {
  public:
    explicit Transaction(const std::string& logInfo);
    // now a non-virtual func
    void logTransaction(const std::string& logInfo) const; 
    ...
};
Transaction::Transaction(const std::string& logInfo){
  ... // now a non-virtual call
  logTransaction(logInfo); 
} 

class BuyTransaction: public Transaction {
  public:
    // pass log info to base class constructor
    BuyTransaction( parameters): Transaction(createLogString( parameters )) 
    { ... } 
    ... 
  private:
    static std::string createLogString( parameters );
}; 
```
___
**Things to Remember**
* Don’t call virtual functions during construction or destruction, because such calls will never go to a more derived class than that of the currently executing constructor or destructor.


<br/>
<br/>




## **Item 10: Have assignment operators return a reference to `*this`.** 
Assignments can be chained together:
```cpp
int x, y, z;
x = y = z = 15;     //chain of assignments
```
assignment is right-associative, os the above assignment chain is parsed like this:
x = (y = (z = 15));
The way this is implemented is that assignment returns a reference to its left-hand argument, and that's the covention you should follow when you implement assignment operators for your classes:
```cpp
// return type is a reference to the current class
Widget& operator=(const Widget& rhs) 
{
  ...
  return *this; // return the left-hand object
}
...
};
```
This convention applies to all assignment operators, not just the standard form shown above. Hence:
```cpp
class Widget {
  public:
    ...
    Widget& operator+=(const Widget& rhs) // the convention applies to
    { // +=, -=, *=, etc.
      ...
      return *this;
    }
    Widget& operator=(int rhs) // it applies even if the
    { // operator’s parameter type
      ... // is unconventional
      return *this;
    }
    ...
};
```
Convention means code that doesn’t follow it will compile. However,unless you have a good reason for doing things differently, don’t.
___
**Things to Remember**
* Have assignment operators return a reference to `*this`.


<br/>
<br/>



## **Item 11: Handle assignment to self in operator=.**
An assignment to self occurs when an object is assigned to itself:
```cpp
class Widget { ... };
Widget w;
w = w; // assignment to self
```
Assignment to self isn’t always so recognizable. For example,
```cpp
a[i] = a[j]; // assignment to self if i == j.
*px = *py; // assignment to self if px and py point to the same thing.
```

In general, code that operates on references or pointers to multiple objects of the same type needs to consider that the objects might be the same. 

In fact, **the two objects need not even be declared to be of the same type if they’re from the same hierarchy**, because a base class reference or pointer can refer or point to an object of a derived class type:

```cpp
class Base { ... };
class Derived: public Base { ... };
// rb and *pd might actually be the same object 
void doSomething(const Base& rb, Derived* pd); 
```

Below is unsafe code. Suppose you creaste a class that holds a raw pointer to a dynamically allocated bitmap.

```cpp
class Bitmap { ... };
class Widget {
  ...
  private:
    Bitmap *pb; // ptr to a heap-allocated object
};

Widget& Widget::operator=(const Widget& rhs) // unsafe impl. of operator=
{
  delete pb; // stop using current bitmap
  pb = new Bitmap(*rhs.pb); // start using a copy of rhs’s bitmap
  return *this; // see Item 10
}
```
**Why? becasue `rhs` could be the same as `*this`**, if this is the case, the delete not only destroys the bitmap for the current object, it destroys the bitmap for `rhs` as well. 
The traditional way to prevent this error is to check for assigment to self via an *identity test*
```cpp
Widget& Widget::operator=(const Widget& rhs)
{
  if (this == &rhs) return *this; // identity test: if a self-assignment, do nothing
  delete pb;
  pb = new Bitmap(*rhs.pb);
  return *this;
}
```

However, this implementation is exception-unsafe. Why? If the `new Bitmap` expression yields an exception (either because there is insufficient memory for the allocation or because Bitmap’s copy constructor throws one), the Widget will end up holding a pointer to a deleted Bitmap.

To achieve exception safe, one can modify the code to not to delte `pb` until after we have copied what it points to. However, a beter way by using **copy and swap** technique. 
```cpp
class Widget {
  ...
  void swap(Widget& rhs); // exchange *this’s and rhs’s data;
  ... // see Item29 for details
};
Widget& Widget::operator=(const Widget& rhs)
{
  Widget temp(rhs); // make a copy of rhs’s data
  swap(temp); // swap *this’s data with the copy’s
  return *this;
}
```
___
**Things to Remember**
* Make sure `operator=` is well-behavied when an object is assigned to itself. Techniques include comparing addresses of source and target objects, careful statement orering, and copy-and-swap.
* Make sure that any fuction operating on more than one object behaves correctly if two or more of the objects are the same. 


<br/>
<br/>



## **Item 12: Copy all parts of an object.**
Consider a class representing customers, where the copying functions have been manually written so that calls to them are logged:
```cpp
void logCall(const std::string& funcName); // make a log entry

class Customer {
  public:
    ...
    Customer(const Customer& rhs);
    Customer& operator=(const Customer& rhs);
    ...
  private:
    std::string name;
};

Customer::Customer(const Customer& rhs)
: name(rhs.name) // copy rhs’s data
{
  logCall("Customer copy constructor");
}

Customer& Customer::operator=(const Customer& rhs)
{
  logCall("Customer copy assignment operator");
  name = rhs.name; // copy rhs’s data
  return *this; // see Item 10
}
```
Everything here looks fine, and in fact everything is fine — until another data member `lastTransaction` is added to Customer:

```cpp
class Date { ... }; // for dates in time
class Customer {
  public:
    ... // as before
  private:
    std::string name;
    Date lastTransaction;
};
```
The code will compile but the existing copying functions are performing a partial copy. (they’re copying the customer’s name, but not its lastTransaction)

One of the most insidious ways this issue can arise is through **inheritance**, for example where derived class copying functions did not invoke their corresponding base class function Consider:
```cpp
class PriorityCustomer: public Customer { // a derived class
  public:
    ...
    PriorityCustomer(const PriorityCustomer& rhs);
    PriorityCustomer& operator=(const PriorityCustomer& rhs);
    ...
  private:
    int priority;
};

PriorityCustomer::PriorityCustomer(const PriorityCustomer& rhs)
: priority(rhs.priority)
{
  logCall("PriorityCustomer copy constructor");
}
PriorityCustomer& PriorityCustomer::operator=(const PriorityCustomer& rhs)
{
  logCall("PriorityCustomer copy assignment operator");
  priority = rhs.priority;
  return *this;
}
```
* `PriorityCustomer`’s copying functions will not copy the data members it inherits from `Customer`. 
* Customer part of the PriorityCustomer object will be initialized by the Customer constructor taking no arguments — by the default constructor. (Assuming it has one. If not, the code won’t compile.)

Instead, **derived class copying functions must invoke their corresponding base class functions:**

```cpp
PriorityCustomer::PriorityCustomer(const PriorityCustomer& rhs)
: Customer(rhs), // invoke base class copy ctor
  priority(rhs.priority)
{
  logCall("PriorityCustomer copy constructor");
}
PriorityCustomer& PriorityCustomer::operator=(const PriorityCustomer& rhs)
{
  logCall("PriorityCustomer copy assignment operator");
  Customer::operator=(rhs); // assign base class parts
  priority = rhs.priority;
  return *this;
}
```
___
**Things to Remember**
* Copying functions should be sure to copy all of an object's data members and all of its base class parts.
* Don't try to implement one of the copying functions in terms of the other(copy constuctor, copy assignment operator). Instead, put commmon functionality in a third function that both call. 