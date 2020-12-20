# CMakeLists.txt
cmake_minimum_required(VERSION 3.1)
message( STATUS "cmake: ${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION}.${CMAKE_PATCH_VERSION}" )




#-- c++17
set (CMAKE_CXX_STANDARD 17)
set (CMAKE_CXX_STANDARD_REQUIRED ON)
set (CMAKE_CXX_EXTENSIONS OFF)


# project name
project ( glfwApp )



# build_Dir
set (EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/publish)

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/publish)





#                     src
FILE(GLOB_RECURSE SRC_FILES   
                        src/*.cpp
                        src/*.c
                        src/*.rc
                        )

#                      project 

if( ${CMAKE_VERSION} VERSION_GREATER_EQUAL 3.16 )
    set ( CMAKE_UNITY_BUILD_BATCH_SIZE 16 ) # 10 ～ 20
endif()


add_executable(
    glfwApp
    ${SRC_FILES}
)

set_target_properties(glfwApp PROPERTIES PREFIX "")
set_target_properties(glfwApp PROPERTIES OUTPUT_NAME "glfwApp")



if( ${CMAKE_VERSION} VERSION_GREATER_EQUAL 3.16 )
    target_precompile_headers( glfwApp PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/src/pch/pch.h ) # PCH
    set_target_properties( glfwApp PROPERTIES UNITY_BUILD ON) # UNITY
endif()


#                   include

MACRO( head_dirs return_list )
    FILE(GLOB_RECURSE   new_list 
                        ${CMAKE_CURRENT_SOURCE_DIR}/src/*.h
                        ${CMAKE_CURRENT_SOURCE_DIR}/src/*.hpp
                        )
    SET(dir_list "")
    FOREACH(file_path ${new_list})
        GET_FILENAME_COMPONENT(dir_path ${file_path} PATH)
        SET(dir_list ${dir_list} ${dir_path})
    ENDFOREACH()
    LIST(REMOVE_DUPLICATES dir_list)
    SET(${return_list} ${dir_list})
ENDMACRO()


head_dirs( header_dirs )

target_include_directories ( glfwApp PUBLIC

    ${CMAKE_CURRENT_SOURCE_DIR}/deps/rapidjson
    ${CMAKE_CURRENT_SOURCE_DIR}/deps/fmt/include

    ${CMAKE_CURRENT_SOURCE_DIR}/src
    ${CMAKE_CURRENT_SOURCE_DIR}/src/pch

    ${header_dirs}
    )


target_include_directories ( glfwApp SYSTEM PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/deps
    )


#-- glfw --
if( UNIX )
    target_include_directories ( glfwApp PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/deps/glfw_src/include)
else()
    target_include_directories ( glfwApp PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/deps/glfw/include)
endif( UNIX )



if( UNIX )
    set(GLFW_BUILD_DOCS OFF CACHE BOOL "" FORCE)
    set(GLFW_BUILD_TESTS OFF CACHE BOOL "" FORCE)
    set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)
    add_subdirectory (${CMAKE_CURRENT_SOURCE_DIR}/deps/glfw_src)
endif( UNIX )

add_subdirectory (${CMAKE_CURRENT_SOURCE_DIR}/deps/glad)
add_subdirectory (${CMAKE_CURRENT_SOURCE_DIR}/deps/fmt EXCLUDE_FROM_ALL)

#             target_link_libraries
if ( UNIX )
    target_link_libraries( glfwApp  
        glad
        glfw
        ${GLFW_LIBRARIES}
        fmt
        )

else()
    target_link_libraries( glfwApp
        glad
        ${CMAKE_CURRENT_SOURCE_DIR}/deps/glfw/lib-vc2019/glfw3.lib
        fmt
        )
endif( UNIX )
