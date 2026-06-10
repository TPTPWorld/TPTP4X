cmake_minimum_required(VERSION 3.18)

project(TPTP4X
  LANGUAGES C
)

include(FetchContent)

if(DEFINED ENV{TPTP_HOME})
  set(TPTP_HOME "$ENV{TPTP_HOME}" CACHE PATH "TPTP installation root")
else()
  set(TPTP_HOME "/home/tptp" CACHE PATH "TPTP installation root")
endif()

set(JJPARSER_GIT_TAG "49e46700b2e81b50516f35bbba7595f83cb27814" CACHE STRING "Git tag, branch, or commit to use for JJParser")

FetchContent_Declare(
  jjparser
  GIT_REPOSITORY https://github.com/TPTPWorld/JJParser.git
  GIT_TAG        "${JJPARSER_GIT_TAG}"
  SOURCE_SUBDIR  cmake-do-not-add
)

FetchContent_MakeAvailable(jjparser)

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
target_link_libraries(jjparser_tptp4x PUBLIC CURL::libcurl)

add_executable(tptp4X
  tptp4X.c
  AddEquality.c
  FOFify.c
  NumberNames.c
  Randomize.c
)

target_compile_definitions(tptp4X PRIVATE TPTP_HOME="${TPTP_HOME}")
target_link_libraries(tptp4X PRIVATE jjparser_tptp4x m)

if(CMAKE_C_COMPILER_ID MATCHES "GNU|Clang")
  target_compile_options(jjparser_tptp4x PRIVATE -Wall)
  target_compile_options(tptp4X PRIVATE -Wall)
endif()

if(CMAKE_C_COMPILER_ID STREQUAL "GNU")
  target_compile_options(jjparser_tptp4x PRIVATE -Wno-format-overflow)
  target_compile_options(tptp4X PRIVATE -Wno-format-overflow)
endif()

install(TARGETS tptp4X RUNTIME DESTINATION "${TPTP_HOME}/ServiceTools")
