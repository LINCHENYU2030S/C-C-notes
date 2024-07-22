# Notes for CPP

* CPP Basics
    * [Namespace ](#name-space)
    * [struct in cpp ](#struct-in-c)
    * [constant variable in cpp ](#constant-variable-in-c)
    * [Pass by reference ](#pass-by-reference)
    * [Inline method ](#inline-method)
--------------------------------------------------------

### Name Space

- Purpose: Name Space exists to prevent naming collision in cpp. 
- Note that namespace can only be written in global region. Example of use: 
```cpp
namespace Math {
    int add(int a, int b) {
        return a + b;
    }
}

namespace Physics {
    int add(int a, int b) {
        return a + b;
    }
}
int main() {
    cout<<Math::add(2, 3)<<std::endl;
    cout<<Physics::add(2, 3)<<std::endl;
}
```

- we can also chain namespace with another namespace:
```cpp
namespace A {
    int a = 20;
    namespace B {
        int b = 10;
    }
}
int main() {
    cout<<A:a<<endl;
    cout<<A:B:b<<endl;
}
```

- anonymous namespace as static global variable:
```cpp
namespace {
    int a = 0;
}
// The above is equivalent as static int a = 0;
```

- access a variable by its namespace only after that variable is added into that namespace:
```cpp
namespace A {
    int a = 0;
}

int main() {
    cout<<A::b<<endl; // error since variable b is added to namespace A after this line.
}

namespace A {
    int b = 10;
} // add variable b to the existing namespace A (not redefine namespace A)
```

- change the name of an existing namespace:
```cpp
//......
void change() {
    namespace newA = A;
}
```

- "using" keyword as declaraction:
```cpp
namespace A {
    int a = 0;
}

void func() {
    using A:a; // same as bringing A::a inside this function so that we can access it by just 'a' instead of 'A::a'
    cout<<a<<endl;
    int a = 1; // error since colliding with A::a after the statement using A::a;
}

```

- "using" as compilation command: allow direct accessing of the elements of a namespace without using the name of the namespace.
```cpp
using namespace std; // allow direct access to all the variables under the namespace "std".
namespace A {
    int a = 1;
    int b = 2;
}

void func() {
    using namespace A;
    cout<<a<<endl;
    cout<<b<<endl;
}
```

--------------------------------------------------------

### struct in C++

- Unlike in C, we do not need to use the struct keyword everytime we want to define a variable in C++.
- Unlike in C, we can define related function inside struct declaration. For example:
```cpp
struct A {
    int a = 0;
    void fun() {
        cout<<"hello world"<<endl;
    }
}
int main() {
    A aa;
    aa.fun();
}
```

--------------------------------------------------------

### constant variable in c++

- unlike in C, a global constant variable in cpp will not be allocated with memory. The compiler will 
replace every occurrence of global constant variable with its constant value:
```cpp
const int a = 10;
int main() {
    cout<<a<<endl; //after compilation, this line becomes cout<<10<<endl;
}
```

- when you use "volatile" keyword or you take the address of a local constant variable, the local constant variable will be allocated memory:
```cpp
int main() {
    volatile int a = 10;
    int *p = (int*) &a;
    *p = 100;
    cout<<a<<endl;//a replaced by 10 will not occur because of the 'volatile' keyword, output 100
    cout<<*p<<endl;// output 100;
}

```
- consant global variable can only be accessed in current file, except if we define it using "extern" keyword:
```cpp
const int a = 10; // a can only be accessed in current file
extern const int b = 20; // b can be accessed in other files by using 'extern const int b;' in other file.
```

- when assiging the value of another variable to a constant variable, the line of code can not be optimized by the compiler during compilation. For example:
```cpp
int main() {
    int a = 10;
    const int b = a; // compiler will not be able to optimize by replacing every occurrence of b with 10.
    int *p = (int *) &b;
    *p = 20;
    cout<< b << endl; // output 20, since b here is not replaced with 10 by the compiler.
}
```

- const struct in cpp can not be optimized. For example:
```cpp
struct A {
    int a = 10;
}
int main() {
    const A aa;
    cout<<aa.a<<endl; // aa.a is not replaced by 10
}
```

--------------------------------------------------------
### Pass by reference

- Just like in C, a variable name is just a name to a contiguous memory block. we can change/read the content of that memory through this name.

- Remember one usefulness of pointer is to save memory since we just need to pass 4 bytes of address (in 32bit system) value to a function call instead of passing value of an argument that possibly occupy a lot of memory in our call stack.

- In C++ (also in some other languages), there is another way that is called "pass by reference". For example:
```cpp
int a = 10;
int &b = a; // after this line of code, both a and b are names to the same block of memory! b is a reference to a!
/* 
Note:
1) The symbol '&' here is not "taking address", it means we are declaring an alias to an existing variable name.
2) When declaring 'b', we must initialize it at the same time.
3) Only declararing alias to a valid existing memory region is allowed, reference to NULL is not allowed.
4) after declaring this alias(and initialize it), we cant change its memory reference.
*/
```

- By using pass by reference, it makes us more convenient as compared to passing pointer into function call(by user, remember that pass by reference under the hood is still passing constant pointer to function call). For example:
```cpp
void func(int &b) { // when passing argument, it's equivalent to int &b = a; so we can edit the memory region referenced by a!
    b = 10;
}
int main() {
    int a = 1;
    func(a); 
}
```
- There are 3 ways to declare alias to reference to an array in cpp:
```cpp
int main() {
    int arr[5] = {1, 2, 3, 4, 5};

    /* Method 1 */
    // define array type
    typedef int(arrType)[5];
    // build up reference relationship
    arrType &arrref = arr;

    /* Method 2 (most often used)*/ 
    // direct build up reference relationship
    int (&arrref2)[5] = arr;

    /* Method 3 */
    // define reference array type
    typedef int(&arrTypeRef)[5];
    // build up reference relationship
    arrTypeRef arrref3 = arr;
}
```

- Behind the scence, pass by reference is implemented using constant pointer(thus occupy the same memory size as pointer):
```cpp
TYPE &a = b; // behind the scene: TYPE* const a = &b;

// example:
int b = 10;
int &a = b; // compiler will do int * const a = &b;
a = 20; // behind the scene: *a = 20;
```

--------------------------------------------------------
### Inline Method

- How does inlining a method can POSSIBLY increase the efficiency of a program: when inlining a method, the compiler will replace everywhere where the method is called with its detail instruction so that the cost for functional call is saved during execution(does not need to set up call stack frame).

- What kind of method should not inline: function with high complexity (e.g. many if-else statement, loop etc), function will large piece of code/instruction. we encourage inlining small, frequently called method.

- When large function is inlined (and the compiler does not reject it), the binary file after compilatin will become very big, leading to code bloat. This will possibly slow down the efficiency of the program since the miss rate for instructions cache (Icache) will possibly increase since it now cannot hold most of the instructions.