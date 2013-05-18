cmake_minimum_required(VERSION 2.8)
 
project(meta-ser CXX)

# Set CXX Flags 
set (CMAKE_CXX_FLAGS 		  "-Wall -std=c++11")
set (CMAKE_CXX_FLAGS_DEBUG   "-O0 -g")
set (CMAKE_CXX_FLAGS_RELEASE "-O3 -DNDEBUG")

# Compiler-specific C++11 activation.
if (CMAKE_COMPILER_IS_GNUCXX)
    execute_process(
        COMMAND ${CMAKE_CXX_COMPILER} -dumpversion OUTPUT_VARIABLE GCC_VERSION)
    if (NOT (GCC_VERSION VERSION_GREATER 4.7 OR GCC_VERSION VERSION_EQUAL 4.7))
        message(FATAL_ERROR "${PROJECT_NAME} requires g++ 4.7 or greater.")
    endif ()
elseif ("${CMAKE_CXX_COMPILER_ID}" MATCHES "Clang")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++")
else ()
    message(FATAL_ERROR "Your C++ compiler does not support C++11.")
endif ()

# Check for dependencies 
find_package(Boost REQUIRED serialization)
find_package(GTest REQUIRED)

include_directories(${CMAKE_SOURCE_DIR}/include )
include_directories(${Boost_INCLUDE_DIRS})

set(CMAKE_BUILD_TYPE Release)

# Unit tests 
enable_testing()

include_directories(${GTEST_INCLUDE_DIRS})

macro(add_unit_test case_name)

	add_test(NAME ${case_name} 
			COMMAND valgrind
				--leak-check=full
				--show-reachable=no
				--track-fds=yes
				--error-exitcode=1
				--track-origins=yes
				#--log-file=${CMAKE_CURRENT_BINARY_DIR}/valgrind.log.${case_name}
				${CMAKE_CURRENT_BINARY_DIR}/${case_name}
				WORKING_DIRECTORY
				${CMAKE_CURRENT_BINARY_DIR}
		)

endmacro(add_unit_test)

message(${Boost_LIBRARIES})

set ( ut_prefix  ut_ms_ )
file(GLOB_RECURSE test_cases test/*.cc)
foreach ( case_file ${test_cases})
	get_filename_component( case_name ${case_file} NAME_WE )
	
	set (case_name ${ut_prefix}${case_name})
	add_executable(${case_name} ${case_file})
	target_link_libraries(${case_name} ${Boost_LIBRARIES} ${GTEST_LIBRARY} ${GTEST_MAIN_LIBRARY})
	add_unit_test(${case_name})

endforeach(case_file)

