source of this entire page is [here](https://github.com/methylDragon/coding-notes/blob/master/C++/08%20C++%20-%20Linkages%20and%20Preprocessor%20Directives.md)

# C++ Build Process

**Flow**
Editor <--> Pre-processor <--> Compiler <--> Linker <--> Dynamic library Loader <--> Input Output Executer

## Compilation steps
1. **Pre-processing** : ```#include``` , ```#define``` and other preprocessor directives. In prepocessing these are completely replaced by some form of text and forms a **completely parseable C++ file**.
2. **Compilation** : Compiles **each source file** separately into assembly code and forms **object file**.
3. **Linking** : Linker takes the object file from above and forms either a library file(.lib or .dll) or an executable file. The linking instrcutions are given in CMakeLists.txt

## .lib or .dll and .exe files

### .lib
- Contains only compiled code in form of functions, variables, etc. but doesn't include an **entry point (main function)**
- Only meant for static linking i.e. code from ```.lib``` file is copied and integrated in .exe files
- a ```.lib``` files can be created alognside a .dll file to be linked to a program to resolve references issue during compilation
- **CREATION** : C or C++ files is compiled into .obj files and a **linker tool** links this object files resolved references like extern too. The output of linker tool is saved in form of **.lib** files
- On **Visual Studio** one can condfigure project to create a .lib files and on **Command Line Tools** one can use ```cl(compiler)``` and ```lib(linker)``` to do the same



### .dll
- Meant for dynamic loading
- The key difference between .lib and .dll file is that .dll files have **exported functions** defined which makes them accessible and are used by other programs
- The function export works generally using ```__declspec(dllexport)```
- ```DLLMain``` is an entry point to dlls

```
#ifdef _WIN32

// Define dllexport for Windows
#define DLL_EXPORT __declspec(dllexport)

#else

// Define dllexport for other platforms (optional)
#define DLL_EXPORT

#endif

DLL_EXPORT int add(int a, int b) {
  return a + b;
}
```

**DLLMain**
- It is an optional entry point to a DLL
- When a DLL is loaded using ```LoadLibrary``` and unloads using ```FreeLibrary``` , it utilizes ```DLLMain```

*Sample code*

```
BOOL WINAPI DllMain(

HINSTANCE hinstDLL, //A handle to the DLL instance itself.

DWORD fdwReason,   //Flag indicating reason for call
                   //DLL_PROCESS_ATTACH: Process initialization or LoadLibrary call.
                   //DLL_THREAD_ATTACH: Thread creation.
                   //DLL_THREAD_DETACH: Thread termination.
                   //DLL_PROCESS_DETACH

LPVOID lpvReserved); //Reserved parameter, usually set to NULL.
```

### .exe
- Contains normal code + necessary instructions and resources to launch on its own
- It contains the **entry point - the main function** which is required for the program to launch


# Creating a .dll 

**Source Code**
```
#include <windows.h>

// Define the export macro for portability
#ifdef _WIN32
#define DLL_EXPORT __declspec(dllexport)
#else
#error "This code is currently only supported for Windows systems."
#endif

// Function to add two numbers
DLL_EXPORT int Add(int a, int b) {
    return a + b;
}

// Optional DllMain function
BOOL WINAPI DllMain(HINSTANCE hinstDLL, DWORD fdwReason, LPVOID lpvReserved) {
    switch (fdwReason) {
        case DLL_PROCESS_ATTACH:
            // Perform any process-level initialization here if needed
            break;
        case DLL_PROCESS_DETACH:
            // Perform any process-level cleanup here if needed
            break;
        case DLL_THREAD_ATTACH:
        case DLL_THREAD_DETACH:
            // Perform any thread-level initialization/cleanup here if needed
            break;
    }

    return TRUE; // Indicate successful execution
}

// Add a main function for testing purposes can be removed in production
int main() {
    int x = 5, y = 10;
    int result = Add(x, y);
    std::cout << "Result: " << result << std::endl;
    return 0;
}
```
- ```#include <windows.h>```: This header is essential for Windows API functions like DllMain.
- ```DLL_EXPORT``` marks the Add function for exporting.
- 

**Compilation**

```
cl /EHsc /Fe:add_dll.dll add_dll.cpp /link user32.lib kernel32.lib
```
- In your project settings, configure the output type as "DLL" and link the necessary libraries (e.g., ```kernel32.lib``` ).
- Replace ```/EHsc``` with the appropriate exception handling model for your project.
- Adjust the library paths (```user32.lib``` and ```kernel32.lib```) if necessary

# Linkages
source is [here](https://www.goldsborough.me/c/c++/linker/2016/03/30/19-34-25-internal_and_external_linkage_in_c++/)
<br>
Linkages are basically visibility of symbols/variables/functions to the linkers in a translation unit
Linkers look into the aspect of internal and external linkages before linking the object files. Following are the keywords and their linkge behaviour:
1) **static** : internal linkage
2) **extern** : external linkage
3) **non-const global variables** : external linkage
4) **const global variables** : internal linkage
5) **functions** : external

