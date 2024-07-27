# Notes for CPP

* CPP Basics
    * [Namespace ](#name-space)
    * [struct in cpp ](#struct-in-c)
    * [constant variable in cpp ](#constant-variable-in-c)
    * [Pass by reference ](#pass-by-reference)
    * [Inline method ](#inline-method)
    * [Method overloading ](#method-overloading)
    * [Constructor and destructor ](#constructor-and-destructor)
    * [Shallow copy vs Deep copy ](#shallow-copy-vs-deep-copy)
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

- member function of a class is inlined by default.


--------------------------------------------------------
### Method Overloading

- Purpose: method overloading allows more flexibility for programmer to name functions. Functions with different method signature can have same name. When the user invoke such function, the compiler will find the one that match the type of argument passed into.

- Why is overloading not allowed in C but allowed in C++? : When C++ code is compiled to assembly code, C++ compiler will rename the functions based on function name given by the programmer and method signature. Thus, those overloaded functions will have different name during assembly stage.

- Invoke C function in C++: When you write a functin in C and you want to invoke it in C++, the C++ may not be able to find the method since C++ assume the function called has another name in assembly stage. If you want to invoke it, you have to tell the compiler when defining that function written in C:

```c
// In header file:
// Tell the compiler if you want to invoke the below method in C++, you must find it in the same way that C does.
#ifdef __cplusplus
extern "C"
{
#endif
    void func();

#ifdef __cplusplus
}
#endif
```

--------------------------------------------------------
### Constructor and destructor
- In cpp, the compiler will insert call for constructor method of a class automatically everytime we instantiate an object of that class. Similarly, when the memory region where an object of a class is no longer valid, the compiler will insert a call for the desctructor method of that class to "delete" that object. Destructor is especially important when there is memory allocated for one of member variable of the object since we need to free the space allocated.

- There are default constructor and desctructor for a class when programmer does not provide their own constructor and destructor. The default constructor and desctructor are just empty method.

- The constructor(s) and destructor must be public.

- There is also a default copy constructor where we pass a reference of other object of the same class to the constructor so that we can create another "similar" object of the same class. The defailt copy constructor simply assign each of the member variable from given object to the new one.

- Some example use of constructor(the destructor will automatically called when the main method calling stack is no longer used in the following case):

```cpp
class Maker
{
    public:
    Maker() { 
        //...
    }
    Maker(int k) {
        a = k;
    }

    Maker(const Maker &m) { //Note that the & here is important, if not it will lead to infinte loop of copying.
        a = m.a;
    }
    ~Maker() {
        //...
    }

    private:
    int a;
}

int main() {
    Maker m; //constructor without parameter is called.
    Maker m1(10) //constructor with one parameter of type int is called.
    Maker m2(m1); // copying constructor is called.

    // Not that often used:
    Maker m4 = Maker(10); // constructor with one parameter of type int is called.
    Maker m3 = m2; // copying constructor is called.
    Maker m5 = 10; // equivalent to Maker m5 = Maker(10)
    
    Maker m6; //constructor without parameter is called.
    m6 = m5; // there is no constructor called, this is an assignment operation.
}
```

- In the example above, the parameter for copying constructor must be const TYPE &NAME. Consider the case where the copying constructor takes in type const Maker m instead:
```cpp
class Maker {
    public:
    Maker() {
        //...
    }
    Maker(int k) {
        a = k;
    }

    Maker(const Maker m) { 
        //error, here is why:
        // when we do something like Maker m2(m1); to call copying constructor
        // the next instructor will be argument passing: const Maker m = m1; which is also a copying construction action, thus the copying constructor is called again.
        // copying constructor is called again and again,....infinite loop
    }

    ~Maker() {
        //...
    }

    private:
    int a;

}
```

- Member Initializer List: 
(1) Allow a class to call a particular constructor to initialize its member object
(2) Allow value to be passed through a class to its member's constructor.
(3) Note: if member initializer list is used for one constructor, it must be used for all constructor.
(4) if member initializer list is not used, the member constructor without parameter will be called automatically.
For example:
```cpp
#include <iostream>
using namespace std;
class Apple
{
    public:
    Apple(int a) {
        //...
        cout<<"Apple constructor is called"<<endl;
    }

    ~Apple() {
        cout<<"Apple destructor is called"<<endl;
    }
}

class Orange
{
    public:
    Orange(int b) {
        //...
        cout<<"Orange constructor is called"<<endl;
    }

    ~Orange() {
        cout<<"Orange destructor is called"<<endl;
    }
}

class Stall
{
    public:
    Stall(int a, int b): Apple(a), Orange(b)
    {
        //...
        cout<<"Stall constructor is called"<<endl;
    }

    ~Stall() {
        cout<<"Stall destructor is called"<<endl;
    }
    private:
    Apple apple;
    Orange orange;
}

int main()
{
    Stall stall(1, 2);
    //result:
    /*
    Apple constructor is called
    Orange constructor is called
    Stall constructor is called
    Stall desctructor is called
    Orange destructor is called
    Apple destructor is called.
    */
}
```

### Shallow Copy vs Deep Copy

- When we do not provide our own copying constructor for a class we implemented, the default copying constructor will just do simple assignment of fields from one instance to the newly created one. This is what we call as "shallow copy". The aliasing issue will occur in this case. Example:
```cpp
class Student {
    public:
    Student(const char * name, int age) {
        studentName = (char *) malloc(strlen(name) + 1);
        strcpy(studentName, name);
        studentAge = age;
    }

    char * getStudentName() {
        return studentName;
    }

    int getStudentAge() {
        return studentAge;
    }

    void printStudent() {
        printf("student name: %s ", studentName);
        printf("student age: %d\n", studentAge);
    }

    ~Student() {
        if (studentName != NULL) {
            free(studentName);
            studentName = NULL;
        }
    }
    private:
    char * studentName;
    int studentAge;
};

int main() {
    Student s1("hello world", 10);
    Student s2(s1);
    s1.printStudent(); //student name: hello world student age: 10
    s2.printStudent(); //student name: hello world student age: 10
    strcpy(s1.getStudentName(), "world");
    s1.printStudent();//student name: world student age: 10
    s2.printStudent();//student name: world student age: 10

    // at the end, there will be an error since the heap memory region pointed by both s1.studentName and s2.studentName are freed up twice!
}
```

- To achieve deep copy, we can simple reimplement the copying constructor:
```cpp
class Student {

    /*

    ...

    */
    Student (const student &otherStudent) {
        studentName = (char *) malloc(strlen(otherStudent.studentName) + 1);
        strcpy(studentName, otherStudent.studentName);
        studentAge = otherStudent.studentAge;
    }
    private:
    char * studentName;
    int studentAge;
};

int main() {
    Student s1("hello world", 10);
    Student s2(s1); // s1.studentName and s2.studentName are stored at separate memory region.
    // No memory related issue occur since no memory region is freed more than once.
}
```