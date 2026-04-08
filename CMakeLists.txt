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
set(CMAKE_CXX_SCAN_FOR_MODULES OFF)

# TODO: Make these into proper options
set(BUILD_TESTING TRUE)
set(CMAKE_BUILD_SHARED_LIBS ON)
option(BUILD_PYBIND11_BINDINGS "Build Python bindings via pybind11" ON)

# Define the documentation target
include(nwx_cxx_api_docs)
nwx_cxx_api_docs("cpp/include" "cpp/src")

# Build the Library
include(cmake/nwx_library.cmake)
nwx_library(${PROJECT_NAME} "cpp/include" "cpp/src")

# Register the C++ tests
include(cmake/catch2_tests_from_dir.cmake)
catch2_tests_from_dir(
        "unit_test_${PROJECT_NAME}"
        tests/cpp/unit_tests/wtf
        ${PROJECT_NAME}
)

# Register the Python module
include(cmake/nwx_python_module.cmake)
nwx_python_module(${PROJECT_NAME}_python "cpp/src")
