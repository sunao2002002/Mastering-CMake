# Writing CMakeLists Files
This chapter will cover the basics of writing effective CMakeLists files for your software. It will cover the basic commands and issues you will need to handle most projects. While CMake can handle extremely complex projects, for most projects you will find this chapter’s contents will tell you all you need to know. CMake is driven by the CMakeLists.txt files written for a software project. The CMakeLists files determine everything from which options to present to users, to which source files to compile. In addition to discussing how to write a CMakeLists file, this chapter will also cover how to make them robust and maintainable.

本章将讲述为您的软件编写有效的CMakeLists文件的基础知识。它将会覆盖处理大多数工程所需要的基命令和问题。尽管CMake可以处理及其复杂的项目，但对于大多数项目，您会发现本章将提供所有您说需要的内容。CMake由为软件编写的CMakeLists文件驱动。CMakeLists文件决定了从向用户展示那些选项到编译源文件在内的所有内容。除了讨论如何编写CMakeLists文件外，本章还将介绍如何使他们变得健壮和可维护。

## Editing CMakeLists Files
CMakeLists files can be edited in almost any text editor. Some editors, such as Notepad++, come with CMake syntax highlighting and indentation support built-in. For editors such as Emacs or Vim, CMake includes indentation and syntax highlighting modes. These can be found in the Auxiliary directory of the source distribution, or downloaded from the CMake Download page.

CMakeLists文件可以由大多数编辑器编辑。一些编辑器如Notepad++带有CMake语法高亮和缩进支持。对于Emacs或者Vim这类编辑器，CMake包括缩进和语法高亮模式。这些可以在与源码发布的Auxiliary目录中找到，或者从CMake下载页面下载。

Within any of the supported generators (Makefiles, Visual Studio, etc.), if you edit a CMakeLists file and rebuild, there are rules that will automatically invoke CMake to update the generated files (e.g. Makefiles or project files) as required. This helps to assure that your generated files are always in sync with your CMakeLists files.



对于CMake支持的生成器（如Makefile,VisualStudio等），如果您编辑了CMakeLists并重建，有一些规则会自动调用CMake来更新所生成的文件（如Makefiles或者项目文件），这个有助于保证您生成的文件始终与您的CMakeLists文件同步。

## CMake Language

The CMake language is composed of comments, commands, and variables.

CMake语言由注释、命令和变量构成。

## Comments

Comments start with # and run to the end of the line. See the cmake-language manual for more details.

注释由#开头，并且包含到行尾，参考cmake-language手册获取更详细信息。

## Variables

CMakeLists files use variables much like any programming language. CMake variable names are case sensitive and may only contain alphanumeric characters and underscores.

CMakeLists文件和其他编程语言一样使用变量。CMake中变量名是大小写敏感的，只包含字符和下划线。

A number of useful variables are automatically defined by CMake and are discussed in the cmake-variables manual. These variables begin with CMAKE_. Avoid this naming convention (and, ideally, establish your own) for variables specific to your project.

CMake中自动定义了一些有用的变量，可以参考cmake-varialbes手册。这些变量以CMAKE_开头。避免在您的工程中使用CMAKE_这样的变量命名方式，最好定义自己的方式。

All CMake variables are stored internally as strings although they may sometimes be interpreted as other types.
所有的CMake变量在内部都是已字符串的方式存储，虽然他们有时被解释为其他类型。

Use the set command to set variable values. In its simplest form, the first argument to set is the name of the variable and the rest of the arguments are the values. Multiple value arguments are packed into a semicolon-separated list and stored in the variable as a string. For example:
使用set命令设置变量的值。最简单的格式，第一个参数是变量名称，其他参数是变量的值。多个值参数被打包成以分号分隔的列表，以字符串方式存储。例如：

``` cmake
set(Foo "")      # 1 quoted arg -> value is ""
set(Foo a)       # 1 unquoted arg -> value is "a"
set(Foo "a b c") # 1 quoted arg -> value is "a b c"
set(Foo a b c)   # 3 unquoted args -> value is "a;b;c"
```

Variables may be referenced in command arguments using syntax ${VAR} where VAR is the variable name. If the named variable is not defined, the reference is replaced with an empty string; otherwise it is replaced by the value of the variable. Replacement is performed prior to the expansion of unquoted arguments, so variable values containing semicolons are split into zero-or-more arguments in place of the original unquoted argument. For example:

