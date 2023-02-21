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

CALL_WIWTH_MAX(++a, b);     // a is incremented twice
// since it expand into ((++a) > (b) ? (++a) : (b)) so 'a' is incremented twice when ++a < b

CALL_WIWTH_MAX(++a, b+10);  // a is incremented once
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
<br/>

```cpp
char greeting[] = "Helle";
char *p = greeting;         //non-const pointer, //non-const data
const char *p = greeting;   //non-const pointer, //const data
char *const p = greeting;   //const pointer, //non-const data
const char *const p = greeting; //const pointer, const data
```
1. If the word *const* apppers to the left of the asterisk, what it pointed to is constant.
2. If the word const appears to the right of the asterisk, the pointer itself is constant. 
3. If *const* appears on both sides, both are constant. 

<br/>

```cpp
void f1(const Widget *pw) // f1&f2 takes a pinter to a cosnt Widget object
void f2(Widget const *pw)
```
When what's pointed to is constant, list *const* before and after the type means the same. 
<br/>

```cpp
std::vector<int> vec;
...
// iter act like a T* const, constant pointer. 
const std::vector<int>::iterator iter = vec.begin();
*iter =10;      //OK. change what iter points to. 
++iter;         //error! iter is const

// clter act like a const T*
std::vector<int>::const_iterator clter = vec.begin();
*clter=10;      //error! *clter is const
++clter;        //fine, change clter
```
STL terators are models on pointers, so *iterator* acts like a `T* pointer`. 
* Declaring a iterator const, iterator is not allowed to pointer to something else, but the things it points to may be modifited. 
* By using `const_iterator`, the iterator will point to someting that can not be modified. 

```cpp
class TextBlock {
    public:
    ...
    // operator[] for const objects
    const char& operator[](std::size_t position) const 
    {return text[position];}
    
    // operator[] for non-const objects
    const char& operator[](std::size_t position)    
    {return text[position];}
    private:
        std::string text;
}
// TextBlock's operator[]s can be used like this
TextBlock tb("Hello")
std::cout<<tb[0];       //call non-const operator[]
const TextBlock ctb("World")
std::cout<<ctb[0];      // call const operator[]
```
usage of const and non-const member function

```cpp
void print(const TextBlock& ctb)
{
    std::cout<<ctb[0];  //calls const TextBlock::operator[]
}
```
This is more of a realistic usage of const objects in real programs, as a result of being passed by pointer or reference to const.

<br/>
what does it mean for a member function to be const? 

1. Bitwise constness
    * A member function is const if and only if it doesn't modify any of the object's data members.
2. Logical constness
    * A member function might modify some of the bits in the object on which it's invoked, but only in ways that clients cannot detect.

```cpp
class CTextBlock{
    public:
    ...
        char& operator[](std::size_t position) const 
        {return pText[position];}
    private:
        char *pText;
//but you can still modify the object using
const CTextBlock cctb("Hello");     //declare constant object
char *pc = &cctb[0];                //call the const operator[] to get a pointer to the cctb's data
*pc = 'J';                          // cctb noew has the value "Jello"
};
```
member function that does not act very const could pass bitwise const test. 

```cpp
class CTextBlock{
    public:
        std::size_t length() const;
    private::
        char *pText;
        // HAVE to use the keyword mutable
        mutable std:: size_t textLength; //last calculated length of textblock
        mutable bool lengthIsValid;     //whether length is currently valid

        std::size_t CTextBlock::length() const
        {
            if (!lengthIsValid)
            {   // the data memeber not can be modified, even in a const member function.
                textLength = std::strlen(pText);
                lengthIsValid = true;
            }
            return textLength;
        }
};
```
The function `length()` is logical const, but not bitwise const. 
*mutable* frees non-static data members from the constrains of bitwise constness. 
<br/>

```cpp
// code with duplication
class CTextBlock{
    public:
    ...
        char& operator[](std::size_t position) const {
            ...
            return pText[position];
        }
        char& operator[](std::size_t position) {
            ...
            // const_cast cast away const in op[]'s return type
            return const_cast<char&>(
                // static_cast add const to *this's type, so it will call const version of op[]
                static_cast<const TextBlock&>(*this)[position]
                );
        }
    ...
};
```

```cpp
// code without duplication
class CTextBlock{
    public:
    ...
        char& operator[](std::size_t position) const {
            ...
            return pText[position];}
    
        char& operator[](std::size_t position) {
            ...
            return pText[position];}
    ...
};
```

Having the non-const member function call the const version is a safe way to avioid code duplication, even though it requires a cast. However, if you were to call a non-const function from a const one, you'd runn the risk that the object you' promised not to modify would be changed.

<br/>

* Declaring something *const* helps compilers detect usage errors, *const* can be applied to objects at any scope, to function parameters and return types, and to member functions as a whole. 
* Compiler enforce bitwise constness, but you should program using logical constness.
* When *const* and non-*const* member functiosn have essentially identical implementations, code duplication can be avoided by having the non-*const* version call the *const* version. 

<br/>

## **Item 4: Make sure that objects are initialized before they're used**

