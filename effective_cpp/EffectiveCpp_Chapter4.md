# Chapter 4: Designs and Declarations

## Item 18: Make interfaces easy to use correctly and hard to use incorrectly.

___
**Things to Remember**
* Good interfaces are easy to use correctly and hard to use incorrectly. You should strive for these characteristics in all your interfaces.
* Ways to facilitate correct use include consistency in interfaces and behavioral compatibility with built-in types.
* Ways to prevent errors include creating new types, restricting operations on types, constraining object values, and eliminating client resource management responsibilities.
* tr1::shared_ptr supports custom deleters. This prevents the cross- DLL problem, can be used to automatically unlock mutexes (see Item 14), etc.

## Item 19: Treat class design as type design

___
**Things to Remember**
* Class design is type design. Before defining a new type, be sure to
consider all the issues discussed in this Item.

## Item 20: Perfer pass-by-reference-to-*const* to pass-by-value.

___
**Things to Remember**
* Prefer pass-by-reference-to-const over pass-by-value. It’s typically
more efficient and it avoids the slicing problem.
* The rule doesn’t apply to built-in types and STL iterator and function
object types. For them, pass-by-value is usually appropriate.

## Item 21: Don’t try to return a reference when you must return an object.

___
**Things to Remember**
* Never return a pointer or reference to a local stack object, a reference to a heap-allocated object, or a pointer or reference to a local static object if there is a chance that more than one such object will be needed. (Item 4 provides an example of a design where returning a reference to a local static is reasonable, at least in single-threaded

## Item 22: Declare data members private.

___
**Things to Remember**
* Declare data members private. It gives clients syntactically uniform access to data, affords fine-grained access control, allows invariants to be enforced, and offers class authors implementation flexibility.
* protected is no more encapsulated than public.

## Item 23: Prefer non-member non-friend functions to member functions.

___
**Things to Remember**
* Prefer non-member non-friend functions to member functions. Doing so increases encapsulation, packaging flexibility, and functional extensibility.

## Item 24: Declare non-member functions when type conversions should apply to all parameters.

___
**Things to Remember**
* If you need type conversions on all parameters to a function (including the one that would otherwise be pointed to by the this pointer), the function must be a non-member.

## Item 25: Consider support for a non-throwing swap.

___
**Things to Remember**
* Provide a swap member function when std::swap would be inefficient for your type. Make sure your swap doesn’t throw exceptions.
* If you offer a member swap, also offer a non-member swap that calls the member. For classes (not templates), specialize std::swap, too.
* When calling swap, employ a using declaration for std::swap, then call swap without namespace qualification.
* It’s fine to totally specialize std templates for user-defined types, but never try to add something completely new to std.