变量可以作为命令的参数，以“${VAR}”方式引用， 其中“VAR”为变量名。如果指定的变量未被定义，那么该引用会被一个空字符串代替，否则会被变量的值替换。这个替换实在引号变量展开前进行的，因此那些带有分号的变量会被分割成0个或多个不带引号的变量，取代之前的位置，例如：

``` cmake
set(Foo a b c)    # 3 unquoted args -> value is "a;b;c"
command(${Foo})   # unquoted arg replaced by a;b;c
                  # and expands to three arguments
command("${Foo}") # quoted arg value is "a;b;c"
set(Foo "")       # 1 quoted arg -> value is empty string
command(${Foo})   # unquoted arg replaced by empty string
                  # and expands to zero arguments
command("${Foo}") # quoted arg value is empty string
```

System environment variables and Windows registry values can be accessed directly in CMake. To access system environment variables, use the syntax $ENV{VAR}. CMake can also reference registry entries in many commands using a syntax of the form [HKEY_CURRENT_USER\\Software\\path1\\path2;key], where the paths are built from the registry tree and key.

在CMake中可以直接访问系统变量和Windows注册表值。使用“$ENV{VAR}”方式访问系统环境变量。CMake也可以使用“[HKEY_CURRENT_USER\\Software\\path1\\path2;key]”的方式访问注册表，path和Key由注册表path和Key构成。

### Variables Scope

Variables in CMake have a scope that is a little different from most languages. When you set a variable, it is visible to the current CMakeLists file or function and any subdirectory’s CMakeLists files, any functions or macros that are invoked, and any files that are included using the include command. When a new subdirectory is processed (or a function called), a new variable scope is created and initialized with the current value of all variables in the calling scope. Any new variables created in the child scope, or changes made to existing variables, will not impact the parent scope. Consider the following example:
CMake中的变量作用类型和大多数语言都有一些不一样。如果设置了一个变量，那么他对当前CMakeLists文件或者函数， 子目录的CMakeLists文件，或者任意调用的函数和宏，以及任何通过include命令包含进来的文件都是可见的。但一个新的目录被处理或者函数被调用，一个新的变量域被创建，并以所有变量当前的值看哦买吃哦买和初始化。子作用域中新创建的变量以及对现有变量的值的修改对父作用域都是不可见的。思考下面的例子：

``` cmake
function(foo)
  message(${test}) # test is 1 here
  set(test 2)
  message(${test}) # test is 2 here, but only in this scope
endfunction()

set(test 1)
foo()
message(${test}) # test will still be 1 here
```

In some cases, you might want a function or subdirectory to set a variable in its parent’s scope. There is a way for CMake to return a value from a function, and it can be done by using the PARENT_SCOPE option with the set command. We can modify the prior example so that the function foo changes the value of test in its parent’s scope as follows:
一些情况下，您可能需要一个函数或者子目录在父作用域中设置一个变量。有一种方法可以让CMake从函数中返回一个值，可以通过set命令的PARENT_SCOPE选项实现。我们可以像下面这样修改上面的示例代码，让foo函数可以修改父作用域中的test变量，

``` cmake
function(foo)
  message(${test}) # test is 1 here
  set(test 2 PARENT_SCOPE)
  message(${test}) # test still 1 in this scope
endfunction()

set(test 1)
foo()
message(${test}) # test will now be 2 here
```

Variables in CMake are defined in the order of the execution of set commands.
变量在CMake中以set命令执行顺序来定义的，考虑下面这个例子，
Consider the following example:

``` cmake

# FOO is undefined

set(FOO 1)
# FOO is now set to 1

set(FOO 0)
# FOO is now set to 0
```

To understand the scope of variables, consider this example:
为了理解变量的作用域，参考下面的例子，

``` cmake
set(foo 1)

# process the dir1 subdirectory
add_subdirectory(dir1)

# include and process the commands in file1.cmake
include(file1.cmake)

set(bar 2)
# process the dir2 subdirectory
add_subdirectory(dir2)

# include and process the commands in file2.cmake
include(file2.cmake)
```

In this example, because the variable foo is defined at the beginning, it will be defined while processing both dir1 and dir2. In contrast, bar will only be defined when processing dir2. Likewise, foo will be defined when processing both file1.cmake and file2.cmake, whereas bar will only be defined while processing file2.cmake.

在这个例子中，因为变量foo是在开头定义的，所以在处理dir1和dir2的时候都被定义了。相反的，bar只有在处理dir2的时候才定义。同样的，foo在处理file1.cmake和file2.cmake时都有定义，而bar只有在处理file2.cmake时才定义。

