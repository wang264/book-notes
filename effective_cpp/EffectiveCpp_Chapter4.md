# **Chapter 4: Designs and Declarations**


<br/>
<br/>



## **Item 18: Make interfaces easy to use correctly and hard to use incorrectly.**
Developing interfaces that are easy to use correctly and hard to useincorrectly requires that you consider the kinds of mistakes that clients might make. 

For example, suppose you’re designing the constructor for a class representing dates in time:
```cpp
class Date {
  public:
    Date(int month, int day, int year);
    ...
};
```
Seem reasonable, but there are at least two errors that clients might easily make.
* First, they might pass parameters in the wrong order:
  ```cpp
  Date d(30, 3, 1995); // Oops! Should be “3, 30” , not “30, 3”
  ```
* Second, they might pass an invalid month or day number:
  ```cpp
  Date d(3, 40, 1995); // Oops! Should be “3, 30” , not “3, 40”
  ```

A way to prevent client errors by the introduction of new types.
```cpp
struct Day{
  explicit Day(int d): val(d){}
  int val;
};
struct Month{
  explicit Month(int m): val(m){}
  int val;
};
struct Year{
  explicit Year(int y): val(y){}
  int val;
};
class Date {
  public:
    Date(const Month& m, const Day& d, const Year& y);
  ...
};

Date d(30, 3, 1995); // error! wrong types
Date d(Day(30), Month(3), Year(1995)); // error! wrong types
Date d(Month(3), Day(30), Year(1995)); // okay, types are correct
```
Once the right types are in place, it can sometimes be reasonable to restrict the values of those types.

For example, month（only 12 valid value). We could use enum, but enums are not
as type-safe since enums can be used like ints(see Item 2). A safer solution is to predefine the set of all valid Months:
```cpp
class Month {
  public:
    //// functions returning all valid Month values,
    static Month Jan() { return Month(1); } 
    static Month Feb() { return Month(2); } 
    ... 
    static Month Dec() { return Month(12); } 
    ... // other member functions
  private:
    explicit Month(int m); // prevent creation of new Month values
  ... // month-specific data
};
Date d(Month::Mar(), Day(30), Year(1995));
```
___

Any interface that requires that clients remember to do something is prone to incorrect use, because clients can forget to do it. 

For example, Item 13 introduces a factory function that returns pointers to dynamically allocated objects in an Investment hierarchy:
```cpp
Investment* createInvestment(); // from Item13; parameters omitted for simplicity
```
Client could easily forget to delete a pointer, delete the same pointer more than once.
A better interface decision would be to preempt the problem by having the factory function
return a smart pointer in the first place:
```cpp
std::tr1::shared_ptr<Investment> createInvestment();
```
This essentially forces clients to store the return value in a `tr1::shared_ptr`, all but eliminating the possibility of forgetting to delete the underlying Investment object. 

Suppose an `Investment*` pointer from `createInvestment()` are expected to pass that pointer to a function called `getRidOfInvestment()` instead of using *`delete`* on it. The implementer of `createInvestment` can forestall client to use the wrong deleter by returning a tr1::shared_ptr with getRidOfInvestment bound to it as its deleter.

`tr1::shared_ptr` offers a constructor taking two arguments: the pointer to be managed and the deleter. 

```cpp
std::tr1::shared_ptr<Investment> createInvestment()
{ 
  // create a null shared_ptr with getRidOfInvestment as its deleter; see Item27 for info on static_cast
  std::tr1::shared_ptr<Investment> retVal(static_cast<Investment*>(0), getRidOfInvestment;
  ... // make retVal point to the correct object
  return retVal;
}
```

___

**Things to Remember**
* Good interfaces are easy to use correctly and hard to use incorrectly. You should strive for these characteristics in all your interfaces.
* Ways to facilitate correct use include consistency in interfaces and behavioral compatibility with built-in types.
* Ways to prevent errors include creating new types, restricting operations on types, constraining object values, and eliminating client resource management responsibilities.
* tr1::shared_ptr supports custom deleters. This prevents the cross- DLL problem, can be used to automatically unlock mutexes (see Item 14), etc.


<br/>
<br/>



## **Item 19: Treat class design as type design**
In C++, defining a new class defines a new type. So, how do you design effective classes?

* How should objects of your new type be creasted and destroyed? 
  * constructor, destructor, `new` and `delete`
* How should object initialization differ from object assignment? 
  * difference between your constructors and assignment operators.
* What does it mean for objects of your new type to be passed by value? 
  * copy constructor
* What are the restrictions on legal value of your new type?
* Does your new type fit into an inheritance graph?
  * `virtual` keyword. 
* What kind of type conversions are allowed for your new type?
  * Item 15, implicit and explicit conversion functions. 
* What operators and functions make sense for the new type?
* What standard functions should be disallowed?
  * Item 6, declare private. 
* Who should have access to the members of your new type?
  * public, protected, and private member.
* What is the “undeclared interface” of your new type?
* How general is your new type?
* Is a new type really what you need?
___
**Things to Remember**
* Class design is type design. Before defining a new type, be sure to
consider all the issues discussed in this Item.


<br/>
<br/>



## **Item 20: Perfer pass-by-reference-to-*const* to pass-by-value.**
By default, C++ passes objects to and from functions by value( it inherits from C)

The cost would be one call to the copy constructor and one call to the destructor. 
Consider pass by reference-to-const:
```cpp
bool validateStudent(const Student& s);
```
___
Also, this could prevent the slicing problem.

