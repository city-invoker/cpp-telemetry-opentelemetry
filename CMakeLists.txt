#
#
# Tencent is pleased to support the open source community by making tRPC available.
#
# Copyright (C) 2023 THL A29 Limited, a Tencent company.
# All rights reserved.
#
# If you have downloaded a copy of the tRPC source code from Tencent,
# please note that tRPC source code is licensed under the  Apache 2.0 License,
# A copy of the Apache 2.0 License is included in this file.
#
#

cmake_minimum_required(VERSION 3.14)

project(trpc-cpp-plugin-telemetry-opentelemetry)

set(CMAKE_VERBOSE_MAKEFILE ON)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_BUILD_TYPE DEBUG)

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

# include *.h
set(TRPC_INSTALL_PATH /usr/local/trpc-cpp/trpc)
set(CMAKE_INSTALL_PREFIX  "${TRPC_INSTALL_PATH}/plugins/optl_plugin/")

include(${TRPC_INSTALL_PATH}/cmake/config/trpc_config.cmake)
include(${TRPC_INSTALL_PATH}/cmake/tools/trpc_utils.cmake)
set(PROTOBUF_PROTOC_EXECUTABLE ${TRPC_INSTALL_PATH}/bin/protoc)
set(TRPC_TO_CPP_PLUGIN ${TRPC_INSTALL_PATH}/bin/trpc_cpp_plugin)

include_directories(${INCLUDE_INSTALL_PATHS})

message("Cloning opentelemetry_cpp sdk, please waiting...")
include(com_github_opentelemetry_cpp)
include_directories(${TARGET_INCLUDE_PATHS})

message("Cloning opentelemetry_proto, please waitint...")
include(com_github_opentelemetry_proto)
include_directories(${OPTL_PROTO_INCLUDE_PATH})

include_directories(${CMAKE_CURRENT_SOURCE_DIR})

# Build plugin
file(GLOB_RECURSE SRC_FILES ./trpc/*.cc)
file(GLOB_RECURSE TEST_FILES ./trpc/*test.cc
                             ./trpc/telemetry/opentelemetry/testing/*)
list(REMOVE_ITEM SRC_FILES ${TEST_FILES})

set(TARGET trpc_cpp_plugin_telemetry_opentelemetry)
add_library(trpc_cpp_plugin_telemetry_opentelemetry
              ${SRC_FILES}
              ${OUT_OPENTELEMETRY_PROTO_TRPC_PB_PROTO_FILES})

# lib directories
link_directories(${LIBRARY_INSTALL_PATHS})
target_link_libraries(${TARGET} PRIVATE ${TARGET_LINK_LIBS})

set(LIBRARY_OUTPUT_PATH "${CMAKE_SOURCE_DIR}/lib/")

if(OPTL_BUILD_SHARED)
    set(LIBS_FILTER "*.so*")
    set(COLLECT_LIB_SCRIPT ${CMAKE_CURRENT_SOURCE_DIR}/cmake/tools/shared_lib_pack.sh)
else()
    set(LIBS_FILTER "*.a")
    set(COLLECT_LIB_SCRIPT ${CMAKE_CURRENT_SOURCE_DIR}/cmake/tools/static_lib_pack.sh)
endif()

add_custom_command(
    TARGET  trpc_cpp_plugin_telemetry_opentelemetry POST_BUILD
    COMMAND sh ${COLLECT_LIB_SCRIPT} ${CMAKE_SOURCE_DIR}
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
    COMMENT "optl_plugin collect third-party library files"
    VERBATIM
)


# install *.h *.hpp
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/trpc/
            DESTINATION optl_trpc/include/trpc
            FILES_MATCHING PATTERN "*.h"
                           PATTERN "*.hpp"
                           PATTERN "*test*" EXCLUDE
                           PATTERN "^.*" EXCLUDE
                           )

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/cmake_third_party/com_github_opentelemetry_cpp/api/include/
            DESTINATION optl_cpp/api/include/
            FILES_MATCHING PATTERN "*.h"
                           PATTERN "*.hpp"
                           PATTERN "*test*" EXCLUDE
                           PATTERN "^.*" EXCLUDE
                           )
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/cmake_third_party/com_github_opentelemetry_cpp/sdk/include/
            DESTINATION optl_cpp/sdk/include/
            FILES_MATCHING PATTERN "*.h"
                           PATTERN "*.hpp"
                           PATTERN "*test*" EXCLUDE
                           PATTERN "^.*" EXCLUDE
                           )

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/cmake_third_party/com_github_opentelemetry_proto/
            DESTINATION optl_proto/include/
            FILES_MATCHING PATTERN "*.h"
                           PATTERN "*.hpp"
                           PATTERN "*test*" EXCLUDE
                           PATTERN "^.*" EXCLUDE
                           PATTERN "^/.*" EXCLUDE
                           )
# install .a
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/lib/
            DESTINATION lib
            FILES_MATCHING PATTERN "*.a")

install(DIRECTORY ${CMAKE_BINARY_DIR}/lib/
            DESTINATION lib
            FILES_MATCHING PATTERN "*.a")

