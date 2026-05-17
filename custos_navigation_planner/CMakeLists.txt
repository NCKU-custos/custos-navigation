cmake_minimum_required(VERSION 3.16)
project(custos_navigation_planner)

if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 20)
endif()

find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(custos_common_msgs REQUIRED)
find_package(custos_navigation_msgs REQUIRED)
find_package(custos_perception_msgs REQUIRED)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()
