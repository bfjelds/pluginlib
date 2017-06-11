cmake_minimum_required(VERSION 2.8.3)
project(pluginlib)

# Add support for C++11
if(NOT WIN32)
  add_definitions(-std=c++11)
endif()

find_package(ament_cmake REQUIRED)
find_package(class_loader REQUIRED)
find_package(rospack REQUIRED)
find_package(cmake_modules REQUIRED)

set(INCLUDE_DIRS include ${ament_cmake_INCLUDE_DIRS}
  ${class_loader_INCLUDE_DIRS} ${rospack_INCLUDE_DIRS}
  ${cmake_modules_INCLUDE_DIRS})
include_directories(${INCLUDE_DIRS})

set(LIBRARY_DIRS ${ament_cmake_LIBRARY_DIRS} ${class_loader_LIBRARY_DIRS}
  ${rospack_LIBRARY_DIRS} ${cmake_modules_LIBRARY_DIRS})

link_directories(${LIBRARY_DIRS})

set(LIBS ${ament_cmake_LIBRARIES} ${class_loader_LIBRARIES}
  ${rospack_LIBRARIES} ${cmake_modules_LIBRARIES} dl)

find_package(Boost REQUIRED COMPONENTS filesystem system)
find_package(TinyXML REQUIRED)

add_executable(plugin_tool src/plugin_tool.cpp)
target_link_libraries(plugin_tool ${Boost_LIBRARIES} ${LIBS}
  ${TinyXML_LIBRARIES})

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

if(CATKIN_ENABLE_TESTING)
  add_library(test_plugins EXCLUDE_FROM_ALL SHARED test/test_plugins.cpp)

  catkin_add_gtest(${PROJECT_NAME}_utest test/utest.cpp)
  if(TARGET ${PROJECT_NAME}_utest)
    target_link_libraries(${PROJECT_NAME}_utest ${TinyXML_LIBRARIES} ${LIBS}
      ${Boost_LIBRARIES})
    add_dependencies(${PROJECT_NAME}_utest test_plugins)
  endif()
endif()

ament_export_dependencies(ament_cmake)
ament_export_dependencies(class_loader)
ament_export_dependencies(rospack)
ament_export_dependencies(cmake_modules)
ament_export_include_directories(${INCLUDE_DIRS})
ament_export_libraries(test_plugins)

ament_package()
