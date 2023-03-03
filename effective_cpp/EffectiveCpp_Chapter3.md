# Chapter 3: Resourse Management

## Item 13: Use objects to manage resources
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

**Things to Remember**
* To prevent resource leaks, use RAII objecsts that acquire resources in their constructors and release them in their destructors. 
* Two commonly useful RAII classes are `tr1::shared_ptr` and `auto_ptr`. `tr1::shared_ptr` is usually the better choice, because its behavior when copied is intuitve. Copying an `auto_ptr` sets it to null. 

## Item 14: Think carefully about copying behavior in resource-managing classes.
**Things to Remember**
* Copying an RAII object entails copying the resource it manages, so the copying behavior of the resource determines the copying behavior of the RAII object.
* Common RAII class copying behaviors are disallowing copying and performing reference counting, but other behaviors are possible.

## Item 15: Provide access to raw resources in resource-managing classes.
**Things to Remember**
* APIs often require access to raw resources, so each RAII class should offer a way to get at the resource it manages.
* Access may be via explicit conversion or implicit conversion. In general, explicit conversion is safer, but implicit conversion is more convenient for clients.

## Item 16: Use the same form in corresponding uses of *new* and *delete*
**Things to Remember**
* If you use [] in a new expression, you must use [] in the corresponding delete expression. If you don’t use [] in a new expression, you mustn’t use [] in the corresponding delete expression.


## Item 17: Store *new*ed objects in smart pointers in standalone statements.
**Things to Remember**
* Store newed objects in smart pointers in standalone statements.
Failure to do this can lead to subtle resource leaks when exceptions
are thrown.