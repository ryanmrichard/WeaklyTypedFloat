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

# Version from scikit-build-core when building wheels/sdists; plain CMake keeps 1.0.0.
# project(VERSION) accepts only numeric major.minor.patch[.tweak]; strip any PEP440 suffix.
if(DEFINED SKBUILD_PROJECT_VERSION AND NOT SKBUILD_PROJECT_VERSION STREQUAL "")
    set(_wtf_version_source "${SKBUILD_PROJECT_VERSION}")
else()
    set(_wtf_version_source "1.0.0")
endif()
if(_wtf_version_source MATCHES "^([0-9]+)\\.([0-9]+)\\.([0-9]+)")
    set(_wtf_cmake_version "${CMAKE_MATCH_1}.${CMAKE_MATCH_2}.${CMAKE_MATCH_3}")
else()
    set(_wtf_cmake_version "1.0.0")
endif()

# Downloads common CMake modules used throughout NWChemEx
project(wtf VERSION "${_wtf_cmake_version}" LANGUAGES CXX)

include(cmake/get_nwx_cmake.cmake)

include(disable_in_source_builds)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_SCAN_FOR_MODULES OFF)

# TODO: Make these into proper options
set(BUILD_TESTING TRUE)
set(CMAKE_BUILD_SHARED_LIBS ON)
option(BUILD_PYBIND11_BINDINGS "Build Python bindings via pybind11" ON)

# Define the documentation target
include(nwx_cxx_api_docs)
nwx_cxx_api_docs("cxx/include" "cxx/src")

# Build the Library
include(nwx_library)
nwx_library(${PROJECT_NAME} "cxx/include" "cxx/src")

# Register the C++ tests
include(catch2_tests_from_dir)
catch2_tests_from_dir(
    "unit_test_${PROJECT_NAME}"
    "tests/cxx/unit_tests/${PROJECT_NAME}"
    ${PROJECT_NAME}
)

# Register the Python module
include(nwx_python_module)
nwx_python_module(${PROJECT_NAME} "cxx/src")
