# ================================= Apache 2.0 =================================
#
# Copyright (C) 2021 Continental
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
# ================================= Apache 2.0 =================================
cmake_minimum_required(VERSION 3.12)

project(rosidl_typesupport_protobuf_cpp)

if(DEFINED ENV{PROTOBUF_STATIC_DISABLE})
  set(PROTOBUF_STATIC_DISABLE $ENV{PROTOBUF_STATIC_DISABLE}
    CACHE BOOL "If Protobuf Static should be disabled.")
else()
  set(PROTOBUF_STATIC_DISABLE FALSE
    CACHE BOOL "If Protobuf Static should be disabled.")
endif()

if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

if (CMAKE_CXX_COMPILER_ID MATCHES "MSVC")
	add_compile_options(/W4)
	add_compile_definitions(_CRT_SECURE_NO_WARNINGS)
endif()

if(PROTOBUF_STATIC_DISABLE)
  ament_package()
  message(STATUS "Protobuf static typesupport implementation explicitly disabled - skipping '${PROJECT_NAME}'")
  return()
endif()

find_package(ament_cmake_python REQUIRED)
find_package(rosidl_typesupport_protobuf REQUIRED)
find_package(rosidl_generator_c REQUIRED)

ament_export_dependencies(rmw)
ament_export_dependencies(rcutils)
ament_export_dependencies(rosidl_cmake)
ament_export_dependencies(rosidl_typesupport_interface)
ament_export_dependencies(rosidl_typesupport_protobuf)
ament_export_dependencies(rosidl_adapter_proto)

ament_export_include_directories(include)

if(${is_foxy_or_greater})
  ament_export_dependencies(rosidl_runtime_cpp)
else()
  ament_export_dependencies(rosidl_generator_c)
  ament_export_dependencies(rosidl_generator_cpp)
endif()

ament_python_install_package(${PROJECT_NAME})

add_library(${PROJECT_NAME} SHARED
  src/identifier.cpp
  src/wstring_conversion.cpp)

if(WIN32)
  target_compile_definitions(${PROJECT_NAME}
    PRIVATE "ROSIDL_TYPESUPPORT_PROTOBUF_CPP_BUILDING_DLL")
endif()

target_include_directories(${PROJECT_NAME}
  PUBLIC include
)

ament_target_dependencies(${PROJECT_NAME} rosidl_typesupport_protobuf)

ament_export_libraries(${PROJECT_NAME})

ament_index_register_resource("rosidl_typesupport_cpp")

ament_package(CONFIG_EXTRAS "cmake/rosidl_typesupport_protobuf_cpp-extras.cmake.in")

install(
  PROGRAMS bin/rosidl_typesupport_protobuf_cpp
  DESTINATION lib/rosidl_typesupport_protobuf_cpp
)

install(
  DIRECTORY cmake resource
  DESTINATION share/${PROJECT_NAME}
)

install(
  DIRECTORY include/
  DESTINATION include
)

install(
  TARGETS ${PROJECT_NAME} EXPORT ${PROJECT_NAME}
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)