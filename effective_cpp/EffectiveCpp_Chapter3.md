# Chapter 3: Resourse Management


<br/>
<br/>



## **Item 13: Use objects to manage resources**
Suppose we are working with a library for modeling investments(`Investment` Class) and the we use a factory function(Item 7) to get a specific `Investment` objects. 
```cpp
// return ptr to dynamically allocated object in the Investment hierarchy;
// the caller must delete it (parameters omitted for simplicity)
Investment* createInvestment(); 

//we use it like this
void f()
{
  Investment *pInv = createInvestment(); // call factory function use pInv
  ... 
  delete pInv; // release object
}
```
If somewhere throw an exception in the ..., things will not get deleted.
**What to do?** Put resources inside objects so we can rely on C++'s automatic destructor invoation to make sure that the resources are released. 

Better way is to use **`auto_ptr`**(in C++ 11, we use the `unique_ptr`), a smart pointer whose destructor automatically calls delete on what it points to. 

```cpp
void f()
{
  std::auto_ptr<Investment> pInv(createInvestment()); // call factory function
  ... // use pInv as before
} // automatically delete pInv via auto_ptr’s dtor
```

Two critical aspects of using objects to manage resources:
* Resources are acquired and immediately turned over to resource-
managing objects. (The idea of using objects to manage resources is often called **Resource Acquisition Is Initialization(RAII)**)
* Resource-managing objects use their destructors to ensure
that resources are released.

___
`auto_ptr` automatically deletes what it points to when the
auto_ptr is destroyed so if more than one `auto_ptr` point to the same object, the object would be deleted more than once.

To prevent such problems, auto_ptrs have an unusual characteristic: 
1. copying them (via copy constructor or copy assignment operator) 
2. sets them to null. 
3. and the copying pointer assumes sole ownership of the resource!

```cpp
// pInv1 points to the object returned from createInvestment
std::auto_ptr<Investment> pInv1(createInvestment());
// pInv2 now points to the object; pInv1 is now null
std::auto_ptr<Investment> pInv2(pInv1); 
pInv1 = pInv2; // now pInv1 points to the object, and pInv2 is null
```

An alternative to auto_ptr is a **reference-counting smart pointer (RCSP)**`tr1::shared_ptr` (see Item 54) is an RCSP, so you could write f this
way:
```cpp
void f()
{
  ...
  std::tr1::shared_ptr<Investment> pInv(createInvestment()); // call factory function
  ... // use pInv as before
} // automatically delete pInv via shared_ptr’s dtor

void f()
{
  ... // pInv1 points to the object returned from createInvestment
  std::tr1::shared_ptr<Investment> pInv1(createInvestment()); 
  // both pInv1 and pInv2 now point to the object
  std::tr1::shared_ptr<Investment> pInv2(pInv1); 
  pInv1 = pInv2; // ditto — nothing has changed
  ...
} // pInv1 and pInv2 aredestroyed, and the object they point to is automatically deleted
```
This Item isn’t about auto_ptr, tr1::shared_ptr, or any other kind of smart pointer. It’s about the importance of using objects to manage resources.

___
**Things to Remember**
* To prevent resource leaks, use RAII objecsts that acquire resources in their constructors and release them in their destructors. 
* Two commonly useful RAII classes are `tr1::shared_ptr` and `auto_ptr`. `tr1::shared_ptr` is usually the better choice, because its behavior when copied is intuitve. Copying an `auto_ptr` sets it to null. 


<br/>
<br/>



## **Item 14: Think carefully about copying behavior in resource-managing classes.**
suppose you’re using a C API to manipulate mutex objects of type Mutex offering functions lock and unlock, a RAII class could be constructed as:
```cpp
void lock(Mutex *pm); // lock mutex pointed to by pm
void unlock(Mutex *pm); // unlock the mutex

class Lock {
  public:
    explicit Lock(Mutex *pm): mutexPtr(pm)
    { lock(mutexPtr); } // acquire resource
    ~Lock() { unlock(mutexPtr); } // release resource
  private:
    Mutex *mutexPtr;
};
```
Clients use `Lock` in the conventional RAII fashion. However, question arise, what should happen if a Lock object is copied?
```cpp
Mutex m; // define the mutex you need to use
{ // create block to define critical section
  Lock ml(&m); // lock the mutex
  ... // perform critical section operations
} // automatically unlock mutex at end of block

Lock ml1(&m); // lock m
Lock ml2(ml1); // copy ml1 to ml2 — what should happen here?
```
You usually have the following options for an RAII object is copied. 
* Prohibit copying
  * It make no sense to allow RAII objects to be copied. (likely true for `Lock`)
  * You can declare the copying operations private.(Item 6)
