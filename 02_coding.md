# When to use static inline instead of regular functions
	-- from: https://stackoverflow.com/questions/28926828/when-to-use-static-inline-instead-of-regular-functions

# Function Pointer in C Struct
The function pointer is a pointer hold the address of the function. Since C is not OOP language I Consider the Function Pointer as the Father of virtual functionality in the modern languages. In the oop language each driven class will  implements the virtual method depend on its need. In the C it is something similar, since we give the function pointer the address of the desired function implementation.

	-- from: https://www.codeproject.com/Tips/800474/Function-Pointer-in-C-Struct

# printf("\e[1;1H\e[2J");
Using \e[1;1H\e[2J Regex :
 
In C, \e[1;1H\e[2J regex is used to clear the console screen just like any other method or function.

Where

/e provides an escape. and e [1;1H] place your cursor in the upper right corner of the console screen. and e [2J adds a space to the top of all existing screen characters.

Whenever regex is invoked in a program, it clears the previous console screen and makes the system available for new data.

Properties :
Not platform specific
Easy to remember and understand

	-- from: https://www.geeksforgeeks.org/clear-console-c-language/

# write (C System Call)
ssize_t write(int fildes, const void *buf, size_t nbytes);
Field	Description
int fildes	The file descriptor of where to write the output. You can either use a file descriptor obtained from the open system call, or you can use 0, 1, or 2, to refer to standard input, standard output, or standard error, respectively.
const void *buf	A pointer to a buffer of at least nbytes bytes, which will be written to the file.
size_t nbytes	The number of bytes to write. If smaller than the provided buffer, the output is truncated.
return value	Returns the number of bytes that were written. If value is negative, then the system call returned an error.

	-- from: http://codewiki.wikidot.com/c:system-calls:write

# snprintf() in C library
The snprintf() function is defined in the <stdio.h> header file and is used to store the specified string till a specified length in the specified format.

Characteristics of snprintf() method:
The snprintf() function formats and stores a series of characters and values in the array buffer. 
The snprintf() function accepts an argument ‘n’, which indicates the maximum number of characters (including at the end of null character) to be written to buffer. 
The snprintf() function is used to redirect the output of  printf() function onto a buffer. 
The snprintf() also returns the number characters that were supposed to be written onto the buffer (excluding the null terminator), irrespective of the value of ‘n’ passed.
So, only when the returned value is non-negative and less than ‘n’, the string has been completely written as expected.

Syntax:
int snprintf(char *str, size_t size, const char *format, …);

	-- from: https://www.geeksforgeeks.org/snprintf-c-library/

# argv[0] in c
the first argument (argv[0]) is the name by which the program was called.

	-- from: http://crasseux.com/books/ctutorial/argc-and-argv.html

# C sample
```
// C program to illustrate
// fgets()
#include <stdio.h>
#define MAX 15
int main()
{
	char buf[MAX];
	fgets(buf, MAX, stdin);
	printf("string is: %s\n", buf);

	return 0;
}
```

/*
Input:
Hello and welcome to GeeksforGeeks

Output:
Hello and welc
*/

```
char *fgets(char *str, int n, FILE *stream);

/*
Return Value
On success, the function returns the same str parameter. 
If the End-of-File is encountered and no characters have been read, the contents of str remain unchanged and a null pointer is returned.

If an error occurs, a null pointer is returned.
*/
```

```
unsigned long long int strtoull(const char* str, char** endPtr, int base);
/*
str is a pointer to constant character. str points to string representing an integer.
endPtr is pointer to character pointer. On success it points to first character after number otherwise points to NULL.
Last parameter is base of string input.
On success the function return converted integer as unsigned long long int type and set endPtr to point to first character after number. On failure it return 0 and set endPtr to point to NULL pointer.
It handles integer overflows efficiently and return ULONG_LONG_MAX on overflow. However,
for integer underflows its behaviour is undefined.
*/
```
