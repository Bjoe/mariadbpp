#          Copyright The ViaDuck Project 2016 - 2018.
# Distributed under the Boost Software License, Version 1.0.
#    (See accompanying file LICENSE or copy at
#          http://www.boost.org/LICENSE_1_0.txt)

cmake_minimum_required(VERSION 3.1)

option(HUNTER_ENABLED "Enable Hunter package manager" NO)
include("cmake/HunterGate.cmake")
HunterGate(
    URL "https://github.com/ruslo/hunter/archive/v0.23.29.tar.gz"
    SHA1 "797be99a8a4f4666766257563fad8aa3aa032da2"
)

project(mariadbclientpp)

hunter_add_package(OpenSSL)
hunter_add_package(MariaDB-client)
find_package(OpenSSL REQUIRED)

# add additional cmake modules
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/external/cmake-modules")

# c++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

option(MARIADBPP_WITH_TESTS "With unit tests" OFF)
option(MARIADBPP_WITH_DOCS "Generate documentation (Needs doxygen)" OFF)

# dependencies
find_package(MariaDBClient REQUIRED)

# find files
file(GLOB mariadbclientpp_PUBLIC_HEADERS include/mariadb++/*)
file(GLOB mariadbclientpp_PRIVATE_HEADERS src/*.hpp)
file(GLOB mariadbclientpp_SOURCES src/*.cpp)

# set up target
add_library(mariadbclientpp ${mariadbclientpp_SOURCES} ${mariadbclientpp_PUBLIC_HEADERS} ${mariadbclientpp_PRIVATE_HEADERS})

# target options
target_link_libraries(mariadbclientpp PUBLIC MariaDB::client pthread)
target_include_directories(mariadbclientpp PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)

# install configuration
install(FILES ${mariadbclientpp_PUBLIC_HEADERS} DESTINATION include/mariadb++)
install(TARGETS mariadbclientpp
	EXPORT mariadbclientpp
	ARCHIVE DESTINATION lib
	RUNTIME DESTINATION bin
	LIBRARY DESTINATION lib
	PUBLIC_HEADER DESTINATION include
)

install(EXPORT mariadbclientpp NAMESPACE mariadb:: DESTINATION lib/cmake/mariadb)

SET(CPACK_PACKAGE_DESCRIPTION_SUMMARY "MariaDB++")
SET(CPACK_PACKAGE_VENDOR "ViaDuck")
SET(CPACK_PACKAGE_VERSION "0.0.1")
SET(CPACK_GENERATOR "RPM;DEB")
SET(CPACK_DEBIAN_PACKAGE_MAINTAINER "Nobody")
INCLUDE(CPack)

if(MARIADBPP_WITH_TESTS)
# tests
  add_subdirectory(test)
endif()

if(MARIADBPP_WITH_DOCS)
  # doxygen
  include(Doxygen)
  if (DOXYGEN_FOUND)
	  setup_doxygen(mariadbpp_doc ${CMAKE_CURRENT_SOURCE_DIR}/Doxyfile.in)
  endif()
endif()
