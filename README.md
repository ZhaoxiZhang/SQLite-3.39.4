# SQLite-3.39.4

## 官方编译方式

SQLite3 源码提供了非常便捷的编译脚本，通过执行以下命令可以编译得到`sqlite3.c` 、`sqlite3.h`、`sqlite3ext.h`、`shell.c` 以及一个可执行程序 sqlite3。所有的代码都被合并到了`sqlite3.c` 文件中，根据官网[How To Compile SQLite](https://www.sqlite.org/howtocompile.html) 一文中提到，这样做的好处是编译器能够进行更进一步的优化，从而提升5%～10%的性能。

```
mkdir bld                ;#  Build will occur in a sibling directory
cd bld                   ;#  Change to the build directory
../sqlite/configure      ;#  Run the configure script
make                     ;#  Run the makefile.
make sqlite3.c           ;#  Build the "amalgamation" source file
make test                ;#  Run some tests (requires Tcl)
```

将所有源码合并得到一个文件的方式尽管带来了很多便利的地方，但在实际使用时却有一些问题。由于合并得到的文件太大，通过`Clion`打开后会提示：
```
The file size (8.56MB) exceeds the configured limit(2.56MB). Code insight features are not available.
```
并且对其编辑或者调试可以感受到IDE明显的卡顿。尽管这些是外部工具自身的问题，那么我们能不能将`SQLite`退回多文件的编译方式缓解由于工具性能不足带来的问题呢？

## 改造项目
基于 3.39.4 版本更改
- 新建一个空项目
- 将 sqlite3 源码底下 src 目录中所有文件拷贝到新建的项目中
- 将 sqlite3 源码底下 tool 目录中的`lemon.c`和`lempar.c`拷贝过来
- 执行`gcc -o lemon lemon.c`生成lemon程序
- 运行`./lemon parse.y`生成`parse.c`和`parse.h`文件
- 在 sqlite3 源码执行`./configure`和`make`命令
- 将编译得到的`tscr`底下的`opencodes.c`、`opencodec.h`和`sqlite.h`拷贝过来
- 删除 test* 等一系列测试文件，以及`tclsqlite.c`文件
- 创建CMakeLists.txt如下
```CMake
cmake_minimum_required(VERSION 3.21)  
project(SqliteBuild C)  
  
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c99 -DSQLITE_CORE=1")  
  
include_directories(${CMAKE_CURRENT_SOURCE_DIR})  
file(GLOB sources "*.c")  
file(GLOB headers "*.h")  
list(REMOVE_ITEM sources ${CMAKE_CURRENT_SOURCE_DIR}/lempar.c)  
list(REMOVE_ITEM sources ${CMAKE_CURRENT_SOURCE_DIR}/lemon.c)  
  
add_executable(SqliteBuild ${sources} ${headers} SqliteMain.c)
```

SqliteMain.c 为自己调试添加的入口文件，可自行更换
```C
#include <stdio.h>  
#include "sqlite3.h"  
  
/* print a record from table outputed by sql statement */  
static int callback(void* NotUsed, int argc, char** argv, char** azColName) {  
    int i;  
    for (i = 0; i < argc; i++) {  
        printf("%s = %s\n", azColName[i], argv[i] ? argv[i] : "NULL");  
    }  
    printf("\n");  
    return 0;  
}  
  
int main(int argc, char** argv) {  
    sqlite3* db;  
    char* zErrMsg = 0;  
    int rc;  
  
    if (argc != 3) {  
        fprintf(stderr, "Usage: %s DATABASE SQL-STATEMENT\n", argv[0]);  
        return(1);  
    }  
    rc = sqlite3_open(argv[1], &db);  /* open database */  
    if (rc) {  
        fprintf(stderr, "Can't open database: %s\n", sqlite3_errmsg(db));  
        sqlite3_close(db);  
        return(1);  
    }  
    rc = sqlite3_exec(db, argv[2], callback, 0, &zErrMsg);  /* execute SQL statement */  
    if (rc != SQLITE_OK) {  
        fprintf(stderr, "SQL error: %s\n", zErrMsg);  
        sqlite3_free(zErrMsg);  
    }  
    sqlite3_close(db);  /* close database */  
    return 0;  
}
```

完整源码地址：https://github.com/ZhaoxiZhang/SQLite-3.39.4