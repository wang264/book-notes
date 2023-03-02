# Chapter 3: Resourse Management

## Item 13: Use objects to manage resources

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