> Explaination <br>
> - **Translation Units** : It means all the .cpp / .c files and header files .h/.hpp files it includes
> - **Internal Linkage**  : These are visible to the _linker_ within that translation units
> - **External Linkage**  : _Linker_ can see it when processing other translation units too. The symbols are visible among different translation units

## Translation Units

Header expanded in a .cpp files makes a translation unit. Below is an example :
```
int strlen(const char* string); //-> Exanded from #include "header.hpp"

int strlen(const char* string)
{
	int length = 0;

	while(string[length]) ++length;

	return length + VALUE;
}
```

## Linker
You can make your linker very angry if your symbol is declared and defined in the same file.<br>
And if you include that file in multiple places then it will cry. (due to heavy code inclusion)

### External Linkage

**extern examples**
```
extern int x;
extern void f(const std::string& argument);
```
```int x``` cannot be a global variable as it comes with definition due to default constructor
```
int x;          // is the same as
extern int x{}; // which will both likely cause linker errors.
extern int x;   // while this only declares the integer, which is ok.
```

**C Way**
Using macros, they can be used across files
```
#define CLK 1000000
```

**C++ Way**
Using extern variables inside namespace:
```
// global.hpp
namespace Global{
	extern unsigned int clock_rate;
}
// global.cpp
namespace Global{
	unsigned int clock_rate = 1000000;
}
```

**Linking Process**

- extern declared in ```header.h```
```
//header.h
extern int x;
```
- The definition of the extern keyword defined in ```header.cpp```
```
//header.cpp
int x = 5
```
- main file outputs the value of x by just including ```header.h```
```
//main.cpp
#include "header.h"
int main(){
	std::cout << x;
}
```

In main.cpp header.h expands to extern int x and turns into a translation unit. Without including header.cpp, how does main.cpp is able to find definition of x?

**Explaination** <br>
Here two translation units are generated:
- One with `header.cpp` and included `header.h`
- One with `main.cpp` and included `header.h`

In compilation process `main.cpp` expands to :
```
extern int x;
int main() { std::cout << x; }
```

Now the job of the linker is the find the definition of `x` in other object files generated as x is an external linkage and finds it in one of the translation units. <br> Similar behaviour is observed with functions are by default external linkage.

### Internal Linkage
The symbols with internal linkage are visible within same translation unit. <br>

If you declared a internal linkage symbol in a header file, then each translation unit you include this header file in will get its own unique copy of that symbol as if you redefined each symbol in the translation unit.

**Example**
```
// header.h
static int value = 0; // Internal linkage symbol
```
```
// translation_unit1.cpp
#include "header.h"

void increment() {
  value++;
}
```
```
// translation_unit2.cpp
#include "header.h"

void print_value() {
  std::cout << value << std::endl; // Might not print the expected value
}
```

**Explaination :**
- In this example, value is declared as static in the header. However, each translation unit gets its own copy of value due to inclusion. This means that incrementing value in _translation_unit1_ might not affect the value printed in _translation_unit2_. <br>
- To avoid this issue, **avoid** declaring static symbols in header files.

**Make symbols internal linkage explicitly**

