cmake_minimum_required(VERSION 3.5)
project(FlashMob LANGUAGES CXX)

# ===============================
# C++ STANDARD
# ===============================
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/third_party/graphvite/cmake)

# add externals
include(ExternalProject)
set(EXTERNAL_DIR ${PROJECT_SOURCE_DIR}/external)

if (NOT UPDATE)
    set(SKIP_UPDATE TRUE)
else()
    set(SKIP_UPDATE FALSE)
endif()

# import externals
include(cmake/glog.cmake)
include(cmake/gflags.cmake)
include(cmake/args.cmake)
include(cmake/gtest.cmake)

# enable ctest
include(cmake/test.cmake)

# ===============================
# OpenMP
# ===============================
find_package(OpenMP REQUIRED)
if (OPENMP_FOUND)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")
endif()

# ===============================
# Compilation flags (NO -std HERE)
# ===============================
set(CMAKE_CXX_FLAGS
    "${CMAKE_CXX_FLAGS} -Wall -Wno-maybe-uninitialized -mclflushopt"
)

# ===============================
# Profiling flags
# ===============================
if (PROFILE)
    if (PROFILE STREQUAL "ALL")
        set(PROFILE_LEVEL PROFILE_ALL)
    elseif (PROFILE STREQUAL "DETAIL")
        set(PROFILE_LEVEL PROFILE_DETAIL)
    elseif (PROFILE STREQUAL "NORMAL")
        set(PROFILE_LEVEL PROFILE_NORMAL)
    else ()
        set(PROFILE_LEVEL PROFILE_BRIEF)
    endif()
else ()
    set(PROFILE_LEVEL PROFILE_BRIEF)
endif()

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DPROFILE_LEVEL=${PROFILE_LEVEL}")

if (PROFILE_BF)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DPROFILE_BF")
endif()

if (ENABLE_PERF)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DENABLE_PERF")
elseif (ENABLE_VTUNE)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DENABLE_VTUNE")
endif()

message("Top level CMAKE_CXX_FLAGS: ${CMAKE_CXX_FLAGS}")
message("CMAKE_CXX_STANDARD: ${CMAKE_CXX_STANDARD}")

# ===============================
# Output directories
# ===============================
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/bin)

# ===============================
# Include paths
# ===============================
include_directories(include)
include_directories(third_party)

# ===============================
# Executable helper
# ===============================
function(add_exec EXEC_NAME)
    add_executable(${EXEC_NAME} ${EXEC_NAME}.cpp)
    target_link_libraries(${EXEC_NAME}
        PUBLIC
        ${OpenMP_CXX_LIBRARIES}
        ${GTEST_LIBRARIES}
        glog
        numa
    )
    add_dependencies(${EXEC_NAME}
        external-args
        external-gtest
        ${glog_dependency}
        ${gflag_dependency}
    )
endfunction()

add_subdirectory(src)

install(
    DIRECTORY bin
    DESTINATION ${CMAKE_BINARY_DIR}
    USE_SOURCE_PERMISSIONS
)
