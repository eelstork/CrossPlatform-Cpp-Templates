# Cross platform C++ templates #

This document summarises typical errors encountered when porting template heavy VC++ code to Clang. Overall I find that VS is lenient compared to Clang. To a large extent the following applies to porting from VC++ to GCC.

Relevant errors are annotated using three letter codes summarised at the end of this article.

### Use of typename ###

Use typename ahead of qualified names. Do not use typename ahead of typedef'd aliases.

EQN if 'typename' is found ahead of typedef'd names.
UTN if 'typename' is missing ahead of qualified names.

### Use of template ###

Occasionally clang may issue UTK (pending clarification).

### Default template arguments ###

Exclude default template arguments from out of line member definitions (CAD)

### Function templates ###

Do not repeat class templates ahead of inline function definitions (DTS, NTT)

### Redefining template parameters using typedef ###

GCC and Clang do not allow this (DTS).

```C++
template <typename T>
struct Foo{
    typedef T T;     // Declaration shadows template param.
    typedef T Td;    // Correct
};
```
 
### Specialise ahead of template instantiation ###

Within a translation unit, whatever may cause template instantiation may forbid later specialisation, or cause said specialisation to be ignored.

GCC 5.2 will compile and run the following example, printing nothing. Clang will raise ESA.
VS behaviour unknown.

```C++
#include <string>
#include <iostream>

template<typename S>
class Foo{
    public:
    static const S message;
};

std::string greeting = Foo<std::string>::message;  // Causes template instantiation

// Specialisation. 
template <>
const std::string Foo<std::string>::message = "Hello World";

int main(){
   std:: cout << greeting; // prints nothing
}
```

### Member template specialisations are not allowed in class scope ###

```C++
class Foo{
  template <typename T> struct Bar;
  struct Bar<char>; // raises [ESC, ESN]
}
```

Further, specialising a member template without specialising all enclosing type definitions is not allowed. 
Example [here](https://gist.github.com/uni-bbl/c0d61799e62bd9a8ffe80965cfda1431).
Reference [here](http://en.cppreference.com/w/cpp/language/template_specialization).

## Related Errors ##

### Clang ###

```
[CAD] Cannot add a default template argument to the definition of a member of a class template [DTS] Declaration of 'T' shadows template parameter 
[EPF] Expected '(' for function-style cast or type construction 
[ESA] Explicit specialisation of T after instantiation 
[ESC] Explicit specialisation of T in class scope 
[EQN] Expected a qualified name after 'typename' 
[NTT] A non-type template parameter cannot have type T 
[UTN] Unexpected type name 'T': expected expression 
[UTK] Use 'template' keyword to treat 'NodeTypes' as a dependent template name 
```

### GCC ###

```
[ESN] Explicit specialisation in non namespace scope
```