Using anonymous namespaces, one can access the variable within that translation unit
```
namespace {
int variable = 0;
int cannotAccessOutsideThisFile() { ... };
}
```
does (almost) the same thing as this:
```
static int variable = 0;
```
**One Definition Rule**
- You can declare something multiple times across all translation units for a particular exectuable
- You cannot define something multiple times across all translation units for a particular executable
- There can only be **ONE** definition per symbol within each translation unit!

# Preprocessor Directives

source is [here](https://www.math.utah.edu/docs/info/cpp_1.html)

- Macros make code very hard to debug
- If you can afford to not use them then better not

**Steps of building an executable**
Preprocessor (Include Header, Expand Macro) <--> Compiler (Assembly Code) <--> Assembler (Machine Code)<-->  Linker <--> .exe file

## Comments
Should be always like this `/* like this one */`

## newline character
```
int someFun\
ction()\
{
	return \
		1;
}

int main(){
	std::cout << someFunction();
}
```

The above code works as `\` is deleted

## Macro concatenation
```
#define WOW(NAME, VALUE) int wow_var_ ## NAME = VALUE; /* Definition */
WOW(A, 5) // Usage
    
// When preprocessed, becomes
int wow_var_A = 5;
```
## Macro Arguments

We can make functions too for varied usage:

```
#define PRINT_THIS(X) std::cout << X; /* Definition */
PRINT_THIS("A"); // Usage

// expands to
std::cout << "A";
```

Using it to pass another set of arguments:
```
#define min(X, Y)  ((X) < (Y) ? (X) : (Y)) /* Definition */
min(1, 2); // Usage

// When preprocessed, becomes
((1) < (2) ? (1) : (2));

// It supports arbitrary arguments too!
min(x + y, foo(z));

// Becomes
// Beware with this, since foo(z) gets called twice as a result of this macro expansion
((x + y) < (foo(z)) ? (x + y) : (foo(z)));
```

## if-elif

```
#ifdef SOME_MACRO_NAME
// RUN THIS IS SOME_MACRO_NAME IS DEFINED
#endif

#ifndef SOME_OTHER_NAME
// RUN THIS IF SOME_OTTHER_NAME IS NOT DEFINED
#endif

// These are equivalent to
#if defined (SOME_MACRO_NAME)
#endif

// And
#if ! defined (SOME_OTHER_NAME)
#endif
```

## Errors and Warnings
```
//#define SOME_OTHER_CONDITION 1

#if SOME_CONDITION
#error THROW THIS ERROR MSSAGE
#endif

#if SOME_OTHER_CONDITION
#warning THROW THIS WARNING MESSAGE
#endif
```
The above case would show error in case no definition is found during the compile time itself.

## Good usages

### Variable Expansion
```
#define DOUBLE_VAR(n, v) int int_ ## n(v); double double_ ## n(v);

int main(){
	DOUBLE_VAR(2, 3); //becomes int int_2(3);
	DOUBLE_VAR(b, 3); //becomes int int_b(3);
}
```
### Function Expansion

```
#define STORE_VAR(n, v) int n(void)\
{\
  static stored_var_ ## n = v;\
  return stored_var_ ## n;\
}

// Usage
STORE_VAR(A, 5);
A();

// Becomes
int A()
{
  static int stored_var_A = 5;
  return stored_var_A;
}
A();
```
**How did into come here**

```
int A(void) {
  static stored_var_A = 5;  // Replace ##n with A
  return stored_var_A;
}
```
is functionally equivalent too:

```
int A() {
  static int stored_var_A = 5;
  return stored_var_A;
}
```
## Macro Prescans

[source](https://github.com/methylDragon/coding-notes/blob/master/C++/08%20C++%20-%20Linkages%20and%20Preprocessor%20Directives.md#314-macro-prescans-)

This understanding of macros will help you in debuggin them :
```#define MACRO_NAME MACRO_TEXT```

`MACRO_TEXT` will replace `MACRO_NAME` and withing `MACRO_NAME` it will search for an predefined macro. If it has  #s to stringify the macro, then it will stringfy it there itself **uncless you use prescan method before it**.

```
#define str(s) #s
#define foo 4
str (foo)
``` 
// Will become "foo"
If you wanted to stringify it, you can use the prescan mechanism to do it
```
#define xstr(s) str(s)
#define str(s) #s
#define foo 4
xstr (foo)
```
// Will become "4"

**NOTE**: Never forget to put your macro expressions withing paranthesis.

## Prefdefined Macros
[source](https://gcc.gnu.org/onlinedocs/cpp/Standard-Predefined-Macros.html)

### __LINE__, __FILE__
```
std::cout << "File name: " << __FILE__ << std::endl;
File name: C:\Users\pk152268\source\repos\Macros\Macros\Macros.cpp

std::cout << "Line number: " << __LINE__ << std::endl;
Line number: 18
```
### __cplusplus
For conditional compilation based on various C++ versions. At the compile time itself it compiles the code based on current C++ version :
```

#ifdef __cplusplus
#  if __cplusplus == 201703L  // Check for C++17 standard
#    define USE_C17_FEATURES
#  elif __cplusplus == 202002L  // Check for C++20 standard
#    define USE_C20_FEATURES
#  else
#    // Code for older standards (C++98, C++11, C++14)
#  endif
#else
#  error "This code requires a C++ compiler."
#endif

int main() {
#ifdef USE_C17_FEATURES
    // Use features specific to C++17 or later, such as fold expressions
    std::cout << std::accumulate({ 1, 2, 3, 4 }, 0, [](int x, int y) { return x + y; }) << std::endl;
#elif defined(USE_C20_FEATURES)
    // Use features specific to C++20 or later, such as concepts
    // (Concepts require additional library support, not shown here)
    std::cout << "C++20 features supported" << std::endl;
#else
    // Use features compatible with older standards
    int sum = 0;
    for (int num : {1, 2, 3, 4}) {
        sum += num;
    }
    std::cout << sum << std::endl;
#endif

    return 0;
}
```

### __DATE__, __TIME__
```
#include <iostream>
#include <string>

int main() {
  std::string compile_date = __DATE__;
  std::string compile_time = __TIME__;

  std::cout << "This program was compiled on: " << compile_date << " at " << compile_time << std::endl;

  return 0;
}
```

### __STDC__, __STDC_VERSION__
```
#include <iostream>

int main() {
#ifdef __STDC__
    std::cout << "This compiler conforms to an ISO C standard (at least C89)." << std::endl;

#ifdef __STDC_VERSION__
    std::cout << "Supported C standard version (if defined): " << __STDC_VERSION__ << std::endl;
#else
    std::cout << "C standard version details unavailable." << std::endl;
#endif
#else
    std::cout << "This compiler might not fully conform to an ISO C standard." << std::endl;
#endif

    return 0;
}
```

### __FUNCTION__

```
#include <iostream>
#include <string>

void printFunctionCall(const std::string& message) {
  std::cout << "Function " << __FUNCTION__ << " called with message: " << message << std::endl;
}

int main() {
  printFunctionCall("This is a message from main");
  return 0;
}
```
__FUNCTION__ prints the current function name

### __VA_ARGS__ 

- It's a variadic macro and you can call it with any number of arguments.
- It is used along with `...` which indicates that the macro can accept zero or more additional arguments of any type (__VA_ARGS__).

We are making a custom printing macro function:
```
#include <iostream>
#include <string>

// Variadic macro for printing messages with different types
#define PRINT_VARARGS(message, ...)  \
  do {                               \
    std::cout << message << " : ";   \
    print_arguments(__VA_ARGS__);    \
  } while (0)

// Helper function to print individual arguments
void print_arguments() {}

// Function template to handle arguments of different types
template <typename T>
void print_arguments(T arg) {
  std::cout << arg << std::endl;
}

// Function template overload to handle multiple arguments
template <typename T, typename... Args>
void print_arguments(T first, Args... rest) {
  std::cout << first << ", ";
  print_arguments(rest...); // Recursive call to handle remaining arguments
}

int main() {
  PRINT_VARARGS("Integer value", 42);
  PRINT_VARARGS("Floating-point value", 3.14);
  PRINT_VARARGS("String message", "Hello, world!");
  PRINT_VARARGS("Multiple arguments", 10, "apple", 20.5);
  return 0;
}
```
