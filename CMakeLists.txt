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
# download JJParser at CMake configuration time
# but ignore JJParser's own CMake project

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
    > If the top directory of the populated content contains a CMakeLists.txt file,
    > call add_subdirectory() to add it to the main build.
    > It is not an error for there to be no CMakeLists.txt file,
    > which allows the command to be used for dependencies
    > that make downloaded content available at a known location,
    > but which do not need or support being added directly to the build.
  #]]
)

FetchContent_MakeAvailable(jjparser)

################################################################################
# configuration of JJParser (linked to curl and math library)

find_package(CURL REQUIRED)

set(JJPARSER_SOURCES
  "${jjparser_SOURCE_DIR}/ListStatistics.c"
  "${jjparser_SOURCE_DIR}/List.c"
  "${jjparser_SOURCE_DIR}/Examine.c"
  "${jjparser_SOURCE_DIR}/Modify.c"
  "${jjparser_SOURCE_DIR}/Tree.c"
  "${jjparser_SOURCE_DIR}/TreeStatistics.c"
  "${jjparser_SOURCE_DIR}/ParseTPTP.c"
  "${jjparser_SOURCE_DIR}/ParseTSTP.c"
  "${jjparser_SOURCE_DIR}/PrintTSTP.c"
  "${jjparser_SOURCE_DIR}/Parsing.c"
  "${jjparser_SOURCE_DIR}/Signature.c"
  "${jjparser_SOURCE_DIR}/Tokenizer.c"
  "${jjparser_SOURCE_DIR}/Utilities.c"
  "${jjparser_SOURCE_DIR}/Compare.c"
  "${jjparser_SOURCE_DIR}/Interpret.c"
  "${jjparser_SOURCE_DIR}/PrintXML.c"
  "${jjparser_SOURCE_DIR}/PrintSMT2.c"
  "${jjparser_SOURCE_DIR}/PrintDFG.c"
  "${jjparser_SOURCE_DIR}/PrintOtter.c"
  "${jjparser_SOURCE_DIR}/PrintKIF.c"
  "${jjparser_SOURCE_DIR}/PrintSUMO.c"
  "${jjparser_SOURCE_DIR}/PrintLP.c"
  "${jjparser_SOURCE_DIR}/PrintDK.c"
  "${jjparser_SOURCE_DIR}/FileUtilities.c"
  "${jjparser_SOURCE_DIR}/SystemOnTPTP.c"
  "${jjparser_SOURCE_DIR}/Statistics.c"
)

add_library(jjparser_tptp4x STATIC ${JJPARSER_SOURCES})
target_include_directories(jjparser_tptp4x PUBLIC "${jjparser_SOURCE_DIR}")
target_compile_definitions(jjparser_tptp4x PRIVATE TPTP_HOME="${TPTP_HOME}")
target_link_libraries(jjparser_tptp4x PUBLIC CURL::libcurl m)

################################################################################

add_executable(tptp4X
  tptp4X.c
  AddEquality.c
  FOFify.c
  NumberNames.c
  Randomize.c
)

target_compile_definitions(tptp4X PRIVATE TPTP_HOME="${TPTP_HOME}")
target_link_libraries(tptp4X PRIVATE jjparser_tptp4x)

################################################################################

set(TPTP4X_CFLAGS_WARNINGS
  -Wall
  -Wno-format-overflow
)

target_compile_options(jjparser_tptp4x PRIVATE ${TPTP4X_CFLAGS_WARNINGS})
target_compile_options(tptp4X PRIVATE ${TPTP4X_CFLAGS_WARNINGS})

################################################################################

install(TARGETS tptp4X RUNTIME DESTINATION "${TPTP_HOME}/ServiceTools")
