cmake_minimum_required(VERSION 2.8.3)
project(warehouse_ros_mongo)

set(CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake/" ${CMAKE_MODULE_PATH})

find_package(catkin REQUIRED
  roscpp
  rostest
  rostime
  std_msgs
  warehouse_ros
)
find_package(Boost COMPONENTS system filesystem thread)
find_package(OpenSSL)
find_package(MongoDB)

set(MONGO_EXPORT)
if("${MongoDB_LIBRARIES}" MATCHES "\\.so$")
  set(MONGO_EXPORT MongoDB)
endif()

file(MAKE_DIRECTORY "${CATKIN_DEVEL_PREFIX}/include")

catkin_python_setup()

catkin_package(
  INCLUDE_DIRS ${CATKIN_DEVEL_PREFIX}/include include
  LIBRARIES warehouse_ros_mongo
  CATKIN_DEPENDS roscpp rostime std_msgs
  DEPENDS Boost ${MONGO_EXPORT}
)

if (NOT MongoDB_EXPOSE_MACROS)
  add_definitions(-DMONGO_EXPOSE_MACROS)
endif()
configure_file("include/warehouse_ros_mongo/config.h.in" "${CATKIN_DEVEL_PREFIX}/include/warehouse_ros_mongo/config.h")

include_directories(${CATKIN_DEVEL_PREFIX}/include include ${catkin_INCLUDE_DIRS} ${MongoDB_INCLUDE_DIR} ${Boost_INCLUDE_DIRS})
link_directories(${catkin_LINK_DIRS})
link_directories(${Boost_LIBRARY_DIRS})

set(warehouse_srcs
  src/warehouse_ros_mongo_dummy.cpp
  src/query_results.cpp
  src/message_collection.cpp
  src/database_connection.cpp)

add_library(warehouse_ros_mongo SHARED ${warehouse_srcs})

target_link_libraries(warehouse_ros_mongo ${catkin_LIBRARIES} ${MongoDB_LIBRARIES} ${OPENSSL_LIBRARIES} ${Boost_LIBRARIES})
catkin_add_gtest(test_warehouse_ros_mongo_cpp test/test_warehouse_ros_mongo.cpp)
target_link_libraries(test_warehouse_ros_mongo_cpp warehouse_ros_mongo)

install(TARGETS warehouse_ros_mongo LIBRARY DESTINATION lib)
install(DIRECTORY include/ DESTINATION include FILES_MATCHING PATTERN "*.h" PATTERN "*.hpp")
install(PROGRAMS src/mongo_wrapper_ros.py DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})
install(
  FILES
    "${CATKIN_DEVEL_PREFIX}/include/warehouse_ros_mongo/config.h"
    mongo_database_connection_plugin_description.xml
  DESTINATION
    include/warehouse_ros_mongo)
