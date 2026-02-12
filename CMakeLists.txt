cmake_minimum_required(VERSION 3.10)

project(jit-jss-tabu-search VERSION 1.0)

# C++ Standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Modules path for CPLEX
set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)

# --- Dependencies ---

# CPLEX and its components
find_package(CPLEX REQUIRED)

# Boost
set(Boost_NO_BOOST_CMAKE ON)
find_package(Boost REQUIRED COMPONENTS program_options timer chrono)

# --- Library Target ---

# Define the core logic as a library
add_library(jit_jss_core
    src/InitialSolution.cpp
    src/Instance.cpp
    src/JumpList.cpp
    src/Neighborhoods.cpp
    src/Poset.cpp
    src/Schedulers.cpp
    src/State.cpp
    src/Tabu.cpp
    src/TabuList.cpp
    src/TabuTrio.cpp
    src/Utilities.cpp
)

# Target Link Libraries
target_link_libraries(jit_jss_core PUBLIC 
    ilocplex
    cplex-cp
    ${Boost_LIBRARIES}
    rt
    pthread
    dl
)

target_include_directories(jit_jss_core PUBLIC src)

# Compile definitions and flags
target_compile_definitions(jit_jss_core PUBLIC IL_STD)
target_compile_options(jit_jss_core PRIVATE -Wall -Wunused -Wno-unknown-pragmas -fPIC -fexceptions)

# --- Executable Target ---

# Main application
add_executable(main apps/main.cpp)
target_link_libraries(main PRIVATE jit_jss_core)

# --- Build Types Configuration ---

# Debug flags
target_compile_options(jit_jss_core PUBLIC $<$<CONFIG:Debug>:-g -ggdb>)

# Release flags
target_compile_options(jit_jss_core PUBLIC $<$<CONFIG:Release>:-O3 -march=native -mtune=native>)