For example, suppose you’re working on a set of classes for implementing a graphical window system:
```cpp
class Window {
  public:
    ...
    std::string name() const; // return name of window
    virtual void display() const; // dynamic binding, draw window and contents
};

class WindowWithScrollBars: public Window {
    public:
      ...
      virtual void display() const;
};
```
If you want a function to print out a window’s name and then display the window. Here’s the wrong way:
```cpp
void printNameAndDisplay(Window w) // incorrect! parameter may be sliced!
{ 
  std::cout << w.name();
  w.display();
}
WindowWithScrollBars wwsb;
printNameAndDisplay(wwsb);
```
Inside `printNameAndDisplay`, w will always act like an object of class Window, regardless of the type of object passed to the function, and will always call `Window::display`.

The way around the slicing problem is to pass w by reference-to-const:
```cpp
void printNameAndDisplay(const Window& w) // fine, parameter won’t be sliced
{ 
  std::cout << w.name();
  w.display();
}
```
Now w will act like whatever kind of window is actually passed in.

___
**Things to Remember**
* Prefer pass-by-reference-to-const over pass-by-value. It’s typically
more efficient and it avoids the slicing problem.
* The rule doesn’t apply to built-in types and STL iterator and function
object types. For them, pass-by-value is usually appropriate.


<br/>
<br/>



## **Item 21: Don’t try to return a reference when you must return an object.**

Consider a class for representing rational numbers, including a function
for multiplying two rationals together:
```cpp
class Rational {
  public:
    Rational(int numerator = 0, // see Item 24 for why this
    int denominator = 1); // ctor isn’t declared explicit
    ...
  private:
    int n, d; // numerator and denominator
    friend const Rational operator*(const Rational& lhs, const Rational& rhs);
};
//clinet code
Rational a(1, 2); // a = 1/2
Rational b(3, 5); // b = 3/5
Rational c = a * b; // c should be 3/10
```
This version of operator* is returning its result object by value and you want to save the cost of construction call. 

You can return a reference of `Rational`, but refernce is alias, it is still and object. The object still need to be created, on the stack or on the heap. 
**Method 1**
```cpp
const Rational& operator*(const Rational& lhs, // warning! bad code!
const Rational& rhs)
{
  Rational result(lhs.n * rhs.n, lhs.d * rhs.d);
  return result;
}
```
Return a reference to someting on the stack, this is bad and will leak memory. 

<br/>

**Method 2**
```cpp
const Rational& operator*(const Rational& lhs, // warning! more bad
const Rational& rhs) // code!
{
  Rational *result = new Rational(lhs.n * rhs.n, lhs.d * rhs.d);
  return *result;
}

Rational w, x, y, z;
w = x * y * z; // same as operator*(operator*(x, y), z)
```
The problem, who will apply delete to the object conjured up by your use of new? Will have resource leak.

<br/>

**Method 3**
an implementation based on operator* returning a reference to a static Rational object, one defined inside the function:
```cpp
const Rational& operator*(const Rational& lhs, // warning! yet more
const Rational& rhs) // bad code!
{
  //// static object to which a reference will be returned
  static Rational result; 
  result = ... ; // multiply lhs by rhs and put then product inside result
  return result;
}
```
Consider this client code
```cpp
bool operator==(const Rational& lhs, // an operator==
const Rational& rhs); // for Rationals
Rational a, b, c, d;
...
if ((a * b) == (c * d)) {
  ... // do whatever’s appropriate when the products are equal;
} else {
  ... // do whatever’s appropriate when they’re not;
}
```
The expression ((a*b) == (c*d)) will always evaluate to true. if we rewrite the statement like this `if (operator==(operator*(a, b), operator*(c, d)))`, `operator*(c, d)` and `operator*(a, b)` will return the reference to the same static Rational object. 

**The right way to write a function that must return a new object is to have that function return a new object.** For Rational’s operator*, that means either the following code or something essentially equivalent:
```cpp
inline const Rational operator*(const Rational& lhs, const Rational& rhs)
{
  return Rational(lhs.n * rhs.n, lhs.d * rhs.d);
}
```

___
**Things to Remember**
* Never return a pointer or reference to a local stack object, a reference to a heap-allocated object, or a pointer or reference to a local static object if there is a chance that more than one such object will be needed. (Item 4 provides an example of a design where returning a reference to a local static is reasonable, at least in single-threaded

<br/>
<br/>

## **Item 22: Declare data members private.**

___
**Things to Remember**
* Declare data members private. It gives clients syntactically uniform access to data, affords fine-grained access control, allows invariants to be enforced, and offers class authors implementation flexibility.
* protected is no more encapsulated than public.


<br/>
<br/>


## **Item 23: Prefer non-member non-friend functions to member functions.**

___
**Things to Remember**
* Prefer non-member non-friend functions to member functions. Doing so increases encapsulation, packaging flexibility, and functional extensibility.


<br/>
<br/>



## **Item 24: Declare non-member functions when type conversions should apply to all parameters.**

___
**Things to Remember**
* If you need type conversions on all parameters to a function (including the one that would otherwise be pointed to by the this pointer), the function must be a non-member.


<br/>
<br/>



## **Item 25: Consider support for a non-throwing swap.**

___
**Things to Remember**
* Provide a swap member function when std::swap would be inefficient for your type. Make sure your swap doesn’t throw exceptions.
* If you offer a member swap, also offer a non-member swap that calls the member. For classes (not templates), specialize std::swap, too.
* When calling swap, employ a using declaration for std::swap, then call swap without namespace qualification.
* It’s fine to totally specialize std templates for user-defined types, but never try to add something completely new to std.
