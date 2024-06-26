cmake_minimum_required(VERSION 3.1)
project(PCA9685 VERSION 1.0.0)

set(CMAKE_CXX_STANDARD 14)

#
# Dependencies
#

set(PYBIND11_PYTHON_VERSION "3" CACHE STRING "Version of the Python interpreter you'd like to use. (ie. 2.7 or 3.5)")

find_package(PythonInterp ${PYTHON_VERSION} REQUIRED)
message(STATUS "Using python interpreter: ${PYTHON_EXECUTABLE}")
set(PYBIND11_CPP_STANDARD -std=c++14)
find_package(pybind11 REQUIRED)

include_directories(include)


#
# LIBRARY
#

add_library(PCA9685 src/PCA9685.cpp src/I2CPeripheral.cpp)
target_link_libraries(PCA9685 i2c)
set_property(TARGET PCA9685 PROPERTY POSITION_INDEPENDENT_CODE ON)
set_target_properties(PCA9685 PROPERTIES VERSION ${PCA9685_VERSION} SOVERSION ${PCA9685_VERSION})
target_include_directories(PCA9685 PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>")
message(STATUS "PCA9685: Added PCA9685 library.")

#
# PYTHON BINDINGS
#

pybind11_add_module(PCA9685_python src/PCA9685-Python.cpp)
set_target_properties(PCA9685_python PROPERTIES OUTPUT_NAME PCA9685)
target_link_libraries(PCA9685_python PRIVATE PCA9685)


#
# C++ INSTALLATION
#

set(INSTALL_INCLUDE_DIR "include")
set(INSTALL_LIBS_DIR "lib")
set(INSTALL_STATIC_DIR "lib/static")
set(INSTALL_RUNTIME_DIR "bin")
set(INSTALL_CONFIG_DIR "${INSTALL_LIBS_DIR}/PCA9685/cmake")

include(CMakePackageConfigHelpers)
configure_package_config_file(
    cmake/PCA9685Config.cmake.in
        PCA9685Config.cmake
    INSTALL_DESTINATION lib/cmake/PCA9685
    PATH_VARS INSTALL_INCLUDE_DIR
)

install(
    TARGETS PCA9685
    EXPORT PCA9685Targets
    RUNTIME DESTINATION ${INSTALL_RUNTIME_DIR}
    LIBRARY DESTINATION ${INSTALL_LIBS_DIR}
    ARCHIVE DESTINATION ${INSTALL_STATIC_DIR}
)

install(
    DIRECTORY include/PCA9685
    DESTINATION ${INSTALL_INCLUDE_DIR}
)

install(
    EXPORT PCA9685Targets
    FILE PCA9685Targets.cmake
    DESTINATION ${INSTALL_CONFIG_DIR}
)

install(
    FILES "${CMAKE_CURRENT_BINARY_DIR}/PCA9685Config.cmake"
    DESTINATION ${INSTALL_CONFIG_DIR}
)


#
# PYTHON INSTALLATION
#

file(WRITE ${CMAKE_BINARY_DIR}/python_package/PCA9685/__init__.py "")
file(COPY cmake/setup.py DESTINATION ${CMAKE_BINARY_DIR}/python_package)
add_custom_target(
    python_copy_targets ALL
    COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:PCA9685> ${CMAKE_BINARY_DIR}/python_package/PCA9685
    COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:PCA9685_python> ${CMAKE_BINARY_DIR}/python_package/PCA9685
)


add_custom_target(
    install_python
    COMMAND ${PYTHON_EXECUTABLE} setup.py install --user
    WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/python_package
)
