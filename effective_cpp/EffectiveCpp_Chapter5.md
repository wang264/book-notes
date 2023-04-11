# **Chapter 5: Implementations**

<br/>
<br/>


## **Item 26: Postpone variable definitions as long as possible.**

There is a cost associated with unused variables, so you want to avoid them whenever you can.

Consider the following function, which returns an encrypted version of a password, provided the password is long enough. If the password is too short, the function throws. 

```cpp
// this function defines the variable "encrypted" too soon
std::string encryptPassword(const std::string& password)
{
  using namespace std;
  string encrypted;
  if (password.length() < MinimumPasswordLength) {
    throw logic_error("Password is too short");
  }
  ... // do whatever is necessary to place an encrypted version of password in encrypted
  return encrypted;
}
```
The above function define `encrypted` too soon. The variable is unused if an exception is thrown. We can change to below

```cpp
// this function postpones encrypted’s definition until it’s truly necessary
std::string encryptPassword(const std::string& password)
{
  using namespace std;
  if (password.length() < MinimumPasswordLength) {
    throw logic_error("Password is too short");
  }
  string encrypted;
  ... // do whatever is necessary to place an encrypted version of password in encrypted
  return encrypted;
}
```
This code still isn’t as tight as it might be, because encrypted is defined without any initialization arguments. That means its default constructor will be used. We can do the following. 

For example, suppose the hard part of `encryptPassword` is performed in this function: `void encrypt(std::string& s);`
Then we can do the following:
```cpp
// this function postpones encrypted’s definition until it’s necessary, but it’s still needlessly inefficient
std::string encryptPassword(const std::string& password)
{
  ... // import std and check length as above
  string encrypted; // default-construct encrypted assign to encrypted
  encrypted = password; 
  encrypt(encrypted);
  return encrypted;
}
```
A better way is to do the following. Finally! 
```cpp
// finally, the best way to define and initialize encrypted
std::string encryptPassword(const std::string& password)
{
  ... // import std and check length
  string encrypted(password); // define and initialize via copy constructor
  encrypt(encrypted);
  return encrypted;
}
```
This suggests the real meaning of “as long as possible” in this Item’s title. 
* Not only should you postpone a variable’s definition until right before you have to use the variable, you should also try to postpone
the definition until you have initialization arguments for it. 
* By doing so, you avoid constructing and destructing unneeded objects, and you avoid unnecessary default constructions.

___

**What about loops?**
Approach A
```cpp
// Approach A: define outside loop 
Widget w;
for (int i = 0; i < n; ++i){
  w = some value dependent on i;
  ...
}
```

Approach B 
```cpp
// Approach B: define inside loop 
for (int i = 0; i < n; ++i){
  Widget w(some value dependent on i);
  ...
}
```
* Approach A: 1 constructor + 1 destructor + n assignments.
* Approach B: n constructors + n destructors.


For classes where an assignment costs less than a constructor-destructor pair, Approach A is generally more efficient. This is especially the case as n gets large. Otherwise, Approach B is probably better.

Furthermore, Approach A makes the name w visible in a larger scope (the one containing the loop) than Approach B, something that’s contrary to program comprehensibility and maintainability. As a result, unless you know that (1) assignment is less expensive than a constructor-destructor pair and (2) you’re dealing with a performance sensitive part of your code, you should default to using Approach B.

___

**Things to Remember**
* Postpone variable definitions as long as possible. It increases program clarity and improves program efficiency.
<br/>
<br/>

## **Item 27: Minimize casting.**

___

**Things to Remember**
* Avoid casts whenever practical, especially dynamic_casts in performance-sensitive code. If a design requires casting, try to develop a cast-free alternative.
* When casting is necessary, try to hide it inside a function. Clients can then call the function instead of putting casts in their own code.
* Prefer C++-style casts to old-style casts. They are easier to see, and they are more specific about what they do.

<br/>
<br/>

## **Item 28: Avoid returning “handles” to object internals.**

___

**Things to Remember**
* Avoid returning handles (references, pointers, or iterators) to object internals. Not returning handles increases encapsulation, helps const member functions act const, and minimizes the creation of dangling handles.
<br/>
<br/>

## **Item 29: Strive for exception-safe code.**

___

**Things to Remember**
* Exception-safe functions leak no resources and allow no data structures to become corrupted, even when exceptions are thrown. Such functions offer the basic, strong, or nothrow guarantees.
* The strong guarantee can often be implemented via copy-and-swap, but the strong guarantee is not practical for all functions.
* A function can usually offer a guarantee no stronger than the weakest guarantee of the functions it calls.
<br/>
<br/>

## **Item 30: Understand the ins and outs of inlining.**

___

**Things to Remember**
* Limit most inlining to small, frequently called functions. This facilitates debugging and binary upgradability, minimizes potential code bloat, and maximizes the chances of greater program speed.
* Don’t declare function templates inline just because they appear in header files.
<br/>
<br/>

## **Item 31: Minimize compilation dependencies between files.** 

___

**Things to Remember**
* The general idea behind minimizing compilation dependencies is to depend on declarations instead of definitions. Two approaches based on this idea are Handle classes and Interface classes.
* Library header files should exist in full and declaration-only forms. This applies regardless of whether templates are involved.
<br/>
<br/>