## Commands

A command consists of the command name, opening parenthesis, whitespace separated arguments, and a closing parenthesis. Each command is evaluated in the order that it appears in the CMakeLists file. See the cmake-commands manual for a full list of CMake commands.

命令有命令名称，左括号，空格分割的参数，以及右括号组成。命令执行顺序由它们在CMakeLists文件中的顺序决定。参考cmake-commands手册获取CMake命令的完整列表。

CMake is no longer case sensitive to command names, so where you see command, you could use COMMAND or Command instead. It is considered best practice to use lowercase commands. All whitespace (spaces, line feeds, tabs) is ignored except to separate arguments. Therefore, commands may span multiple lines as long as the command name and the opening parenthesis are on the same line.

CMake中命令名称不再大小写敏感的，当使用command的时候，可以用COMMAND或者Command代替。命令采用小写可以考虑作为最佳实践。所有的空白（空格、行末、tab）在分割参数外都被忽略掉。因此，命令可以占用多行，只要命令名称和左括号在同一行即可。

CMake command arguments are space separated and case sensitive. Command arguments may be either quoted or unquoted. A quoted argument starts and ends in a double quote (“) and always represents exactly one argument. Any double quotes contained inside the value must be escaped with a backslash. Consider using bracket arguments for arguments that require escaping, see the cmake-language manual. An unquoted argument starts in any character other than a double quote (later double quotes are literal) and is automatically expanded into zero-or-more arguments by separating on semicolons within the value.

CMake 命令参数是由空白分割的，且对大小写敏感。命令参数可以带引号也可以不带。带引号的参数的开始和结尾都有一个双引号，作为一个参数。在双引号见的双引号需要由反斜杠转义，可以参考<cmake-language>手册。对需要转义的参数可以使用中括号括起来。不带引号的参数可以以除引号外的任意字符开始，自动展开为0个或多个以分号分隔的参数。

For example:

``` cmake
command("")          # 1 quoted argument
command("a b c")     # 1 quoted argument
command("a;b;c")     # 1 quoted argument
command("a" "b" "c") # 3 quoted arguments
command(a b c)       # 3 unquoted arguments
command(a;b;c)       # 1 unquoted argument expands to 3
```

### Basic Commands

As we saw earlier in the, the set and command:unset commands explicitly set or unset variables. The string, list, and separate_arguments commands offer basic manipulation of strings and lists.

如我们之前看到的set和unset命令用于设置和去设置变量。字符串、列表和分隔参数命令提供了对字符串和列表的基本操作。

The add_executable and add_library commands are the main commands for defining the executables and libraries to build, and which source files comprise them. For Visual Studio projects, the source files will show up in the IDE as usual, but any header files the project uses will not be. To have the header files show up, simply add them to the list of source files for the executable or library; this can be done for all generators. Any generators that do not use the header files directly (such as Makefile based generators) will simply ignore them.

add_executable 和add_library命令是定义编译可执行文件和库的主要命令，以及是有哪些源文件构成。对于Visual Studio工程，IDE中会展示源文件，但头文件不会显示在IDE中。为了在IDE中显示头文件，可以将头文件列表加到源文件列表里面。这对所有的生成器都有效。任何不直接使用头文件的生成器（比如基于Makefile的生成器）都会简单的忽略他们。

## Flow Control

The CMake language provides three flow control constructs to help organize your CMakeLists files and keep them maintainable.

- Conditional statements (e.g. if)
- Looping constructs (e.g. foreach and while)
- Procedure definitions (e.g. macro and function)

CMake语言提供了3种流程控制的结构来帮助组织CMakeLists文件并使得他们可维护：

- 条件表达式（比如if）
- 循环结构（比如，foreach和while)
- 过程定义（比如宏和函数）

### Conditional Statements

First we will consider the if command. In many ways, the if command in CMake is just like the if command in any other language. It evaluates its expression and uses it to execute the code in its body or optionally the code in the else clause.

首先我们考虑if命令。在很多情况下，CMake中的if命令和其他语言中的if命令过一样。它首先计算表达式的值，使用这个值去计算if 语句中的代码，或者执行else代码。

For example:

``` cmake
if(FOO)
  # do something here
else()
  # do something else
endif()
```

CMake also supports elseif to help sequentially test for multiple conditions.

CMake以提供了elseif表达式来顺序测试多个条件。

For example:

``` cmake
if(MSVC80)
  # do something here
elseif(MSVC90)
  # do something else
elseif(APPLE)
  # do something else
endif()
```