* Reference-count the underlying resource
  * This is the case when we hold on to a resource untill the last object using it has been destroyed.
  * use the `tr1::shared_ptr`. Howeever, its default behavior is to delete what it points to when the reference count goes to zero, and that’s not what we want. When we’re done with a Mutex, we want to unlock it, not delete it.
  * Fortunately, tr1::shared_ptr allows specification of a **“deleter”** — a function or function object to be called when the reference count goes to zero. It is an optional second parameter to its constructor
  * ```cpp
    class Lock {
      public:
        // init shared_ptr with the Mutex to point to and the unlock func as the deleter†
        explicit Lock(Mutex *pm) : mutexPtr(pm, unlock) 
        { 
          lock(mutexPtr.get()); // see Item15 for info on “get”
        }
      private:
        std::tr1::shared_ptr<Mutex> mutexPtr; // use shared_ptr
    }; // instead of raw pointer
      ```
* Copy the underlying resource
  * perform deep copy.
* Transfer owenership of the underlying resource.
  * can use `auto_ptr`

___
**Things to Remember**
* Copying an RAII object entails copying the resource it manages, so the copying behavior of the resource determines the copying behavior of the RAII object.
* Common RAII class copying behaviors are disallowing copying and performing reference counting, but other behaviors are possible.


<br/>
<br/>



## **Item 15: Provide access to raw resources in resource-managing classes.**

In some situation, you need a way to convert an object of the RAII class into the raw resource it contains. There are two general ways to do it: **explicit conversion
and implicit conversion.**
___
**Explicit Conversion:** the `get()` function which return the resouce. 
The the example from Item 13, use smart pointer to hold the result of a call to factory function like createInvestment.
**Implicit Conversion:** user-defined conversion function.
```cpp
std::tr1::shared_ptr<Investment> pInv(createInvestment());
```
Suppose that a function you’d like to use when working with Investment objects is this:
```cpp
int daysHeld(const Investment *pi); // return number of days investment has been held
int days = daysHeld(pInv.get()); // fine, passes the raw pointer in pInv to daysHeld
```
tr1::shared_ptr and auto_ptr both offer a `get()` member function to perform an **explicit conversion**, i.e., to return (a copy of) the raw pointer inside the smart pointer object

Also, like virtually all smart pointer classes, tr1::shared_ptr and auto_ptr also **overload the pointer dereferencing operators (operator-> and operator*)**, and this allows **implicit conversion** to the underlying raw pointers

Let's say you want to support the implicit conversion from Font to FontHandle, you can do the following.
```cpp
class Font {
  public:
    ...
    operator FontHandle() const // implicit conversion function
    { return f; }
    ...
};
```
___
**Things to Remember**
* APIs often require access to raw resources, so each RAII class should offer a way to get at the resource it manages.
* Access may be via explicit conversion or implicit conversion. In general, explicit conversion is safer, but implicit conversion is more convenient for clients.


<br/>
<br/>



## **Item 16: Use the same form in corresponding uses of *new* and _delete_**
The rule is simple: if you use [] in a new expression, you must use [] in the corresponding delete expression. If you don’t use [] in a new expression, don’t use [] in the matching delete expression.

When you use delete on a pointer, the only way for delete to know whether the array size information is there is for you to tell it. If you use brackets in your use of delete, delete assumes an array is pointed to. Otherwise, it assumes that a single object is pointed to:
```cpp
std::string *stringPtr1 = new std::string;
std::string *stringPtr2 = new std::string[100];
...
delete stringPtr1; // delete an object
delete [] stringPtr2; // delete an array of objects
```
___
One thing that worth pointed out is when you use `typedef`, For example:
```cpp
// a person’s address has 4 lines, each of which is a string
typedef std::string AddressLines[4]; 
// Because AddressLines is an array, this use of new, returns a string*, just like “new string[4]” would
std::string *pal = new AddressLines; 

// must be matched with the array form of delete:
delete pal; // undefined!
delete [] pal; // fine
```
To avoid such confusion, abstain from typedefs for array types. That’s easy, because the standard C++ library (see Item 54) includes string and vector.
___
**Things to Remember**
* If you use [] in a new expression, you must use [] in the corresponding delete expression. If you don’t use [] in a new expression, you mustn’t use [] in the corresponding delete expression.



<br/>
<br/>



## **Item 17: Store *new*ed objects in smart pointers in standalone statements.**
Suppose we have a function to reveal our processing priority and a second function to do some processing on a dynamically allocated Widget in accord with a priority. The following code would compile but might leak resource. 
```cpp
int priority();
void processWidget(std::tr1::shared_ptr<Widget> pw, int priority);

processWidget(std::tr1::shared_ptr<Widget>(new Widget), priority());
```
**why?**
Before processWidget can be called, then, compilers must generate code to do these three things:
* Call priority.
* Execute “new Widget”.
* Call the tr1::shared_ptr constructor.

C++ compilers are granted considerable latitude in determining the order in which these things are to be done.

If the compiler decide to use this order, 
1. Execute “new Widget”.
2. Call priority.
3. Call the tr1::shared_ptr constructor.
If the call to priority yields an exception, In that case, the pointer returned from “new Widget” will be lost. Memory leak.

The way to avoid problems like this is simple: use a separate statement to create the Widget and store it in a smart pointer.
```cpp
std::tr1::shared_ptr<Widget> pw(new Widget); 
processWidget(pw, priority()); // this call won’t leak
```
___
**Things to Remember**
* Store newed objects in smart pointers in standalone statements. Failure to do this can lead to subtle resource leaks when exceptions are thrown.