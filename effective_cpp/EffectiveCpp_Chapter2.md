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

### **Item 7: Declare destructors virtual in polymorphic base classes.** 

### **Item 8: Prevent exceptions from leaving destructors.** 

### **Item 9: Never call virtual functions during construction or destruction.** 

### **Item 10: Have assignment operators return a reference to *this.** 

### **Item 11: Handle assignment to self in operator=.**

### **Item 12: Copy all parts of an object.**