cmake_minimum_required(VERSION 3.5.1)
project(libctemplate C CXX)

# find_package(PythonInterp)
if(NOT PYTHON_EXECUTABLE)
  message(FATAL_ERROR "PYTHON_EXECUTABLE must be set")
endif()

set(CMAKE_DEBUG_POSTFIX d)

if(MSVC)
  add_compile_options(/W3 /wd4005 /wd4996 /wd4018 -D_CRT_SECURE_NO_WARNINGS)
  add_definitions(-D_VARIADIC_MAX=10)
endif()

set(SOURCE_PATH ${PROJECT_SOURCE_DIR}/src)
set(COMMON_INCLUDES ${PROJECT_BINARY_DIR}/include)

set(HTMLPARSER_CONFIG ${SOURCE_PATH}/htmlparser/htmlparser_fsm.config)
set(JSPASPER_CONFIG ${SOURCE_PATH}/htmlparser/jsparser_fsm.config)
set(FSM_GENERATOR ${SOURCE_PATH}/htmlparser/generate_fsm.py)

set(SRC_FILES
  "${SOURCE_PATH}/base/arena.cc"
  "${SOURCE_PATH}/htmlparser/htmlparser.cc"
  "${SOURCE_PATH}/htmlparser/jsparser.cc"
  "${SOURCE_PATH}/htmlparser/statemachine.cc"
  "${SOURCE_PATH}/per_expand_data.cc"
  "${SOURCE_PATH}/template.cc"
  "${SOURCE_PATH}/template_annotator.cc"
  "${SOURCE_PATH}/template_cache.cc"
  "${SOURCE_PATH}/template_dictionary.cc"
  "${SOURCE_PATH}/template_modifiers.cc"
  "${SOURCE_PATH}/template_namelist.cc"
  "${SOURCE_PATH}/template_pathops.cc"
  "${SOURCE_PATH}/template_string.cc"
  "${SOURCE_PATH}/windows/port.cc"
)

make_directory(${COMMON_INCLUDES}/htmlparser)

execute_process(
  COMMAND ${PYTHON_EXECUTABLE} ${FSM_GENERATOR} ${HTMLPARSER_CONFIG}
  WORKING_DIRECTORY ${COMMON_INCLUDES}/htmlparser
  OUTPUT_VARIABLE  HTMLPARSER_CONFIG_H
)
execute_process(
  COMMAND ${PYTHON_EXECUTABLE} ${FSM_GENERATOR} ${JSPASPER_CONFIG}
  WORKING_DIRECTORY ${COMMON_INCLUDES}/htmlparser
  OUTPUT_VARIABLE JSPASPER_CONFIG_H
)

file(WRITE "${COMMON_INCLUDES}/htmlparser/htmlparser_fsm.h" "${HTMLPARSER_CONFIG_H}")
file(WRITE "${COMMON_INCLUDES}/htmlparser/jsparser_fsm.h" "${JSPASPER_CONFIG_H}")

include_directories(src src/windows ${COMMON_INCLUDES})

add_library(libctemplate ${SRC_FILES})
if(NOT BUILD_SHARED_LIBS)
  # Note: CTEMPLATE_DLL_DECL should be empty to build static file
  target_compile_definitions(libctemplate PUBLIC -DCTEMPLATE_DLL_DECL=)
endif()

install(
  TARGETS libctemplate
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
)

if(NOT DISABLE_INSTALL_HEADERS)
  install(DIRECTORY src/windows/ctemplate/ DESTINATION include/ctemplate)
endif()
