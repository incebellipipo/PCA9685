cmake_minimum_required(VERSION 3.1)
project(example)

set(CMAKE_CXX_STANDARD 14)

find_package(PCA9685 REQUIRED)

add_executable(example-cpp main.cpp)
target_link_libraries(example-cpp ${PCA9686_LIBRARY})
