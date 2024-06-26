cmake_minimum_required(VERSION 3.12)
#set(CMAKE_BUILD_TYPE "Debug")

MESSAGE(STATUS "Build type: " ${CMAKE_BUILD_TYPE})

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

project(simcem)

# We need to use EXCLUDE_FROM_ALL to prevent these being installed.
add_subdirectory(${PROJECT_SOURCE_DIR}/src/pybind11 EXCLUDE_FROM_ALL)
add_subdirectory(${PROJECT_SOURCE_DIR}/submodule/sundials EXCLUDE_FROM_ALL)

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake" ${CMAKE_MODULE_PATH})
find_package(IPOPT 3.11.9 REQUIRED)
find_package(Eigen3 3.2 REQUIRED)
find_package(Boost REQUIRED)

message(STATUS "CMakeLists: Checking ${IPOPT_INCLUDE_DIRS}")

include_directories(src/stator src/simcem src submodule/sundials/include)
add_library(simcem STATIC src/simcem/simcem/simcem.cpp)

target_include_directories(simcem PUBLIC src/simcem/ ${EIGEN3_INCLUDE_DIR} ${IPOPT_INCLUDE_DIRS} ${Boost_INCLUDE_DIRS})
target_link_libraries(simcem PUBLIC PRIVATE ${IPOPT_LIBRARIES} ${IPOPT_LINK_FLAGS})
set_target_properties(simcem PROPERTIES POSITION_INDEPENDENT_CODE True)

pybind11_add_module(core pysrc/simcem/core.cpp)
target_link_libraries(core PRIVATE simcem)
target_link_libraries(core PUBLIC PRIVATE ${IPOPT_LIBRARIES} ${IPOPT_LINK_FLAGS})

pybind11_add_module(kiln pysrc/simcem/kiln.cpp)
target_link_libraries(kiln PRIVATE simcem SUNDIALS::ida_static ${IPOPT_LIBRARIES} ${IPOPT_LINK_FLAGS})
#target_include_directories(kiln PUBLIC ${SUNDIALS_INCLUDE_DIRS})
