# cmake构建系统

备忘

author : 区长


## cmake常用指令

 - 参考链接 [CMake Practice](http://sewm.pku.edu.cn/src/paradise/reference/CMake Practice.pdf)
 - 参考链接 [cmake 手册详解](http://www.cnblogs.com/coderfenghc/tag/cmake/)

## CMAKE_MINIMUM_REQUIRED指令

### 指令语法格式

```
CMAKE_MINIMUM_REQUIRED(VERSION versionNumber [FATAL_ERROR])
```

### 作用

指定cmake的最小版本

### 例子

```
CMAKE_MINIMUM_REQUIRED(VERSION 2.5 FATAL_ERROR)
```

如果cmake小于2.5版本，则出现严重错误，整个过程中止。



## PROJECT指令

### 指令语法格式

```
PROJECT(projectname [CXX] [C] [Java])
```

### 作用

定义工程名称，并可指定工程支持的语言。

支持的语言列表是可以忽略的，默认表示支持所有语言。

### 例子

```
PROJECT(DEMO)
```

隐含的两个变量

```
<projectname>_BINARY_DIR 和 <projectname>_SOURCE_DIR
```

上面的例子就是

```
DEMO_BINARY_DIR 和 DEMO_SOURCE_DIR
```

以上两个变量等价于以下两个变量

```
PROJECT_BINARY_DIR 和 PROJECT_SOURCE_DIR
```

## SET指令

### 指令语法格式

```
SET(VAR [VALUE] [CACHE TYPE DOCSTRING [FORCE]])
```

### 作用

显示的定义变量

### 例子

```
SET(SRC_LIST main.c test1.c test2.c)
```

表示定义SRC_LIST变量，其值为一系列的源文件

## MESSAGE指令

### 指令语法格式

```
MESSAGE([SEND_ERROR | STATUS | FATAL_ERROR] "message to display"
...)
```

### 作用

向终端输入用户定义的信息，分三种类型

 - SEND_ERROR，产生错误，生成过程被跳过。
 - SATUS，输出前缀为—的信息。
 - FATAL_ERROR，立即终止所有 cmake 过程.

### 例子

```
MESSAGE(STATUS "This is BINARY dir " ${PROJECT_BINARY_DIR})
MESSAGE(STATUS "This is SOURCE dir "${PROJECT_SOURCE_DIR})
```

## ADD_EXECUTABLE指令

### 指令语法格式

```
ADD_EXECUTABLE(executable_name source1 source2 ... sourceN)
```

### 例子

```
ADD_EXECUTABLE(demo main.c test1.c test2.c)
```

或

```
ADD_EXECUTABLE(demo ${SRC_LIST})
```

## ADD_LIBRARY指令

### 指令语法格式

```
ADD_LIBRARY(libname [SHARED|STATIC|MODULE]
 [EXCLUDE_FROM_ALL]
 source1 source2 ... sourceN)
```

### 参数说明

 - SHARED，动态库
 - STATIC，静态库
 - MODULE，在使用dyld的系统有效，如果不支持dyld，则被当作SHARED对待
 - EXCLUDE_FROM_ALL，表示这个库不会被默认构建，除非有其他的组件依赖或者手动构建

### 作用

生成静态库和动态库等

### 添加静态库

```
ADD_LIBRARY(demo_static STATIC ${SRC_LIST}
```

编译产生libdemo_static.a静态库

### 重命名库文件名称

将编译产生的libdemo_static.a静态库重命名为libdemo.a

```
SET_TARGET_PROPERTIES(demo PROPERTIES CLEAN_DIRECT_OUTPUT 1)
SET_TARGET_PROPERTIES(demo_static PROPERTIES OUTPUT_NAME "demo")
```

### 静态库安装

 静态库安装需要使用ARCHIVE关键字


### 添加动态库

```
ADD_LIBRARY(demo_static SHARED ${SRC_LIST}
```

### 动态库版本号

```
libdemo.so.1.2
libdemo.so ->libdemo.so.1
libdemo.so.1->libdemo.so.1.2
```

```
SET_TARGET_PROPERTIES(demo PROPERTIES VERSION 1.2 SOVERSION 1)
```

VERSION指代动态库版本，SOVERSION指代API版本。


## SET_TARGET_PROPERTIES指令

### 指令语法格式

```
SET_TARGET_PROPERTIES(target1 target2 ...
 PROPERTIES prop1 value1
 prop2 value2 ...)
```

### 指令作用

用来设置输出的名称，对于动态库，还可以用来指定动态库版本和API版本

### 例子

```
SET_TARGET_PROPERTIES(hello_static PROPERTIES OUTPUT_NAME "hello")
```

## GET_TARGET_PROPERTY指令

### 指令语法格式

```
GET_TARGET_PROPERTY(VAR target property)
```

### 指令作用

获取SET_TARGET_PROPERTIES指令设置的属性

### 例子

```
GET_TARGET_PROPERTY(OUTPUT_VALUE hello_static OUTPUT_NAME)
MESSAGE(STATUS “This is the hello_static OUTPUT_NAME:”${OUTPUT_VALUE})
```

## ADD_DEFINITIONS指令

### 作用

向C/C++编译器添加-D定义

### 例子

```
ADD_DEFINITIONS(-DENABLE_DEBUG -DABC)
```

如果你的代码中定义了#ifdef ENABLE_DEBUG #endif，这个代码块就会生效。


## ADD_DEPENDENCIES指令

### 指令语法格式

```
ADD_DEPENDENCIES(target-name depend-target1
 depend-target2 ...)
```

### 作用

定义target依赖的其他target，确保在编译本target之前，其他的target已经被构建。


## ADD_SUBDIRECTORY指令

### 指令语法格式

```
ADD_SUBDIRECTORY(source_dir [binary_dir] [EXCLUDE_FROM_ALL])
```

### 作用

用于向当前工程添加存放源文件的子目录，并可以指定中间二进制和目标二进制存放的位置，EXCLUDE_FROM_ALL 参数的含义是将这个目录从编译过程中排除。

### 例子

```
ADD_SUBDIRECTORY(src bin)
```

将src子目录加入工程，并指定编译输出(包含编译中间结果)路径为bin目录。如果不进行 bin 目录的指定，那么编译结果(包括中间结果)都将存放在src目录，指定bin目录后，所有的中间结果和目标二进制都将存放在bin目录。

### 换个地方保存目标二进制

```
SET(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
SET(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)
```

通过SET指令重新定义EXECUTABLE_OUTPUT_PATH和LIBRARY_OUTPUT_PATH变量来指定最终的目标二进制的位置

以上命令指定后，可执行二进制的输出路径为bin和库的输出路径为lib

## 外部构建

 - 1、首先创建外部构建目录，如build
 - 2、进入build目录，执行cmake path(path指向CMakeLists.txt文件所在目录)，如cmake ..，父目录存在我们需要的CMakeLists.txt，查看一下 build 目录，就会发现了生成了编译需要的 Makefile 以及其他的中间文件
 - 3、在build目录执行make命令进行构建
 - 优点：对原有的工程没有任何影响，所有动作全部发生在编译目录
 - 对于外部构建PROJECT_SOURCE_DIR仍然指工程路径，但是PROJECT_BINARY_DIR则指代编译路径，即build目录路径


## CMAKE_INSTALL_PREFIX变量

### 例子

```
cmake -DCMAKE_INSTALL_PREFIX=/usr ..
```

### 作用

指定安装前缀目录

## INSTALL指令

### 语法格式

```
INSTALL(TARGETS targets...
 [[ARCHIVE|LIBRARY|RUNTIME]
 [DESTINATION <dir>]
 [PERMISSIONS permissions...]
 [CONFIGURATIONS
 [Debug|Release|...]]
 [COMPONENT <component>]
 [OPTIONAL]
 ] [...])
```

### 参数说明

 - TARGETS 后面跟通过ADD_EXECUTABLE或者ADD_LIBRARY定义的目标文件，如可执行二进制、动态库、静态库
 - 目标类型有三种，ARCHIVE指静态库，LIBRARY指动态库，RUNTIME指可执行目标二进制。
 - DESTINATION定义安装的路径，如果路径以/开头，那么指的是绝对路径，这时候CMAKE_INSTALL_PREFIX就无效了。如果使用CMAKE_INSTALL_PREFIX来定义安装路径，需写成相对路径，不要以/开头。安装后的路径就是以${CMAKE_INSTALL_PREFIX}/DESTINATION定义的路径

### 作用

定义安装规则，安装的内容可以包括目标二进制、动态库、静态库以及文件、目录、脚本等

### 例子

```
INSTALL(TARGETS myrun mylib mystaticlib
RUNTIME DESTINATION bin CONFIGURATIONS Release
LIBRARY DESTINATION lib CONFIGURATIONS Release
ARCHIVE DESTINATION libstatic CONFIGURATIONS Release
)
```

 - Release构建的可执行二进制myrun安装到${CMAKE_INSTALL_PREFIX}/bin目录，
 - Release构建的动态库libmylib安装到${CMAKE_INSTALL_PREFIX}/lib 目录
 - Release构建的静态库libmystaticlib安装到${CMAKE_INSTALL_PREFIX}/libstatic目录

### 普通文件的安装

```
INSTALL(FILES files... DESTINATION <dir>
 [PERMISSIONS permissions...]
 [CONFIGURATIONS [Debug|Release|...]]
 [COMPONENT <component>]
 [RENAME <name>] [OPTIONAL])
```

用于安装一般文件，并可以指定访问权限，文件名是此指令所在路径下的相对路径。如果默认不定义权限PERMISSIONS，安装后的权限为：OWNER_WRITE, OWNER_READ, GROUP_READ,和 WORLD_READ，即 644 权限。


### 非目标文件的可执行程序安装（如shell脚本）

```
INSTALL(PROGRAMS files... DESTINATION <dir>
 [PERMISSIONS permissions...]
 [CONFIGURATIONS [Debug|Release|...]]
 [COMPONENT <component>]
 [RENAME <name>] [OPTIONAL])
```

与上面的FILES指令使用方法一样，唯一的不同是安装后权限为:OWNER_EXECUTE, GROUP_EXECUTE, 和 WORLD_EXECUTE，即 755 权限

### 目录的安装

```
INSTALL(DIRECTORY dirs... DESTINATION <dir>
 [FILE_PERMISSIONS permissions...]
 [DIRECTORY_PERMISSIONS permissions...]
 [USE_SOURCE_PERMISSIONS]
 [CONFIGURATIONS [Debug|Release|...]]
 [COMPONENT <component>]
 [[PATTERN <pattern> | REGEX <regex>][EXCLUDE] 
 [PERMISSIONS permissions...]] [...])
```

 - DIRECTORY后面连接的是所在Source目录的相对路径，如果目录名不以/结尾，那么这个目录将被安装为目标路径下，如果以/结尾，代表将这个目录中的内容安装到目标路径，但不包括这个目录本身。
 - PATTERN用于使用正则表达式进行过滤
 - PERMISSIONS用于指定PATTERN过滤后的文件权限。

### 例子

```
INSTALL(DIRECTORY icons scripts/ DESTINATION share/myproj
 PATTERN "CVS" EXCLUDE
 PATTERN "scripts/*"
 PERMISSIONS OWNER_EXECUTE OWNER_WRITE OWNER_READ
 GROUP_EXECUTE GROUP_READ)
```

 - 将icons目录安装到 <prefix>/share/myproj
 - 将scripts/中的内容安装到<prefix>/share/myproj
 - 不包含目录名为CVS的目录
 - 对于scripts/*文件指定权限为OWNER_EXECUTE OWNER_WRITE OWNER_READ GROUP_EXECUTE GROUP_READ

### 安装时cmake脚本的执行

```
INSTALL([[SCRIPT <file>] [CODE <code>]] [...])
```

 - SCRIPT参数用于在安装时调用cmake脚本文件（也就是<abc>.cmake文件）
 - CODE参数用于执行CMAKE指令，必须以双引号括起来。

```
INSTALL(CODE "MESSAGE(\"Sample install message.\")")
```

### 安装

```
cmake -DCMAKE_INSTALL_PREFIX=/tmp/test/usr ..
make
make install
```

进入/tmp/test/usr目录，即可看到安装结果。CMAKE_INSTALL_PREFIX的默认定义是/usr/local


## INCLUDE_DIRECTORIES指令

### 指令语法格式

```
INCLUDE_DIRECTORIES([AFTER|BEFORE] [SYSTEM] dir1 dir2 ...)
```

### 作用

用来向工程添加多个特定的头文件搜索路径，路径之间用空格分割，如果路径中包含了空格，可以使用双引号将它括起来，默认的行为是追加到当前的头文件搜索路径的后面，你可以通过两种方式来进行控制搜索路径添加的方式：

 - CMAKE_INCLUDE_DIRECTORIES_BEFORE，通过SET这个cmake变量为on，可以将添加的头文件搜索路径放在已有路径的前面。
 - 通过AFTER或者BEFORE参数，也可以控制是追加还是置前

### 例子

```
INCLUDE_DIRECTORIES(/usr/include/demo)
```

## LINK_DIRECTORIES 指令

### 指令语法格式

```
LINK_DIRECTORIES(directory1 directory2 ...）
```

### 作用

添加非标准的共享库搜索路径，比如在工程内部同时存在共享库和可执行二进制，在编译时就需要指定一下这些共享库的路径

## TARGET_LINK_LIBRARIES 指令

### 指令语法格式

```
TARGET_LINK_LIBRARIES(target library1
 <debug | optimized> library2
 ...)
```

### 作用

为target添加需要链接的共享库

### 例子

```
TARGET_LINK_LIBRARIES(main demo)
```

或


```
TARGET_LINK_LIBRARIES(main libdemo.so)
```

### 链接到静态库

```
TARGET_LINK_LIBRARIES(main libdemo.a)
```

## FIND_FILE指令

### 指令语法格式

```
FIND_FILE(<VAR> name1 path1 path2 ...)
```

VAR变量代表找到的文件全路径，包含文件名

## FIND_LIBRARY指令

### 指令语法格式

```
FIND_LIBRARY(<VAR> name1 path1 path2 ...)
```

VAR变量表示找到的库全路径，包含库文件名

### 例子

```
FIND_LIBRARY(libX X11 /usr/lib)
IF(NOT libX)
MESSAGE(FATAL_ERROR “libX not found”)
ENDIF(NOT libX)
```

## FIND_PATH指令

### 指令语法格式

```
FIND_PATH(<VAR> name1 path1 path2 ...)
```

VAR 变量代表包含这个文件的路径。

## FIND_PROGRAM指令

### 指令语法格式

```
FIND_PROGRAM(<VAR> name1 path1 path2 ...)
```

VAR变量代表包含这个程序的全路径。

## FIND_PACKAGE指令

### 指令语法格式

```
FIND_PACKAGE(<name> [major.minor] [QUIET] [NO_MODULE]
 [[REQUIRED|COMPONENTS] [componets...]])
```

## ADD_TEST指令和ENABLE_TESTING指令

### 指令语法格式

```
ADD_TEST(testname Exename arg1 arg2 ...)
ENABLE_TESTING()
```

### 作用

ENABLE_TESTING指令用来控制Makefile是否构建test目标

### 例子

```
ADD_TEST(mytest ${PROJECT_BINARY_DIR}/bin/demo)
ENABLE_TESTING()
```

## AUX_SOURCE_DIRECTORY指令

### 指令语法格式

```
AUX_SOURCE_DIRECTORY(dir VARIABLE)
```

### 作用

发现一个目录下所有的源代码文件并将列表存储在一个变量中，这个指令临时被用来
自动构建源文件列表。因为目前cmake还不能自动发现新添加的源文件。

### 例子

```
AUX_SOURCE_DIRECTORY(. SRC_LIST)
ADD_EXECUTABLE(main ${SRC_LIST})
```

## EXEC_PROGRAM指令

### 指令语法格式

```
EXEC_PROGRAM(Executable [directory in which to run]
 [ARGS <arguments to executable>]
 [OUTPUT_VARIABLE <var>]
 [RETURN_VALUE <var>])
```

### 作用

用于在指定的目录运行某个程序，通过ARGS添加参数，如果要获取输出和返回值，可通过OUTPUT_VARIABLE和 RETURN_VALUE分别定义两个变量

### 例子

```
EXEC_PROGRAM(ls ARGS "*.c" OUTPUT_VARIABLE LS_OUTPUT RETURN_VALUE LS_RVALUE)
IF(not LS_RVALUE)
MESSAGE(STATUS "ls result: " ${LS_OUTPUT})
ENDIF(not LS_RVALUE)
```

## FILE 指令

### 指令语法格式

```
FILE(WRITE filename "message to write"... )
FILE(APPEND filename "message to write"... )
FILE(READ filename variable)
FILE(GLOB variable [RELATIVE path] [globbing expressions]...)
FILE(GLOB_RECURSE variable [RELATIVE path] [globbing expressions]...)
FILE(REMOVE [directory]...)
FILE(REMOVE_RECURSE [directory]...)
FILE(MAKE_DIRECTORY [directory]...)
FILE(RELATIVE_PATH variable directory file)
FILE(TO_CMAKE_PATH path result)
FILE(TO_NATIVE_PATH path result)
```

## INCLUDE指令

### 指令语法格式

```
INCLUDE(file1 [OPTIONAL])
INCLUDE(module [OPTIONAL])
```

OPTIONAL参数的作用是文件不存在也不会产生错误。

### 作用

用来载入CMakeLists.txt文件，也用于载入预定义的cmake模块


## 特殊的环境变量CMAKE_INCLUDE_PATH和CMAKE_LIBRARY_PATH

如果头文件没有存放在常规路径(/usr/include, /usr/local/include 等)，则可以通过这些变量就行弥补。

```
export CMAKE_INCLUDE_PATH=/usr/include/demo

```



## cmake常用变量和常用环境变量

### 变量引用方式

使用${}进行变量的引用。在IF等语句中，是直接使用变量名而不通过${}取值

### 自定义变量的方式

通过SET指令进行设置

### cmake常用变量

```
CMAKE_BINARY_DIR、PROJECT_BINARY_DIR、<projectname>_BINARY_DIR
```

```
CMAKE_SOURCE_DIR、PROJECT_SOURCE_DIR、<projectname>_SOURCE_DIR
```

```
CMAKE_CURRENT_SOURCE_DIR，指当前处理的 CMakeLists.txt 所在的路径
```

```
CMAKE_CURRRENT_BINARY_DIR，如果是 in-source 编译，它跟 CMAKE_CURRENT_SOURCE_DIR 一致，如果是 out-ofsource编译，他指的是 target 编译目录。
```

```
CMAKE_CURRENT_LIST_FILE，输出调用这个变量的 CMakeLists.txt 的完整路径
```

```
CMAKE_CURRENT_LIST_LINE，输出这个变量所在的行
```

### cmake常用变量

```
CMAKE_MODULE_PATH
```

这个变量用来定义自己的cmake模块所在的路径。如果你的工程比较复杂，有可能会自己编写一些 cmake 模块，这些 cmake模块是随你的工程发布的，为了让 cmake 在处理CMakeLists.txt 时找到这些模块，你需要通过SET 指令，将自己的cmake模块路径设置一下。

```
SET(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)
```

### cmake常用变量

```
EXECUTABLE_OUTPUT_PATH和LIBRARY_OUTPUT_PATH，用来重新定义最终结果的存放目录
```

```
PROJECT_NAME，令定义的项目名称
```

### cmake调用环境变量的方式

使用$ENV{NAME}指令就可以调用系统的环境变量了。

```
MESSAGE(STATUS “HOME dir: $ENV{HOME}”)
```

设置环境变量的方式是：
SET(ENV{变量名} 值)


### cmake常用环境变量

```
CMAKE_INCLUDE_CURRENT_DIR
```

自动添加CMAKE_CURRENT_BINARY_DIR和CMAKE_CURRENT_SOURCE_DIR到当前处理
的CMakeLists.txt。相当于在每个 CMakeLists.txt 加入：

```
INCLUDE_DIRECTORIES(${CMAKE_CURRENT_BINARY_DIR} ${CMAKE_CURRENT_SOURCE_DIR})
```


### cmake常用环境变量

```
CMAKE_INCLUDE_DIRECTORIES_PROJECT_BEFORE
```

将工程提供的头文件目录始终至于系统头文件目录的前面，当你定义的头文件确实跟系统发生冲突时可以提供一些帮助

### cmake常用环境变量

```
CMAKE_INCLUDE_PATH和CMAKE_LIBRARY_PATH，设置头文件和库文件搜索路径
```

### 系统信息

```
CMAKE_MAJOR_VERSION，CMAKE 主版本号，比如 2.4.6 中的 2
CMAKE_MINOR_VERSION，CMAKE 次版本号，比如 2.4.6 中的 4
CMAKE_PATCH_VERSION，CMAKE 补丁等级，比如 2.4.6 中的 6
CMAKE_SYSTEM，系统名称，比如 Linux-2.6.22
CMAKE_SYSTEM_NAME，不包含版本的系统名，比如 Linux
CMAKE_SYSTEM_VERSION，系统版本，比如 2.6.22
CMAKE_SYSTEM_PROCESSOR，处理器名称，比如 i686.
UNIX，在所有的类 UNIX 平台为 TRUE，包括 OS X 和 cygwin
WIN32，在所有的 win32 平台为 TRUE，包括 cygwin
```

### 主要开关选项

```
CMAKE_C_FLAGS，设置 C 编译选项，也可以通过指令 ADD_DEFINITIONS()添加
CMAKE_CXX_FLAGS，设置 C++编译选项，也可以通过指令 ADD_DEFINITIONS()添加
```


## IF控制指令


### 指令语法格式


```
IF(expression)
 # THEN section.
 COMMAND1(ARGS ...)
 COMMAND2(ARGS ...)
 ...
 ELSE(expression)
 # ELSE section.
 COMMAND1(ARGS ...)
 COMMAND2(ARGS ...)
 ...
 ENDIF(expression)
```

### 表达式使用方法

```
IF(var)，如果变量不是：空，0，N, NO, OFF, FALSE, NOTFOUND 或
<var>_NOTFOUND 时，表达式为真。
IF(NOT var )，与上述条件相反。
IF(var1 AND var2)，当两个变量都为真是为真。
IF(var1 OR var2)，当两个变量其中一个为真时为真。
IF(COMMAND cmd)，当给定的 cmd 确实是命令并可以调用是为真。
IF(EXISTS dir)或者 IF(EXISTS file)，当目录名或者文件名存在时为真。
IF(file1 IS_NEWER_THAN file2)，当 file1 比 file2 新，或者 file1/file2 其
中有一个不存在时为真，文件名请使用完整路径。
IF(IS_DIRECTORY dirname)，当 dirname 是目录时，为真。
IF(variable MATCHES regex)
IF(string MATCHES regex)，当给定的变量或者字符串能够匹配正则表达式 regex 时为真。
IF(DEFINED variable)，如果变量被定义，为真。
```

### 表达式使用方法

数字比较表达式

```
IF(variable LESS number)
IF(string LESS number)
IF(variable GREATER number)
IF(string GREATER number)
IF(variable EQUAL number)
IF(string EQUAL number)
```

### 表达式使用方法

按照字母序的排列进行比较.

```
IF(variable STRLESS string)
IF(string STRLESS string)
IF(variable STRGREATER string)
IF(string STRGREATER string)
IF(variable STREQUAL string)
IF(string STREQUAL string)
```

### 例子

```
IF(WIN32)
MESSAGE(STATUS “This is windows.”)
#作一些 Windows 相关的操作
ELSE(WIN32)
MESSAGE(STATUS “This is not windows”)
#作一些非 Windows 相关的操作
ENDIF(WIN32)
```


### 例子

```
SET(CMAKE_ALLOW_LOOSE_LOOP_CONSTRUCTS ON)
IF(WIN32)
ELSE()
ENDIF()
```

### 例子

```
SET(CMAKE_ALLOW_LOOSE_LOOP_CONSTRUCTS ON)
IF(WIN32)
#do something related to WIN32
ELSEIF(UNIX)
#do something related to UNIX
ELSEIF(APPLE)
#do something related to APPLE
ENDIF(WIN32)
```

## WHILE控制指令

### 指令语法格式

```
WHILE(condition)
COMMAND1(ARGS ...)
COMMAND2(ARGS ...)
...
ENDWHILE(condition)
```

## FOREACH控制指令

### 指令语法格式(列表)

```
FOREACH(loop_var arg1 arg2 ...)
COMMAND1(ARGS ...)
COMMAND2(ARGS ...)
...
ENDFOREACH(loop_var)
```

### 例子(列表)

```
AUX_SOURCE_DIRECTORY(. SRC_LIST)
FOREACH(F ${SRC_LIST})
MESSAGE(${F})
ENDFOREACH(F)
```

### 指令语法格式(范围)

```
FOREACH(loop_var RANGE total)
ENDFOREACH(loop_var)
```

### 例子(范围)

```
FOREACH(VAR RANGE 10)
MESSAGE(${VAR})
ENDFOREACH(VAR)
```

### 指令语法格式(范围和步进)

```
FOREACH(loop_var RANGE start stop [step])
ENDFOREACH(loop_var)
```

### 例子(范围和步进)

```
FOREACH(A RANGE 5 15 3)
MESSAGE(${A})
ENDFOREACH(A)
```
