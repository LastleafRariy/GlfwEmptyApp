cmake_minimum_required(VERSION 3.1)

project("glad")

add_library(glad STATIC "src/glad.c")

target_include_directories (glad
                            PRIVATE           include
                            SYSTEM INTERFACE  include
                            )