The if command documents the many conditions it can test.

if命令记录了if可以测试的多个表达式。

### Looping Constructs

The foreach and while commands allow you to handle repetitive tasks that occur in sequence. The break command breaks out of a foreach or while loop before it would normally end.

foreach和while命令允许你处理顺序发生的重复任务。break命令在foreach和while命令正常结束前打断他们。

The foreach command enables you to execute a group of CMake commands repeatedly on the members of a list. Consider the following example adapted from VTK

foreache命令使您能够是有列表中的成员执行一组重复的命令，思考下面从VTK中的示例，

``` cmake
foreach(tfile
        TestAnisotropicDiffusion2D
        TestButterworthLowPass
        TestButterworthHighPass
        TestCityBlockDistance
        TestConvolve
        )
  add_test(${tfile}-image ${VTK_EXECUTABLE}
    ${VTK_SOURCE_DIR}/Tests/rtImageTest.tcl
    ${VTK_SOURCE_DIR}/Tests/${tfile}.tcl
    -D ${VTK_DATA_ROOT}
    -V Baseline/Imaging/${tfile}.png
    -A ${VTK_SOURCE_DIR}/Wrapping/Tcl
    )
endforeach()
```

The first argument of the foreach command is the name of the variable that will take on a different value with each iteration of the loop; the remaining arguments are the list of values over which to loop. In this example, the body of the foreach loop is just one CMake command, add_test. In the body of the foreach, each time the loop variable (tfile in this example) is referenced will be replaced with the current value from the list. In the first iteration, occurrences of ${tfile} will be replaced with TestAnisotropicDiffusion2D. In the next iteration, ${tfile} will be replaced with TestButterworthLowPass. The foreach loop will continue to loop until all of the arguments have been processed.

foreach命令第一个参数时用于代替不同之完成循环中每次迭代的变量名。剩下的参数时循环时的列表元素。在这个例子中，foreach循环自信的主体只有一个命令add_test。 在foreach的主体中，每次循环的变量（本例子中是tfile）都会被列表中的当前值替换。在第一次迭代中，${tfile}出现的地方会被TestAnisotropicDiffusion2D替换，第二次\${tfile}TestButterworthLowPass替换。foreach在列表中的所有元素都被处理完后才会停止迭代。

It is worth mentioning that foreach loops can be nested, and that the loop variable is replaced prior to any other variable expansion. This means that in the body of a foreach loop, you can construct variable names using the loop variable. In the code below, the loop variable tfile is expanded, and then concatenated with _TEST_RESULT. The new variable name is then expanded and tested to see if it matches FAILED.

值得一提的是，foreach循环可以是级联的，循环变量的替换在其他变量展开之前。这意味着在foreach循环的主体部分，您可以用循环变量名来构建变量。在下面的代码中，循环变量tfile被展开，连接上_TEST_RESULT。新变量名被展开，测试是否和FAILED相匹配。

``` cmake
if(${${tfile}}_TEST_RESULT} MATCHES FAILED)
  message("Test ${tfile} failed.")
endif()
```

The while command provides looping based on a test condition. The format for the test expression in the while command is the same as it is for the if command, as described earlier. Consider the following example, which is used by CTest. Note that CTest updates the value of CTEST_ELAPSED_TIME internally.

while命令提供一种基于测试条件的循环。while命令中测试表达式的格式和之前提到的if命令相同。考虑下面CTest中使用的日子，注意，CTest内部会更新CTEST_ELAPSED_TIME的值。

``` cmake
#####################################################
# run paraview and ctest test dashboards for 6 hours
#
while(${CTEST_ELAPSED_TIME} LESS 36000)
  set(START_TIME ${CTEST_ELAPSED_TIME})
  ctest_run_script("dash1_ParaView_vs71continuous.cmake")
  ctest_run_script("dash1_cmake_vs71continuous.cmake")
endwhile()
```

### Procedure Definitions

The macro and  function commands support repetitive tasks that may be scattered throughout your CMakeLists files. Once a macro or function is defined, it can be used by any CMakeLists files processed after its definition.

宏和函数命令支持可能充斥在您的CMakeLists文件中的重复任务。一旦一个宏或者函数被定义了，它就可以被CMakeLists文件在定义之后的部分使用。

