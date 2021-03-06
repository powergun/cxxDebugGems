
set(Boost_USE_STATIC_LIBS ON)
include(${PROJECT_SOURCE_DIR}/vendor/boost_.cmake)
find_package(Boost 1.65 REQUIRED
    COMPONENTS
    # to experiment,
    # enable/disable backtrace or addr2line (and the cmake
    # target-linking option below)
    # then add/remove the corresponding definition before
    # including the boost (public) header
    # //////// OR ////////
    # use header-only mode and only define the DVARs
    # this way GDB can see the boost source code (single-stepping)
#    stacktrace_backtrace
#    stacktrace_addr2line
)
add_library(boost_static INTERFACE)
set_target_properties(boost_static
    PROPERTIES
    INTERFACE_INCLUDE_DIRECTORIES
    ${Boost_INCLUDE_DIRS})
target_link_libraries(boost_static
    INTERFACE
    ${Boost_LIBRARIES})

add_library(libcallee_boost STATIC
    callee.cpp)
set_target_properties(libcallee_boost
    PROPERTIES
    CXX_STANDARD 14
    POSITION_INDEPENDENT_CODE 1)
target_compile_definitions(libcallee_boost
    PRIVATE
#    "BOOST_STACKTRACE_LINK=1"
    "BOOST_STACKTRACE_USE_BACKTRACE=1"
#    "BOOST_STACKTRACE_USE_ADDR2LINE=1"
)
target_compile_options(
    libcallee_boost
    PRIVATE
    "-frecord-gcc-switches"
    "-fno-function-sections"
    "-fno-data-sections"
)
target_link_libraries(libcallee_boost
    PRIVATE
    boost_static

    # read this!
    # http://boostorg.github.io/stacktrace/stacktrace/configuration_and_build.html
    dl

    # dont forget to modify this line if the backend changes
    # from libbacktrace to addr2line
    # //////// NOTE ////////
    # I decide to go back to header-only mode so that I
    # can single-step in the boost code
    backtrace
)

add_executable(caller_boost
    caller.cpp)
set_target_properties(caller_boost
    PROPERTIES
    CXX_STANDARD 14)
target_compile_options(caller_boost
    PUBLIC
    "-frecord-gcc-switches"
    "-fno-function-sections"
    "-fno-data-sections")
target_link_libraries(caller_boost PUBLIC
    libcallee_boost
)
