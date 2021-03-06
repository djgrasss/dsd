project(dsd)
cmake_minimum_required(VERSION 2.6)

# use, i.e. don't skip the full RPATH for the build tree
set(CMAKE_SKIP_BUILD_RPATH  FALSE)

# when building, don't use the install RPATH already
# (but later on when installing)
set(CMAKE_BUILD_WITH_INSTALL_RPATH FALSE) 

SET(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")

# add the automatically determined parts of the RPATH
# which point to directories outside the build tree to the install RPATH
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

SET(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${PROJECT_SOURCE_DIR}/cmake/")

include(git_revision)
git_describe(GIT_TAG)

find_package(LibSndFile REQUIRED)
find_package(LibMbe REQUIRED)
find_package(ITPP REQUIRED)
find_package(LibPortAudio)

FILE(GLOB SRCS src/*.c src/*.cpp)
FILE(GLOB HEADERS include/*.h include/*.hpp)

configure_file("src/git_ver.c.in" "${CMAKE_CURRENT_BINARY_DIR}/git_ver.c" @ONLY)
list(APPEND SRCS "${CMAKE_CURRENT_BINARY_DIR}/git_ver.c")

INCLUDE_DIRECTORIES(
    "${PROJECT_SOURCE_DIR}/include"
    "${LIBSNDFILE_INCLUDE_DIR}"
    "${LIBMBE_INCLUDE_DIR}"
    "${ITPP_INCLUDE_DIR}"
)

SET(LIBS ${LIBS} ${LIBMBE_LIBRARY} ${LIBSNDFILE_LIBRARY} ${ITPP_LIBRARY})

if(PORTAUDIO_FOUND)
SET(INCLUDE_DIRECTORIES ${INCLUDE_DIRECTORIES} "${PORTAUDIO_INCLUDE_DIRS}")
SET(LIBS ${LIBS} ${PORTAUDIO_LIBRARIES})
add_definitions(-DUSE_PORTAUDIO)
endif(PORTAUDIO_FOUND)

ADD_EXECUTABLE(dsd ${SRCS} ${HEADERS})
TARGET_LINK_LIBRARIES(dsd ${LIBS})

install(TARGETS dsd DESTINATION bin)

# uninstall target
configure_file(
    "cmake/cmake_uninstall.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
    IMMEDIATE @ONLY)

add_custom_target(uninstall
    COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)

option(DISABLE_TEST "Disable building of test framework." OFF)

if (NOT DISABLE_TEST)
    add_subdirectory(test)
endif()
