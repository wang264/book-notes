# Chapter 1: Accustoming Yourself to C++

## **Item 1: View C++ as a federation of language**
<br/>
<br/>

## **Item 2: Prefer *consts*, *enums*, and *inlines* to *#defines***
<br/>

### **For simple constants, prefer const objects or enums to `#define`**

This item might better be called "prefer the compiler to the preprocessor" since symbol defined by `#define` might not been seen by compiler.

for example if you do

```cpp
#define ASPECT_RATIO 1.653
```
* the symbolic name `ASPECT_RATIO` may never be seen by compiler; it may be removed by the preproceesor before the source code evert get to a compiler. 
* if you get an error about this name, you will see 1.653 instead of `ASPECT_RATIO`

The solution is to replace the marco with a constant.

```cpp
const double ASPECT_RATIO = 1.653;
``` 
* As a language constant, AspectRatio is definately seen by compiler and will entered into their symbol tables.
* Use of the constant may yield smaller code than using a `#define` since the preprocessor's blind substitution of the marco name `ASPECT_RATIO` will result in multiple copied of 1.653 in your code. 

### Two special cases
* Defining constant pointers:
    * _pointer_ should be declared *const*, usually in addition to what the pointer points to. To define a constant char*-based string in a header file, you will do:
    * ```const char * const authorName = "Scott Meyers"; ``` 
    * or you could also do `const std::string authorName("Scott Meyers");`
* Class-specific constants. 
    * To limit the scope of a constant to a class you need to make it a member. And to ensure there's is at most one copy of the constant, you must make it a *static* member. 
    * they way you set up a class constant 
    ```cpp
    // This goes in the class header file.
    class EngineeringConstants {
        private:
        static const double FUDGE_FACTOR;
    };
    // This goes in the class implementation file
    #include "EngineeringConstants.h"
    const double EngineeringConstants::FUDGE_FACTOR = 1.35;
    ```

The enum hack 
```cpp
class GamePlayer {
private:
  enum { NUM_TURNS = 5 };    // "the enum hack" makes
                             // NUM_TURNS a symbolic name
                             // for 5
 
  int scores[NUM_TURNS];     // fine
};
```
### **For function-like macros, prefer inline functions to `#defines`**
Common (mis)use of the `#define` derective is using it to implement macros that look like functions but that don't incur hte overhead of a function call. 
```cpp
\\ call f with the maximum of a and b
#define CALL_WIWTH_MAX(a,b) f((a) > (b) ? (a) : (b))
```
you should use inline template function instead
```cpp
// But a template fixes that problem quite nicely:
template<typename T>
inline void callWithMax(const T& a, const T& b)  // see Item20 for pass by reference-to-const
{ 
    f(a > b ? a : b);  
}
```
</br>

## **Item 3: Use const whenever possible**

## **Item 4: Make sure that objects are initialized before they're used**

