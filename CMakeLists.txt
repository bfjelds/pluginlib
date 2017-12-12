cmake_minimum_required(VERSION 3.5)
project(pluginlib)

# Add support for C++11
if(NOT WIN32)
  add_definitions(-std=c++11)
endif()

find_package(ament_cmake REQUIRED)
find_package(class_loader REQUIRED)
find_package(rospack REQUIRED)
find_package(ros2_console REQUIRED)
find_package(cmake_modules REQUIRED)

set(Boost_USE_STATIC_LIBS ON)
find_package(Boost REQUIRED COMPONENTS filesystem system)
find_package(TinyXML REQUIRED HINTS "*.lib") # add hint seems to allow debug lib to be found when appropriate

set(INCLUDE_DIRS include ${ament_cmake_INCLUDE_DIRS}
  ${class_loader_INCLUDE_DIRS} ${rospack_INCLUDE_DIRS}
  ${ros2_console_INCLUDE_DIRS} ${cmake_modules_INCLUDE_DIRS}
  ${Boost_INCLUDE_DIRS} ${TinyXML_INCLUDE_DIRS})
include_directories(${INCLUDE_DIRS})

set(LIBRARY_DIRS ${ament_cmake_LIBRARY_DIRS} ${class_loader_LIBRARY_DIRS}
  ${rospack_LIBRARY_DIRS} ${ros2_console_LIBRARY_DIRS} ${cmake_modules_LIBRARY_DIRS}
  ${Boost_LIBRARY_DIRS} ${TinyXML_LIBRARY_DIRS})

link_directories(${LIBRARY_DIRS})

if(NOT WIN32)
set(LIBS dl ${ament_cmake_LIBRARIES} ${class_loader_LIBRARIES}
  ${rospack_LIBRARIES} ${ros2_console_LIBRARIES} ${cmake_modules_LIBRARIES}
  ${Boost_LIBRARIES} ${TinyXML_LIBRARIES})
else()
set(LIBS ${ament_cmake_LIBRARIES} ${class_loader_LIBRARIES}
  ${rospack_LIBRARIES} ${ros2_console_LIBRARIES} ${cmake_modules_LIBRARIES}
  ${Boost_LIBRARIES} ${TinyXML_LIBRARIES})
endif()

add_executable(plugin_tool src/plugin_tool.cpp)
target_link_libraries(plugin_tool ${LIBS})

install(TARGETS plugin_tool
  RUNTIME
  DESTINATION lib/${PROJECT_NAME}
  ARCHIVE
  DESTINATION lib
  LIBRARY
  DESTINATION lib)

install(DIRECTORY include/pluginlib/
  DESTINATION include/${PROJECT_NAME})

install(FILES share/pluginlib/typed_class_loader_template.cpp
  DESTINATION share/${PROJECT_NAME})

install(PROGRAMS scripts/plugin_macro_update
  DESTINATION lib/${PROJECT_NAME})

install(FILES test/test_plugins.xml DESTINATION share/${PROJECT_NAME}/test)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
  
  add_library(test_plugins EXCLUDE_FROM_ALL SHARED test/test_plugins.cpp)

  ament_add_gtest(${PROJECT_NAME}_utest test/utest.cpp)
  if(TARGET ${PROJECT_NAME}_utest)
    target_link_libraries(${PROJECT_NAME}_utest ${LIBS})
    add_dependencies(${PROJECT_NAME}_utest test_plugins)
  endif()
endif()

ament_export_dependencies(boost)
ament_export_dependencies(ament_cmake)
ament_export_dependencies(class_loader)
ament_export_dependencies(rospack)
ament_export_dependencies(ros2_console)
ament_export_dependencies(cmake_modules)
ament_export_include_directories(${INCLUDE_DIRS})
ament_export_libraries(${LIBS})

ament_package()