A function in CMake is very much like a function in C or C++. You can pass arguments into it, and they become variables within the function. Likewise, some standard variables such as ARGC, ARGV, ARGN, and ARGV0, ARGV1, etc. are defined. Function calls have a dynamic scope. Within a function you are in a new variable scope; this is like how you drop into a subdirectory using the add_subdirectory command and are in a new variable scope. All the variables that were defined when the function was called remain defined, but any changes to variables or new variables only exist within the function. When the function returns, those variables will go away. Put more simply: when you invoke a function, a new variable scope is pushed; when it returns, that variable scope is popped.

CMake中的函数和C、C++ 中的函数很相似。您可以给函数传递参数，它们会成为函数中的变量。同样的，也定义了一些标准变量如ARGC、ARGV、ARGN以及ARGV0、ARGV1等。函数调用有动态作用域。在函数中，您在一个新的变量作用域中。这和您调用add_subdirectory命令陷入子目录一样，在一个新的作用域中。所有在调用时定义的变量同样被定义了，但所有对变量的改变后者新的变量都只存在于函数作用域中。当函数返回是，这些变量都消失了。更简单的说，但你调用一个函数是，一个新的函数作用域产生，但返回时，之前的作用域被弹出。

The function command defines a new function. The first argument is the name of the function to define; all additional arguments are formal parameters to the function.
function命令用于定义一个新函数。第一个参数时函数名，剩下的参数时函数的参数。

``` cmake
function(DetermineTime _time)
  # pass the result up to whatever invoked this
  set(${_time} "1:23:45" PARENT_SCOPE)
endfunction()

# now use the function we just defined
DetermineTime(current_time)

if(DEFINED current_time)
  message(STATUS "The time is now: ${current_time}")
endif()
```

Note that in this example, _time is used to pass the name of the return variable. The set command is invoked with the value of _time, which will be current_time. Finally, the set command uses the PARENT_SCOPE option to set the variable in the caller’s scope instead of the local scope.

注意在这个例子中，_time被用于传递函数的返回值。set函数调用时传递了_time的值，即current_time.最后，set函数使用了PARENT_SCOPE选项，在函数调用者的作用域下设置了一个变量，而不是在本地作用域下。

Macros are defined and called in the same manner as functions. The main differences are that a macro does not push and pop a new variable scope, and that the arguments to a macro are not treated as variables but as strings replaced prior to execution. This is very much like the differences between a macro and a function in C or C++. The first argument is the name of the macro to create; all additional arguments are formal parameters to the macro.

宏的定义和调用方式和函数是一致的。主要区别在与宏不会push/pop变量作用域，传递给宏的参数不会被认为是变量，而是在执行前替换的字符串。这和C、C++中宏和函数的区别很像。 macro命令的第一个参数时宏名，其他参数时传递给宏的参数。

``` cmake
# define a simple macro
macro(assert TEST COMMENT)
  if(NOT ${TEST})
    message("Assertion failed: ${COMMENT}")
  endif()
endmacro()

# use the macro
find_library(FOO_LIB foo /usr/local/lib)
assert(${FOO_LIB} "Unable to find library foo")
```

The simple example above creates a macro called assert. The macro is defined into two arguments; the first is a value to test and the second is a comment to print out if the test fails. The body of the macro is a simple if command with a message command inside of it. The macro body ends when the endmacro command is found. The macro can be invoked simply by using its name as if it were a command. In the above example, if FOO_LIB was not found then a message would be displayed indicating the error condition.

上面这个简单的例子定义了一个名为assert的宏。它有两个参数，第一个为被测试的值，第二个参数时测试失败时打印的信息。宏的内部是一个简单的主体为message命令的if命令。宏的主题在endmacro命令时结束。宏可以简单的像命令一样通过名称调用。在上面的例子中，如果FOO_LIB没有被找到，一条消息会显示出来来提示这种错误。

The macro command also supports defining macros that take variable argument lists. This can be useful if you want to define a macro that has optional arguments or multiple signatures. Variable arguments can be referenced using ARGC and ARGV0, ARGV1, etc., instead of the formal parameters. ARGV0 represents the first argument to the macro; ARGV1 represents the next, and so forth. You can also use a mixture of formal arguments and variable arguments, as shown in the example below.

macro命令也可以用于定义可变参数宏。这个在定义有可选参数或者多种签名的宏时特别有用。可以通过ARGC、ARGV0、ARGV1这样来使用变量参数。和正式的参数不同，ARGV0表示第一个参数，ARGV1指代下一个，这样，您可以混合使用真是参数和可变参数。就像下面的例子这样

