# Notes for C

* C Basics
    * [Step by step compilation of C Program ](#step-by-step-compilation-of-c-program)
    * [A Basic Computer Architecture ](#a-basic-computer-architecture)
    * [Some shortcuts for VScode IDE ](#some-shortcuts-for-vscode-ide)
    * [Data type in C ](#data-type-in-c)
    * [Input/Output methods ](#inputoutput-method)
    * [Pointers ](#pointers)
    * [Some Useful String Processing Methods ](#some-useful-string-processing-methods)
    * [Variable Scoping ](#variable-scoping)
--------------------------------------------------------

### Step by step compilation of C Program

#### Preprocessing : gcc -E filename.c -o filename.i
- Syntax for preprocessing command: #
- Expand header file, e.g. #include <stdio.h> copy the file stdio.h to the preprocessed file.
- Eliminate comments
- Does not check syntax error.
- Syntax for conditional compilation: #if 0 ... #endif or # if 1 ... #endif (0 means false, 1 means true)
- Other preprocessing commands such as #define _CRT_SECURE_NO_WARNINGS or  #pragma warning(disable:4996) to ignore warnings
- To prevent repeating defining function especially in header file:
```c
#ifndef _FILENAME_H_  //means if _FILENAME_H_ has not yet been defined, then we enter and define it.
#define _FILENAME_H_

int func(int a, int b);
int haha(char a, char b);

#endif
// or equivalently we can do:
#pragma once

int func(int a, int b);
int haha(char a, char b);
```


#### Compilation: gcc -S filename.i -o filename.s
- Convert the preprocessed file into assembly file
- check syntax error

#### Conversion to Binary: gcc -c filename.s -o filename.o
- convert the assembly file into binary file

#### Link: gcc filname.o -o filename
- set up execution environment, heap and stack etc and link to library code.

#### Additional Note:
- After compilation is done, the memory needed for text(code), data(global variable, static variable, constant), and bss(variables that havent been initialised) is "planned" (havent yet been allocated memory).

--------------------------------------------------------

### A Basic Computer Architecture

#### 32 bit system vs 64 bit system
- For 32 bit system, it can process 32 bits of data in one go, address up to 2^32(4GBytes) of RAM, and has registers that are 32 bits wide.
- For 64 bit system, it can process 64 bits of data in one goo, address up to 2^64 bytes of RAM (modern hardware typically supports up to a few terabytes), and has registers that are 64 bits wide.
- Modern 64 bit CPUs are designed to support 32-bit instructions. A 64-bit operating system often inclues a compatibility layer(a software component) to run 32-bit applications.
##### Compatibility layer in 64-bit OS for running 32-bit applications
<details>
<summary>Expand</summary>

##### Separate subsystem for 32-bit applications:
- On linux, this is often achieved through multiarch support, where libraries and runtime environments for both 32-bit and 64-bit applications are installed side by side. When a 32-bit application runs, the 32-bit versions of system libraries are loaded.

##### Redirection of system calls:
- When a 32-bit application makes a system call, the compatibility layer intercepts it and translates it into a corresponding 64-bit system call that the operating system can understand.
- This involves mapping 32-bit memory addresses to 64-bit memory addresses and handling differences in data structures and function signatures.

##### Handling Execution Contexts:
- The compatibility layer manages the CPU modes, ensuring that the processor can switch between 32-bit and 64-bit modes as needed. It also ensures that the application's execution context such as registers, stack etc is appropriately set up for 32-bit code execution.
</details>

- A 32-bit CPU cannot execute 64-bit instructions because it has imcompatible registers, buses to handles 64-bit data. Even if the hardware somehow supported it, a 32-bit operating system wouldn't be able to manage the large memory address space and the 64-bit registers effectively, thus affecting the execution performance.

#### Architecture for data storage
- Registers: Closest to CPU. A hardware that temporarily stores the data and provide very fast data access for CPU. 
- Cache: 2nd Closest to CPU. A hardware that tries to store "the next most likely needed" data (depending on how the cache is implemented, LRU is one possible implementation). A high cache hit rate improves the execution performance.
- RAM: 3rd Closest to CPU. A physical memory space that stores text(code to be executed), data, heap, and stack(There is a mapping between virtual memory space and physical memory space).


--------------------------------------------------------

### Some shortcuts for VScode IDE
- ctrl+k, ctrl+f: autocorrect code format
- ctrl+k, ctrl+c: comments selected code
- ctrl+k, ctrl+u: uncomments selected lines
- F9: set breakpoint
- F5: run and debug
- ctrl + F5: run only
- ctrl + shift + b: compile only
- F10: next 
- F11: step
- option + up/down (windows: alt +up/down): move selected line(s) up or down


--------------------------------------------------------

### Data type in C

#### Categories of data types
- Fundamental data type: char(1 byte), short(2 bytes), int(4 bytes), long(4bytes in windows and 32-bit linux, 8bytes in 64-bit linux), float(4bytes, 7 significant numbers), double(8 bytes, higher precision, 15 significant numbers)
- Array of some other data type: Used to group data of the same type in contiguous memory. The name of the array is like a constant name of an address(the address of the first element of the array), in other words, it is a pointer! Thus, we have &array[0] == array == &array. One thing to note is both &array[0] + 1 and array + 1 are used to skip to the next element of the array, but &array + 1 skip the whole array.
#### Some keywords
- extern: Just tell the compiler there exist some variable in our program, OS does not allocate new memory for just this line of code
- const: declare a variable to be constant so that we can't change the value stored in the memory address corresponding to that variable name through variable name. (But still can change the value through its memory address)
- volatile: tell the compiler not to improve the code involving a particular variable.
- register: recommend the OS to define a particular variable on a register(if there exist a free register) to enhance the efficiency.


#### Signed vs Unsigned:
- difference: signed can represent both positive and negative and zero values while unsigned can only represent nonnegative values, thus their range is different.
- Negative representation in Signed Integer: Take complement and plus one.
- Note: If you assign a 10-based value to a variable, you are assigning the binary that is taken complement and plus one. But if you assign a 8-based or 16-based value to a variable in C, then you are assigning the equivalent binary to that variable(without taking complement and plus one, because assumed to be complement number),

#### Type conversion:
- In order to make sure that there will not be any data loss, we normally convert a smaller in size data type to a bigger one(or at least same size). For example, converting from char to int, from int to double, from int to float etc. If we do the other way around, some data may loss, for example:
```c
int a = 12345678;
char b = (char) a; // three bytes of data will loss
```

--------------------------------------------------------

### Input/Output method

#### scanf (read until either space or \n is encountered)
- use: scanf("%format", memory address), for example:
```c
int a = 0;
scanf("%d", &a);

char b = 0;
scanf("%c", &b);

char c[5] = {0};
scanf("%s", c); // c itself is a constant to the array memory address
```

- disadvantage: in the case of reading string into a char[] variable, if the size of input string is bigger than the size of char[] variable, memory pollution will occur. for example:
```c
char string[5] = {0};
scanf("%s", string); 
// if user input like helloworld which has length bigger than 5, scanf method will just read starting from the address given until all of the word "helloworld" is read in, in which memory not belong to the char array will be overridden.
```

#### gets (read until \n is encountered, will read in space)
- use : gets(* char)
- disadvantage: the memory pollution as described in the scanf method will also occur here.

#### fgets(read until \n is encountered or maximum size is reached)
- use: fgets(*char, max_size, input stream) //will read in at most max_size - 1 of chars and automatically add in \0
- example of use:
```c
char a[128];
fgets(a, sizeof(a), stdin); //will read in at most 127 of chars and then add in \0
```
- disadvantage: will read in \n as well, but we can resolve it as follow:
```c
#include <string.h>
//...
//...
char a[128] = {0};
fgets(a, sizeof(a), stdin);
a[strlen(a) - 1] = 0; // /0 is equivalent as value 0
```

#### puts (write a string to stdout, automatically include \n)
- use: puts(addres of the first element of the string)
- example of use:
```c
char a[128];
fgets(a, sizeof(a), stdin);
puts(a); // equivalent to printf("%s\n", a);
```

#### fputs(write a string to any file, does not automatically include \n)
- use: fputs(address of the first element of the string, file you want to write to)
- example of use:
```c
char a[128];
fgets(a, sizeof(a), stdin);
fputs(a, stdout); // equivalent to printf("%s", a);
```

--------------------------------------------------------

### Pointers
- What is pointer? Answer: pointer is just a logical memory address
- Pointer variable: a variable that stores a logical memory address
- Why do we need char *, short *, int * different types of pointer variable since they are all just some variable stores 4 bytes(or 8 bytes) of memory address value. Answer: When we do dereferencing, the number of bytes of data accessed are different, depending of the type of pointer. Their differences are shown as follow:
```c
int num = 0x12345678;

char *pointer1 = (char *) &num;
short *pointer2 = (short *) &num;
int *pointer2 = &num;

printf("%x ", *pointer1);
printf("%x ", *pointer2);
printf("%x ", *pointer3);
/*
The output would be
78 5678 12345678
The reason is pointer1 is of type char *, when we dereference it it will just access one byte of data given by the address. The same reason for pointer2, since pointer2 is of type short *, *pointer2 will just access 2 bytes of data, 1 byte from exactly what pointer2 is storing and 1 byte from the following byte.
*/


//Another difference
printf("%x ", pointer1 + 1);
printf("%x ", pointer2 + 1);
printf("%x ", pointer3 + 1);
/*
One possible output would be (assuming the address of num is 0x00000000)
0x00000001 0x00000002 0x00000004
(+ 1byte)  (+ 2bytes)  (+ 4bytes)
*/
```

- General Pointer (void *): used to store the address of any data type. Type casting is needed when using it. For example
```c
int a = 0;
void *p = (void *) &a;
printf("%d", * (int *)p) // type cast p into int * and take dereferencing
```

- Using keyword "const" to prevent changing content through pointer dereferencing.
```c
int a = 10;
const int *p = &a;
*p = 100 // fails

// However, the following dereferencing will work
const int a = 10;
int *p = &a;
a = 100; // fail
*p = 100; // success
```

- Using keyword "const" to prevent changing pointer's contenct (in contrast to preventing changing the content where the pointer points to ). For example:
```c
int a = 10;
int b = 1;
int * const p = &a;
p = &b; // changing where p points to will fail

//Thus, the following will prevent changing the content where pointer points to as well as prevent changing where the pointer pointers to
const int * const q = &a;
*q = 12; // fails because of first "const"
q = &b; // fails because of seecond "const"
```

- pointer equivalent to array: As we know, the array name is just a constant to a logical memory address, which is just a pointer!. For example we can do:
```c
int a[10] = {0};
int *p = a
p[0] = 10; // the 0th element of the array is now 10
a[0] = 11; // the 0th element of the array is now 11
*(p + 1) = 12; // the 1st element of the array is now 12
*(a + 1) = 13; // the 1st element of the array is now 13
// thus a[i] equivalent to *(a + i) equivalent to p[i] equivalent to *(p + i)

// However, the following pointer is different from p, although they stores the same address
int *q = &a; // q + 1 is different from p + 1, q + 1 will skip the whole array!

// we can still make them to be the same pointer by:
int *q2 = (int *)&a
```

- Defining a method accepting an array as parameter: the compiler will convert the parameter of type array to a pointer, thus we will also need to pass the number of elements of the array to the method. For example:

```c
void fun(int a[100]) { // equivalent to void fun(int *a), no matter int a[100] or int a[1000]
    // do something
}

// As such, we normally define such function as:
void func(int *a, int len) {
    // do something
}
```

- char * vs char []: Most of the cases char * and char [] acts as the same. However, there are different in some behaviour. For example:
```c
char a[] = "Hello World";
a[0] = "K"; //It works.

// However
char *b = "Hello World"; // Additional Note: when assigning a string constant to a pointer, 
//the address of the first char in the string will be assigned to the pointer
b[0] = "K"; // It fails
*b = "K"; // This fails as well
/*
The reason is that the first Hello World is stored in the array 'a' which was 
preallocated a memory region in the stack while the second Hello World is stored in the constant data region. Changing data content in the constant data region is not allowed.
*/
```

- char *[] used to store the memory address of the program parameter(string), for example:

```c
#include <stdio.h>

int main(int argc, char *argv[]) {
    printf("%d ", argc)
    for (int i = 0; i < argc; i++) {
        printf("%s ", argv[i]);
    }
    //Example run: ./ProgramName hello world 1234
    //Example output: 4 ./ProgramName hello world 1234
    // Note: ./ProgramName is counted as one program argument
}
```


--------------------------------------------------------
### Some useful string processing methods
- strcpy(char *dest, char *src): copy char starting from the address "src" until \0 is encountered (such \0 is also included in copying) to the address starting from "dest"(by overwriting).

- strncpy(char *dest, char *src, int n): same as strcpy, except that copying is terminated until either n characters are copied(exlusive of \0) or \0 is encountered.

- strcat(char *str1, const char *str2): append the string starting from the address str2 (until \0 is encountered) to after the string starting from the address str1.

- strncat(char *str1, const char *str2, int n): same as strcat, except that concatenation is terminated when either n characters starting from str2 is appended or \0 is encountered. 

- strcmp(char *str1, char *str2): compare the ascii value of the two strings starting from the address str1 and str2. Return 1 if "str1" is bigger than "str2", 0 if their value are equal, -1 otherwise.

- strncmp(char *str1, char *str2, int n): Same as strcmp, except that comparison is done until either n characters are compared or \0 is encountered.

- int sprintf(char *str, const char *format, ...): Instead of printing the final string(after combining with the format) to the standard output, we write such final string to "str" and store in it. When the method ends, it will return the length of the final string.

- int sscanf(const char *buf, const char *format, ...): retrieve data from the string starting from buf according to the format given. Return the number of paramters successfully retrived, return -1 if fails.

- char * strchr(char *str, char chr): find the address of the first occurrence of chr in the string starting from str and return it, return NULL if not found.

- char * strstr(char *str1, char * str2): find the address of the first occurrence of str2 in the string starting from str1 and return it, return NULL if not found.

- char * strtok(char *str, const char *delim): tokenize the string with starting address "str" and return the starting address of first token found. For finding the subsequent token, use NULL as str. The argument "delim" should be the string containing all possible deliminator. Example of use:
```c
char str[] = "123#hello$world#hahaha";
char *token[10] = {NULL};
token[0] = strtok(str, "#$"); // parse the string by either # or $
int i = 0;
while (token[i++] != NULL) {
    token[i] = strtok(NULL, "#$");
}
```

- int atoi(const char *str): convert str from string represention to integer type. Accept "+", "-", "0" to "9" to be in str, if space or "\n" is first encountered, it will skip until acceptable char is found. If other unacceptable char is found, just return 0. Other similar function: atof:convert string to float.


--------------------------------------------------------
### Variable Scoping
- local variable: (keyword for declaration: auto, can be omitted)
scoping: inside the closest outer bracket {}
life duration: from the start of function execution until function execution finishes.
default value before initialization: random

- static local variable:
scoping: inside the closest outer bracket {}
life duration: from the start of program execution until the end of program execution. Operating system will allocate memory for static variable before the execution of main method. The initialization of the static variable will only take place once.
default value before initialization: 0

- global variable:
scoping: whole program and all files (need using "extern" keyword in other files)
life duration: from the start of program execution until the end of program exection. Operating system will allocate memory for global bariable before the execution of main method. The initialization of global variable will only take place once.
default value before initialization: 0

- static global variable:
scoping: only current file.
life duration: from the start of program execution until the end of program exection. Operating system will allocate memory for global bariable before the execution of main method. The initialization of global variable will only take place once.
default value before initialization: 0

- normal function declaration:
scoping: can be invoked from all files

- static function declaration:
scoping: can only be invoked from current file.

#### Additonal Note: 
- Better not to define global variable in header file (but can extern) to prevent multiple initialization of variable.
- Different scoping can have variables of the same name.

