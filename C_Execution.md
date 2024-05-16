# Notes for C

### Execution of C Program
Preprocessing : gcc -E filename.c -o filename.i
- Syntax for preprocessing command: #
- Expand header file, e.g. #include <stdio.h> copy the file stdio.h to the preprocessed file.
- Eliminate comments
- Does not check syntax error.
- Syntax for conditional compilation: #if 0 ... #endif or # if 1 ... #endif (0 means false, 1 means true)

Compilation: gcc -S filename.i -o filename.s
- Convert the preprocessed file into assembly file
- check syntax error

Conversion to Binary: gcc -c filename.s -o filename.o
- convert the assembly file into binary file

Link: gcc filname.o -o filename
- set up execution environment, heap and stack etc and link to library code.