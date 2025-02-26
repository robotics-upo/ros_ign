cmake_minimum_required(VERSION 3.5)

project(ros_ign_image)

# Default to C++17
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 17)
endif()
if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra)
endif()

find_package(catkin REQUIRED COMPONENTS
  image_transport
  mav_msgs
  ros_ign_bridge
  roscpp
  sensor_msgs)

# Default to Dome, support Citadel and Blueprint
if ("$ENV{IGNITION_VERSION}" STREQUAL "blueprint")
  find_package(ignition-transport7 REQUIRED)
  set(IGN_TRANSPORT_VER ${ignition-transport7_VERSION_MAJOR})

  find_package(ignition-msgs4 REQUIRED)
  set(IGN_MSGS_VER ${ignition-msgs4_VERSION_MAJOR})

  message(STATUS "Compiling against Ignition Blueprint")
elseif("$ENV{IGNITION_VERSION}" STREQUAL "citadel")
  find_package(ignition-transport8 QUIET)
  set(IGN_TRANSPORT_VER ${ignition-transport8_VERSION_MAJOR})

  find_package(ignition-msgs5 REQUIRED)
  set(IGN_MSGS_VER ${ignition-msgs5_VERSION_MAJOR})

  message(STATUS "Compiling against Ignition Citadel")
elseif("$ENV{IGNITION_VERSION}" STREQUAL "edifice")
  find_package(ignition-transport10 QUIET)
  set(IGN_TRANSPORT_VER ${ignition-transport10_VERSION_MAJOR})

  find_package(ignition-msgs7 REQUIRED)
  set(IGN_MSGS_VER ${ignition-msgs7_VERSION_MAJOR})

  message(STATUS "Compiling against Ignition Edifice")
elseif("$ENV{IGNITION_VERSION}" STREQUAL "fortress")
  find_package(ignition-transport11 QUIET)
  set(IGN_TRANSPORT_VER ${ignition-transport11_VERSION_MAJOR})

  find_package(ignition-msgs8 REQUIRED)
  set(IGN_MSGS_VER ${ignition-msgs8_VERSION_MAJOR})

  message(STATUS "Compiling against Ignition Fortress")
else()
  find_package(ignition-transport11 QUIET)
  set(IGN_TRANSPORT_VER ${ignition-transport10_VERSION_MAJOR})

  find_package(ignition-msgs8 REQUIRED)
  set(IGN_MSGS_VER ${ignition-msgs7_VERSION_MAJOR})

  message(STATUS "Compiling against Ignition Fortress")
endif()

catkin_package()

include_directories(
  ${catkin_INCLUDE_DIRS}
)

set(executable
  image_bridge
)

add_executable(${executable}
  src/image_bridge.cpp
)
target_link_libraries(${executable}
  ${catkin_LIBRARIES}
  ignition-msgs${IGN_MSGS_VER}::core
  ignition-transport${IGN_TRANSPORT_VER}::core
)
install(TARGETS ${executable}
        DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

# Tests
find_package(rostest REQUIRED)

set(test_publishers
  ign_publisher
)

set(test_subscribers
  ros_subscriber
)

foreach(test_publisher ${test_publishers})
  add_executable(${test_publisher}_image
    test/publishers/${test_publisher}.cpp
  )
  target_link_libraries(${test_publisher}_image
    ${catkin_LIBRARIES}
    ignition-msgs${IGN_MSGS_VER}::core
    ignition-transport${IGN_TRANSPORT_VER}::core
    gtest
    gtest_main
  )
endforeach(test_publisher)

foreach(test_subscriber ${test_subscribers})
  add_rostest_gtest(test_${test_subscriber}_image
    test/${test_subscriber}.test
    test/subscribers/${test_subscriber}.cpp)
  target_link_libraries(test_${test_subscriber}_image
    ${catkin_LIBRARIES}
    ignition-msgs${IGN_MSGS_VER}::core
    ignition-transport${IGN_TRANSPORT_VER}::core
  )
endforeach(test_subscriber)

