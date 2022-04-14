CMAKE_MINIMUM_REQUIRED(VERSION 3.6)

PROJECT("lpeglabel")

ENABLE_TESTING()
IF(CMAKE_VERSION VERSION_LESS 3.12.0)
	FIND_PACKAGE(PythonInterp 3.5 REQUIRED)
	SET(PYTHON_EXE "${PYTHON_EXECUTABLE}")
ELSE(CMAKE_VERSION VERSION_LESS 3.12.0)
	FIND_PACKAGE(Python3 REQUIRED)
	SET(PYTHON_EXE "${Python3_EXECUTABLE}")
ENDIF(CMAKE_VERSION VERSION_LESS 3.12.0)

# Build the Lua module.
ADD_LIBRARY(TARGET_lpeglabel MODULE
            lpcap.c
            lpcode.c
            lpprint.c
            lptree.c
            lpvm.c)
TARGET_INCLUDE_DIRECTORIES(TARGET_lpeglabel
                           PRIVATE ${LUA_INCLUDE_DIR})
IF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))
	TARGET_LINK_LIBRARIES(TARGET_lpeglabel ${LUA_LIBRARIES})
ENDIF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))

# Set the name of the output file to "lpeglabel".
SET_TARGET_PROPERTIES(TARGET_lpeglabel PROPERTIES PREFIX "" OUTPUT_NAME "lpeglabel")

# On mingw link all compiler libraries static.
IF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))
	SET_PROPERTY(TARGET TARGET_lpeglabel PROPERTY LINK_FLAGS "-static -static-libgcc -static-libstdc++")
ENDIF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))

# Install the lua module.
INSTALL(TARGETS TARGET_lpeglabel
        EXPORT EXPORT_package
        LIBRARY DESTINATION ${INSTALL_DIR_LUA_MODULES}
        ARCHIVE DESTINATION lib
        RUNTIME DESTINATION bin
        INCLUDES DESTINATION include)

# Add tests for this module.
IF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))
	# Here are the MinGW specific tests.
	ADD_TEST(NAME lpeglabel_MinGW_DLL_dependencies
	         COMMAND "${PYTHON_EXE}" ${TEST_SCRIPT_FOLDER}/mingw_dll_dependencies.py -u lua5.1 -u lua5.2 -u lua5.3 -u lua5.4 $<TARGET_FILE:TARGET_lpeglabel>)
ENDIF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))

# Install all LUA scripts.
INSTALL(FILES relabel.lua
        DESTINATION ${INSTALL_DIR_LUA_SCRIPTS})
