
CMAKE_MINIMUM_REQUIRED(VERSION 3.1)

SET(PROJECT_NAME texture_remix_optimizer)

PROJECT(${PROJECT_NAME})

# ------------------------------- ADDITIONAL CMAKE SCRIPTS --------------------------
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/simple_cl/cmake")

# ------------------------------- GENERAL PROJECT SETTINGS --------------------------
SET(CMAKE_CONFIGURATION_TYPES "Release;Debug;RelWithDebInfo" CACHE STRING "Semicolon separated list of configuration types" FORCE)

SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
FOREACH( OUTPUTCONFIG ${CMAKE_CONFIGURATION_TYPES} )
	STRING( TOUPPER ${OUTPUTCONFIG} OUTPUTCONFIG )
    SET( CMAKE_RUNTIME_OUTPUT_DIRECTORY_${OUTPUTCONFIG} ${CMAKE_BINARY_DIR}/bin )
    SET( CMAKE_LIBRARY_OUTPUT_DIRECTORY_${OUTPUTCONFIG} ${CMAKE_BINARY_DIR}/lib )
    SET( CMAKE_ARCHIVE_OUTPUT_DIRECTORY_${OUTPUTCONFIG} ${CMAKE_BINARY_DIR}/bin )
ENDFOREACH( OUTPUTCONFIG CMAKE_CONFIGURATION_TYPES )

# ------------------------------- EXTERNAL DEPENDENCIES ------------------------------

SET(Boost_USE_STATIC_LIBS ON)
SET(Boost_USE_MULTITHREADED ON)
SET(BOOST_USE_STATIC_RUNTIME OFF)

FIND_PACKAGE(Boost REQUIRED COMPONENTS chrono date_time filesystem log program_options system)
FIND_PACKAGE(OpenCV 3 REQUIRED)
FIND_PACKAGE(OpenMP REQUIRED)

# ----------------------------- SETUP TARGETS / DEPENDENCIES FOR SUB PROJECTS --------

# create separate alias targets for individual dependencies
# boost
ADD_LIBRARY(LIBS_BOOST INTERFACE)
target_include_directories(LIBS_BOOST
	INTERFACE
		${Boost_INCLUDE_DIRS}
)
target_link_libraries(LIBS_BOOST
	INTERFACE
		${Boost_LIBRARIES}
)

# opencv
ADD_LIBRARY(LIBS_OPENCV INTERFACE)
target_include_directories(LIBS_OPENCV
	INTERFACE
		${OpenCV_INCLUDE_DIRS}
)
target_link_libraries(LIBS_OPENCV
	INTERFACE
		${OpenCV_LIBS}
)

# set_target_properties(${Boost_LIBRARIES} PROPERTIES MAP_IMPORTED_CONFIG_RELWITHDEBINFO RELEASE)
set_target_properties(${OpenCV_LIBS} PROPERTIES MAP_IMPORTED_CONFIG_RELWITHDEBINFO RELEASE)

# ------------------------------ INTERNAL LIBS ---------------------------------------

SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")

# 
SET(LIBS
	${Boost_LIBRARIES}
	${OpenCV_LIBS}
)

# dependencies built along with this project
ADD_DEFINITIONS(-DHAVE_CONFIG_H -D_CRT_SECURE_NO_WARNINGS -D_SCL_SECURE_NO_WARNINGS)
ADD_SUBDIRECTORY(lib)
ADD_SUBDIRECTORY(simple_cl)

# add bundle lib target to replace old style global CMake dependencies

ADD_LIBRARY(LIBS_ALLDEPS INTERFACE)
TARGET_LINK_LIBRARIES(LIBS_ALLDEPS
	INTERFACE
		LIBS_BOOST
		LIBS_OPENCV
		trlib)
TARGET_INCLUDE_DIRECTORIES(LIBS_ALLDEPS
	INTERFACE
		${PROJECT_SOURCE_DIR}
		${CMAKE_CURRENT_SOURCE_DIR}
)

# Library targets available:
#	LIBS_BOOST
#	LIBS_OPENCV
#	trlib
#	LIBS_ALLDEPS # old "LIBS" variable.

# --------------------------------- EXECUTABLE ------------------------------------

ADD_SUBDIRECTORY(tools)
