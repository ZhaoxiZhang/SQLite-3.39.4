cmake_minimum_required(VERSION 3.21)
project(SqliteBuild C)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c99 -DSQLITE_CORE=1")

include_directories(${CMAKE_CURRENT_SOURCE_DIR})
file(GLOB sources "*.c")
file(GLOB headers "*.h")
list(REMOVE_ITEM sources ${CMAKE_CURRENT_SOURCE_DIR}/lempar.c)
list(REMOVE_ITEM sources ${CMAKE_CURRENT_SOURCE_DIR}/lemon.c)

add_executable(SqliteBuild ${sources} ${headers} SqliteMain.c)
