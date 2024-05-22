# Notes for C

* C Basics
    * [Step by step compilation of C Program ](#step-by-step-compilation-of-c-program)
    * [A Basic Computer Architecture ](#a-basic-computer-architecture)
    * [Some shortcuts for VScode IDE ](#some-shortcuts-for-vscode-ide)
    * [Data type in C ](#data-type-in-c)
    * [Input/Output methods ](#inputoutput-method)
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
- use: fgets(*char, max_size, input stream) //will read in at most max_size - 1 of chars and automatically add in \n
- example of use:
```c
char a[128];
fgets(a, sizeof(a), stdin); //will read in at most 127 of chars and then add in \n
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

#### puts (write a string, include \n)
- use: puts(addres of the first element of the string)
- example of use:
```c
char a[128];
fgets(a, sizeof(a), stdin);
puts(a); // equivalent to printf("%s\n", a);
```

#### fputs(write a string to any file, does not include \n)
- use: fputs(address of the first element of the string, file you want to write to)
- example of use:
```c
char a[128];
fgets(a, sizeof(a), stdin);
fputs(a, stdout); // equivalent to printf("%s", a);
```