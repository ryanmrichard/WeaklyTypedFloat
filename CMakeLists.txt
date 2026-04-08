# Copyright 2025 NWChemEx-Project
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

cmake_minimum_required(VERSION 3.14)
# Downloads common CMake modules used throughout NWChemEx
include(cmake/get_nwx_cmake.cmake)

project(wtf VERSION "1.0.0" LANGUAGES CXX)
include(cmake/disable_in_source_builds.cmake)
set(CMAKE_CXX_STANDARD 20)

# TODO: Make these into proper options
set(BUILD_TESTING TRUE)
set(CMAKE_BUILD_SHARED_LIBS ON)

include(cmake/get_dependencies.cmake)

# Define the documentation target
include(nwx_cxx_api_docs)
nwx_cxx_api_docs("cpp/include" "cpp/src")

# Build the Library
file(GLOB_RECURSE WTF_HEADER_FILES CONFIGURE_DEPENDS cpp/include/wtf/*.hpp)
file(GLOB_RECURSE WTF_SOURCE_FILES CONFIGURE_DEPENDS cpp/src/wtf/*.cpp)
list(FILTER WTF_SOURCE_FILES EXCLUDE REGEX ".*/export_.*\\.cpp$")
add_library(${PROJECT_NAME} ${WTF_SOURCE_FILES})
target_link_libraries(${PROJECT_NAME})
target_include_directories(${PROJECT_NAME}
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/cpp/include>
        $<INSTALL_INTERFACE:include>
)

if(BUILD_PYBIND11_BINDINGS)
    set_target_properties(${PROJECT_NAME} PROPERTIES POSITION_INDEPENDENT_CODE ON)
endif()

if(${BUILD_TESTING})
    include(CTest)
    get_dependencies(catch2)
    include(cmake/catch2_tests_from_dir.cmake)
    catch2_tests_from_dir(
        "unit_test_${PROJECT_NAME}"
        tests/cpp/unit_tests/wtf
        ${PROJECT_NAME}
    )
endif()


option(BUILD_PYBIND11_BINDINGS "Build Python bindings via pybind11" ON)
if(BUILD_PYBIND11_BINDINGS)
    get_dependencies(pybind11)
    pybind11_add_module(wtf_python
        cpp/src/wtf/export_wtf.cpp
        cpp/src/wtf/fp/export_float.cpp
    )
    target_link_libraries(wtf_python PRIVATE ${PROJECT_NAME})
    target_include_directories(wtf_python PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/cpp/include
    )
    set_target_properties(wtf_python PROPERTIES OUTPUT_NAME wtf)
    install(TARGETS wtf_python
        DESTINATION $<IF:$<BOOL:${SKBUILD_PLATLIB_DIR}>,${SKBUILD_PLATLIB_DIR},lib>
    )
endif()

include(cmake/install_target.cmake)
install_library(
    ${PROJECT_NAME}
    "${CMAKE_CURRENT_SOURCE_DIR}/cpp/include/${PROJECT_NAME}"
)