``` cmake
# define a macro that takes at least two arguments
# (the formal arguments) plus an optional third argument
macro(assert TEST COMMENT)
  if(NOT ${TEST})
    message("Assertion failed: ${COMMENT}")

    # if called with three arguments then also write the
    # message to a file specified as the third argument
    if(${ARGC} MATCHES 3)
      file(APPEND ${ARGV2} "Assertion failed: ${COMMENT}")
    endif()

  endif()
endmacro()

# use the macro
find_library(FOO_LIB foo /usr/local/lib)
assert(${FOO_LIB} "Unable to find library foo")
```

In this example, the two required arguments are TEST and COMMENT. These required arguments can be referenced by name, as they are in this example, or by referencing ARGV0 and ARGV1. If you want to process the arguments as a list, use the ARGV and ARGN variables. ARGV (as opposed to ARGV0, ARGV1, etc) is a list of all the arguments to the macro, while ARGN is a list of all the arguments after the formal arguments. Inside your macro, you can use the foreach command to iterate over ARGV or ARGN as desired.

在这个例子中，两个必选的参数时TEST和COMMENT。这些参数可以通过名称应用，或者通过ARGV0、ARGV1.如果想像列表一样处理这些参数，使用变量ARGV和ARGN。ARGV（和ARGV0、ARGV1这些相对）是所有传给宏的参数列表，ARGN是真实参数后的参数列表。在宏中，可以根据需要用foreach命令来迭代ARGV和ARGN。

The return command returns from a function, directory or file. Note that a macro, unlike a function, is expanded in place and therefore cannot handle return.
return命令从函数、目录或者文件返回。注意，宏和函数不一样，是原地展开的，因此不能处理return。

## Regular Expressions

A few CMake commands, such as if and string, make use of regular expressions or can take a regular expression as an argument. In its simplest form, a regular expression is a sequence of characters used to search for exact character matches. However, many times the exact sequence to be found is unknown, or only a match at the beginning or end of a string is desired. Since there are several different conventions for specifying regular expressions, CMake’s standard is described in the string command documentation. The description is based on the open source regular expression class from Texas Instruments, which is used by CMake for parsing regular expressions.

一些CMake的命令，比如if和string，可以使用正则表达式，或者使用正则表达式作为参数。最简单的形式下，正则表达式是一串字符用于查找精确的字符匹配。但是精确的拼配次数是不可知的，或者仅仅需要文件的开头或者结尾匹配。因为存在几种不同的表达方式来指定正则表达式，CMake的标准定义在string命令文档中。基于TI的开源睁着表达式类，这被CMake用于解析正则表达式。

## Advanced Commands

There are a few commands that can be very useful, but are not typically used in writing CMakeLists files. This section will discuss a few of these commands and when they are useful.

有一些在写CMake Lists文件时不常见但非常有用的命令。本章将讨论其中的一些以及何时使用。

First, consider the add_dependencies command which creates a dependency between two targets. CMake automatically creates dependencies between targets when it can determine them. For example, CMake will automatically create a dependency for an executable target that depends on a library target. The add_dependencies command command is typically used to specify inter-target dependencies between targets where at least one of the targets is a custom target (see Add Custom Command section).

首先是add_dependencies命令，它用于在两个目标见构建一种依赖关系。CMake在可以确定的时候自动在目标间创建依赖关系。例如，CMake会自动在可执行目标和依赖的库之间创建以来关系。add_dependencies典型用法是制定目标间的依赖关系，这些目标中至少有一个是自定义的目标（参考自定义命令章节）。

The include_regular_expression command also relates to dependencies. This command controls the regular expression that is used for tracing source code dependencies. By default, CMake will trace all the dependencies for a source file including system files such as stdio.h. If you specify a regular expression with the include_regular_expression command, that regular expression will be used to limit which include files are processed. For example; if your software project’s include files all started with the prefix foo (e.g. fooMain.c fooStruct.h, etc), you could specify a regular expression of ^foo.*$ to limit the dependency checking to just the files of your project.

include_regular_expression命令也和依赖关系相关。这个命令控制用于追踪源码依赖的正则表达式。默认情况下，CMake会最终所有的源码的依赖关系，包括向stdio.h这样的系统文件。如果通过include_regular_expression命令指定了一个正则表达式，这个正则表达式可以被用来限制哪些包含的文件被处理。例如，你项目包含的文件都是以前缀foo开头（比如 fooMain.c fooStructure.h等），可以指定正则表达式^foo.*$，来限定依赖检查只包含项目中的文件。