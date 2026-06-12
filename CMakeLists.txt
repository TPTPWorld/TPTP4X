cmake_minimum_required(VERSION 3.18)

set(CMAKE_C_COMPILER gcc CACHE FILEPATH "C compiler")

project(TPTP4X
  LANGUAGES C
)

# set(CMAKE_EXPORT_COMPILE_COMMANDS ON)    # provide clues to IDE IntelliSense

if(DEFINED ENV{TPTP_HOME})
  set(TPTP_HOME "$ENV{TPTP_HOME}" CACHE PATH "TPTP installation root")
else()
  set(TPTP_HOME "/home/tptp"      CACHE PATH "TPTP installation root")
endif()

################################################################################
# download jjparser at CMake configuration time
# but ignore jjparser's own CMake project

include(FetchContent)

set(JJPARSER_GIT_TAG "master" CACHE STRING "Git tag, branch, or commit to use for JJParser")

FetchContent_Declare(
  jjparser
  GIT_REPOSITORY https://github.com/TPTPWorld/JJParser.git
  GIT_TAG        "${JJPARSER_GIT_TAG}"
  SOURCE_SUBDIR  cmake-do-not-add
  #[[
  `SOURCE_SUBDIR` points to a non-existent directory.
  This is to avoid building any targets defined in JJParser's own `CMakeLists.txt`.
  This is allowed by the official documentation:
  https://cmake.org/cmake/help/latest/module/FetchContent.html#command:fetchcontent_makeavailable
  #]]
)

FetchContent_MakeAvailable(jjparser)

################################################################################
# configuration of jjparser for tptp4X (linked to curl and math library)

find_package(CURL REQUIRED)

set(JJPARSER_SOURCES
  ListStatistics.c
  List.c
  Examine.c
  Modify.c
  Tree.c
  TreeStatistics.c
  ParseTPTP.c
  ParseTSTP.c
  PrintTSTP.c
  Parsing.c
  Signature.c
  Tokenizer.c
  Utilities.c
  Compare.c
  Interpret.c
  PrintXML.c
  PrintSMT2.c
  PrintDFG.c
  PrintOtter.c
  PrintKIF.c
  PrintSUMO.c
  PrintLP.c
  PrintDK.c
  FileUtilities.c
  SystemOnTPTP.c
  Statistics.c
)
list(TRANSFORM JJPARSER_SOURCES PREPEND "${jjparser_SOURCE_DIR}/")

add_library(jjparser_for_tptp4X STATIC ${JJPARSER_SOURCES})
target_include_directories(jjparser_for_tptp4X PUBLIC "${jjparser_SOURCE_DIR}")
target_compile_definitions(jjparser_for_tptp4X PRIVATE TPTP_HOME="${TPTP_HOME}")
target_link_libraries(jjparser_for_tptp4X PUBLIC CURL::libcurl m)

################################################################################
# configuration of tptp4X

add_executable(tptp4X
  tptp4X.c
  AddEquality.c
  FOFify.c
  NumberNames.c
  Randomize.c
)

target_compile_definitions(tptp4X PRIVATE TPTP_HOME="${TPTP_HOME}")
target_link_libraries(tptp4X PRIVATE jjparser_for_tptp4X)

################################################################################
# gcc warning flags

set(TPTP4X_CFLAGS_WARNINGS
  -Wall
  -Wno-format-overflow
)

target_compile_options(jjparser_for_tptp4X PRIVATE ${TPTP4X_CFLAGS_WARNINGS})
target_compile_options(tptp4X PRIVATE ${TPTP4X_CFLAGS_WARNINGS})

################################################################################
# installation of the executable binary

install(TARGETS tptp4X RUNTIME DESTINATION "${TPTP_HOME}/ServiceTools")
