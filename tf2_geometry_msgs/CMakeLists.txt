cmake_minimum_required(VERSION 3.5)
project(tf2_geometry_msgs)

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()
if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic -Wnon-virtual-dtor -Woverloaded-virtual)
endif()

find_package(geometry_msgs REQUIRED)
find_package(orocos_kdl REQUIRED)
find_package(tf2 REQUIRED)
find_package(tf2_ros REQUIRED)

ament_python_install_package(${PROJECT_NAME}
  PACKAGE_DIR src/${PROJECT_NAME})

add_library(${PROJECT_NAME} INTERFACE)
target_include_directories(${PROJECT_NAME} INTERFACE
  "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
  "$<INSTALL_INTERFACE:include>")
target_link_libraries(${PROJECT_NAME} INTERFACE
  ${geometry_msgs_TARGETS}
  orocos-kdl
  tf2::tf2
  tf2_ros::tf2_ros)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)

  list(APPEND AMENT_LINT_AUTO_EXCLUDE
    ament_cmake_cppcheck
  )
  ament_lint_auto_find_test_dependencies()

  ament_cppcheck(LANGUAGE "c++")

  find_package(ament_cmake_gtest REQUIRED)
  find_package(rclcpp REQUIRED)

  find_package(ament_cmake_pytest REQUIRED)
  ament_add_pytest_test(test_tf2_geometry_msgs_py test/test_tf2_geometry_msgs.py)

  ament_add_gtest(test_tf2_geometry_msgs test/test_tf2_geometry_msgs.cpp)
  if(TARGET test_tf2_geometry_msgs)
    target_link_libraries(test_tf2_geometry_msgs
      ${PROJECT_NAME}
      rclcpp::rclcpp
      # Used, but not linked to test ${PROJECT_NAME}'s exports:
      #   tf2_ros::tf2_ros
    )
  endif()
endif()

install(TARGETS ${PROJECT_NAME} EXPORT export_${PROJECT_NAME})
install(DIRECTORY include/ DESTINATION include)

ament_export_targets(export_${PROJECT_NAME})
ament_export_dependencies(
  "geometry_msgs"
  "orocos_kdl"
  "tf2"
  "tf2_ros")

ament_package()
