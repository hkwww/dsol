cmake_minimum_required(VERSION 3.15)
project(dsol)

set(CMAKE_CXX_STANDARD 17)
list(PREPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

option(BUILD_ROS "Build ros" ON)
option(BUILD_PLAY "Build play" OFF)
option(BUILD_TESTING "Build tests" OFF)
option(BUILD_BENCHMARK "Build benchmarks" OFF)
option(BUILD_VECTORIZE "Build with -march=native" ON)

set(CC_TARGET_PREFIX sv)
include(CMakeHelpers)

add_library(sv_options INTERFACE)
include(CompilerWarnings)
enable_warnings(sv_options)
include(Sanitizers)
enable_sanitizers(sv_options)

find_package(
  catkin QUIET
  COMPONENTS roscpp
             tf2_ros
             tf2_eigen
             cv_bridge
             image_transport
             nav_msgs
             sensor_msgs
             visualization_msgs)


find_package(TBB REQUIRED)
find_package(fmt REQUIRED)
find_package(glog REQUIRED)
find_package(absl REQUIRED)
find_package(Boost REQUIRED) # circular buffer

find_package(Eigen3 REQUIRED)
find_package(Sophus REQUIRED)
find_package(OpenCV REQUIRED)
find_package(xtensor) # reading npy

find_package(GTest REQUIRED)
find_package(benchmark REQUIRED)

include_directories(${catkin_INCLUDE_DIRS})
include_directories(include)


if(BUILD_ROS)
  catkin_package()
endif()



# Compile Lib

file(GLOB_RECURSE
    utils_files
    ${PROJECT_SOURCE_DIR}/src/utils/*.cpp
    ${PROJECT_SOURCE_DIR}/src/ros1/*.cpp
)

add_library (
    dsol_utils
    SHARED
    ${utils_files}
)
target_link_libraries(dsol_utils
    ${catkin_LIBRARIES}
    ${OpenCV_LIBRARIES}
    glog::glog
    absl::flat_hash_map
    absl::time
    TBB::tbb
    fmt::fmt
    GTest::GTest
)

file(GLOB_RECURSE
    core_files
    ${PROJECT_SOURCE_DIR}/src/dsol/*.cpp
)

add_library (
    dsol_core
    SHARED
    ${core_files}
)
target_link_libraries(dsol_core
    dsol_utils
    absl::span
)


# Compile Node

add_executable(dsol_odom_node app/node_odom.cpp)
target_link_libraries(dsol_odom_node  dsol_core dsol_utils absl::flags_parse )

add_executable(dsol_data_node app/node_data.cpp)
target_link_libraries(dsol_data_node  dsol_core dsol_utils absl::flags_parse )