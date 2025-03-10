cmake_minimum_required(VERSION 3.10.2)
project(avoidance)

set(CMAKE_CXX_STANDARD 17)  # Use C++17 for compatibility

## Find necessary packages
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(std_msgs REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(mavros_msgs REQUIRED)
find_package(mavros REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(PCL REQUIRED)

# Find Gazebo Ignition (Ionic)
find_package(gz-sim9 REQUIRED)
find_package(gz-math8 REQUIRED)
find_package(gz-transport14 REQUIRED)
find_package(gz-msgs11 REQUIRED)

if(DISABLE_SIMULATION)
  message(STATUS "Building avoidance without Gazebo Simulation")
  add_definitions(-DDISABLE_SIMULATION)
else()
  message(STATUS "Building avoidance with Gazebo Simulation")
  find_package(yaml-cpp REQUIRED)
endif()

## Include directories
include_directories(
  include
  ${Eigen3_INCLUDE_DIRS}
  ${PCL_INCLUDE_DIRS}
)

## Define the avoidance library
set(AVOIDANCE_CPP_FILES
  src/common.cpp
  src/histogram.cpp
  src/transform_buffer.cpp
  src/avoidance_node.cpp
)

if(NOT DISABLE_SIMULATION)
  list(APPEND AVOIDANCE_CPP_FILES src/rviz_world_loader.cpp)
endif()

add_library(avoidance SHARED ${AVOIDANCE_CPP_FILES})

## Link dependencies
ament_target_dependencies(avoidance
  rclcpp
  tf2_ros
  sensor_msgs
  std_msgs
  geometry_msgs
  mavros_msgs
  Eigen3
  PCL
  gz-sim9
  gz-math8
  gz-transport14
  gz-msgs11
)

if(NOT DISABLE_SIMULATION)
  target_link_libraries(avoidance ${YAML_CPP_LIBRARIES})
endif()

## Install rules
install(TARGETS avoidance
  EXPORT export_avoidance
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)

install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION include/
  FILES_MATCHING PATTERN "*.h"
)

## Testing
if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()

ament_export_dependencies(
  rclcpp
  sensor_msgs
  geometry_msgs
  mavros_msgs
  tf2_ros
  gz-sim9
  gz-math8
  gz-transport14
  gz-msgs11
)

ament_package()
