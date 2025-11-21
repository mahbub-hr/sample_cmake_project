# CMakeLists.txt for ELF Binary Reader
cmake_minimum_required(VERSION 3.15)
project("<project name>" LANGUAGES C CXX)

# Ensure Clang is used (optional, but good practice if you expect Clang)
if (NOT "${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
    message(FATAL_ERROR "Please use Clang to compile ELFReader. Detected: ${CMAKE_CXX_COMPILER_ID}")
endif()

#include
include(ExternalProject)

# Set 
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF) # Use strict C++20

set(LLVM_DIR "<edit>/lib/cmake/llvm" CACHE PATH "Path to LLVM-Config.cmake" FORCE)
set(Clang_DIR "<edit>/lib/cmake/clang" CACHE PATH "Path to ClangConfig.cmake" FORCE)

#=============================
#        Packages            =
#=============================

# --- LLVM ---

find_package(LLVM 22.0 REQUIRED CONFIG)
message(STATUS "Found LLVM ${LLVM_PACKAGE_VERSION}, build type ${LLVM_BUILD_TYPE}")
separate_arguments(LLVM_DEFINITIONS_LIST NATIVE_COMMAND ${LLVM_DEFINITIONS})

llvm_map_components_to_libnames(llvm_libs
    Support
    Core
    Object
    BinaryFormat
)

# find_package(capstone CONFIG REQUIRED)
# find_package(Catch2 CONFIG REQUIRED)
# include(Catch)
# find_package(inja CONFIG REQUIRED)
# pkg_check_modules(KEYSTONE REQUIRED IMPORTED_TARGET keystone)
# find_package(nlohmann_json CONFIG REQUIRED)
# find_package(Boost 1.82.0 REQUIRED)
# include_directories(${Boost_INCLUDE_DIRS})

#=============================
# -- Add Libraries --=
#=============================

# -- LLVMInterface ---
add_library(LLVMInterface INTERFACE)
target_include_directories(LLVMInterface INTERFACE ${LLVM_INCLUDE_DIRS})
target_link_libraries(LLVMInterface INTERFACE ${llvm_libs})
target_compile_definitions(LLVMInterface INTERFACE ${LLVM_DEFINITIONS_LIST})
include_directories(${LLVM_INCLUDE_DIRS})

# -- Example Library ---
set(<example_lib> "<LIB Name>")
add_library(${example_lib} OBJECT
    
)

target_link_libraries(${example_lib} PUBLIC LLVMInterface)
target_include_directories(example_lib PUBLIC
    ${CMAKE_CURRENT_LIST_DIR}/include
)
# --- Compile Options ---
target_compile_options(${example_lib} PUBLIC
    -fno-rtti 
)

# define directives
target_compile_definitions(${example_lib} PUBLIC
    PROJECT_ROOT="${CMAKE_SOURCE_DIR}"
    S2EDIR="${CMAKE_SOURCE_DIR}"
)

# #Some #define directive for your code
# target_compile_definitions(example_lib PRIVATE <VAR_NAME>="$<TARGET_FILE:<LIB_or_Obj_Name>>")

# --- Define the executable target ---
set(<TARGET> "<EXE Name>")

add_executable(${TARGET}
    src/main.cpp
)

target_link_libraries(${TARGET} PRIVATE
    ${<EXAMPLE_LIB>}
)

#=============================
# -- Generic Compile Options --=
#=============================
add_compile_options(-Wall -pedantic)

# Build type specific settings (Debug vs. Release)
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    target_compile_definitions(${TARGET} PRIVATE DEBUG_BUILD _DEBUG)
    target_compile_options(${TARGET} PRIVATE -g -O0) # No optimization for better debugging
else()
    target_compile_definitions(${TARGET} PRIVATE NDEBUG)
    target_compile_options(${TARGET} PRIVATE -O2) # Moderate optimization
endif()

# Testing can be added later if needed.

# add_subdirectory(tools)

ExternalProject_Add( <name>
    SOURCE_DIR ${CMAKE_SOURCE_DIR}/<build dir>
    BINARY_DIR ${CMAKE_BINARY_DIR}/<build dir>      # Build inside top-level build/
    CMAKE_ARGS
        # -G Ninja
        -DCMAKE_TOOLCHAIN_FILE=/work/vcpkg/scripts/buildsystems/vcpkg.cmake
        -DCMAKE_C_COMPILER=clang-15
        -DCMAKE_CXX_COMPILER=clang++-15
        -DCMAKE_EXE_LINKER_FLAGS=-fuse-ld=lld-15
        -DLLVM_DIR=/opt/llvm-15/lib/cmake/llvm
        -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
        # -DCMAKE_INSTALL_PREFIX=${CMAKE_BINARY_DIR}/tools_install
    INSTALL_COMMAND ""
    BUILD_ALWAYS ON
    USES_TERMINAL_CONFIGURE ON
    USES_TERMINAL_BUILD ON
)
ExternalProject_Add_StepTargets(<name> build)  # Only generate 'build' target

#=============================
#       Dependencies         =
#=============================

add_dependencies(${TARGET} <Dependency Name>)
