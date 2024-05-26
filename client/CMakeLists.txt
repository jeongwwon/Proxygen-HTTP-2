cmake_minimum_required(VERSION 3.15)
project(proxygen_client)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# Needed to find proxygen-config.cmake
list(APPEND CMAKE_PREFIX_PATH "/app/proxygen/_build")

# Needed to find sodium-config.cmake
list(APPEND CMAKE_MODULE_PATH "/app/proxygen/build/fbcode_builder/CMake")

# Otherwise you will not find sodium correctly. If you install with
# package-manager as well, you get warnings...
file(GLOB SODIUM_DIR "/app/proxygen/_build/libsodium-*")
if(SODIUM_DIR)
	  list(APPEND CMAKE_PREFIX_PATH ${SODIUM_DIR})
  else()
	    message(FATAL_ERROR "Sodium directory not found")
    endif()

# Find required libraries, for the eche server example
find_package(proxygen CONFIG REQUIRED)
find_package(gflags REQUIRED)
find_package(zstd CONFIG REQUIRED)
find_package(folly REQUIRED)
# find_package(sodium REQUIRED) # Will not work

add_library(proxygencurl CurlClient.cpp)
target_include_directories(
	    proxygencurl PUBLIC
	    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
		)

target_compile_options(
	proxygencurl PRIVATE
	${_PROXYGEN_COMMON_COMPILE_OPTIONS}
	)
target_link_libraries(proxygencurl PUBLIC proxygen::proxygenhttpserver gflags zstd)

if (BUILD_SHARED_LIBS)
	    set_property(TARGET proxygencurl PROPERTY POSITION_INDEPENDENT_CODE ON)
	        if (DEFINED PACKAGE_VERSION)
			        set_target_properties(
					   proxygencurl PROPERTIES VERSION ${PACKAGE_VERSION}
		            )
		 endif()
endif()

install(
	TARGETS proxygencurl
	EXPORT proxygen-exports
	ARCHIVE DESTINATION ${LIB_INSTALL_DIR}
	LIBRARY DESTINATION ${LIB_INSTALL_DIR}
			)

add_executable(proxygen_curl CurlClientMain.cpp)
target_link_libraries(
	    proxygen_curl PUBLIC
	        proxygencurl
		)

target_compile_options(
	    proxygen_curl PRIVATE
	        ${_PROXYGEN_COMMON_COMPILE_OPTIONS}
		)
install(
	    TARGETS proxygen_curl
	        EXPORT proxygen-exports
		    DESTINATION bin
		 )
file(GLOB ZSTD_LIB_DIR "/app/proxygen/_build/zstd-*/lib")
if(ZSTD_LIB_DIR)
	  target_link_directories(proxygen_curl PUBLIC ${ZSTD_LIB_DIR})
  else()
	    message(FATAL_ERROR "Zstd library directory not found")
    endif()

