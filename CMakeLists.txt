# these plugins should be uploaded to upstream repository
# https://github.com/ros-visualization/rviz/pull/634

# http://ros.org/doc/groovy/api/catkin/html/user_guide/supposed.html
cmake_minimum_required(VERSION 2.8.3)
project(jsk_rviz_plugins_fork)
if("$ENV{ROS_DISTRO}" STRGREATER "melodic")
  add_compile_options(-std=c++14)
else()
  add_compile_options(-std=c++11)
endif()

find_package(PCL 1.8.1 REQUIRED COMPONENTS common io)

# Load catkin and all dependencies required for this package
# TODO: remove all from COMPONENTS that are not catkin packages.
find_package(catkin REQUIRED COMPONENTS rviz jsk_hark_msgs jsk_footstep_msgs jsk_recognition_utils
#   people_msgs
  message_generation std_msgs diagnostic_msgs cv_bridge
  jsk_topic_tools
#   image_geometry
#   view_controller_msgs
#   geometry_msgs
#   dynamic_reconfigure
#   jsk_recognition_msgs
#   jsk_gui_msgs
  message_filters
  pcl_ros
)

# Use ccache if installed to make it fast to generate object files
if (CMAKE_VERSION VERSION_LESS 3.4)
  find_program(CCACHE_FOUND ccache)
  if(CCACHE_FOUND)
    set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE ccache)
    set_property(GLOBAL PROPERTY RULE_LAUNCH_LINK ccache)
  endif(CCACHE_FOUND)
endif()

# add_message_files(FILES
#   OverlayText.msg OverlayMenu.msg TransformableMarkerOperate.msg ObjectFitCommand.msg
#   Pictogram.msg PictogramArray.msg RecordCommand.msg StringStamped.msg)
# add_service_files(FILES
#   RequestMarkerOperate.srv EusCommand.srv
#   Screenshot.srv
#   )
# catkin_python_setup()
# generate_dynamic_reconfigure_options(
#   cfg/ClassificationResultVisualizer.cfg
#   cfg/ContactStateMarker.cfg
#   cfg/OverlayTextInterface.cfg)
# generate_messages(DEPENDENCIES std_msgs geometry_msgs)

catkin_package(
    DEPENDS rviz
    CATKIN_DEPENDS jsk_hark_msgs jsk_footstep_msgs
    jsk_recognition_utils cv_bridge people_msgs image_geometry
    INCLUDE_DIRS # TODO include
    LIBRARIES ${PROJECT_NAME}
)


set(ROS_BUILD_TYPE Release)

include_directories(src ${Boost_INCLUDE_DIR} ${catkin_INCLUDE_DIRS})

# TODO: fill in what other packages will need to use this package
## LIBRARIES: libraries you create in this project that dependent projects also need
## CATKIN_DEPENDS: catkin_packages dependent projects also need
## DEPENDS: system dependencies of this project that dependent projects also need

if(rviz_QT_VERSION VERSION_LESS "5")
  find_package(Qt4 COMPONENTS QtCore QtGui REQUIRED)
  include(${QT_USE_FILE})
else()
  find_package(Qt5Widgets REQUIRED)
endif()
add_definitions(-DQT_NO_KEYWORDS -g)

include_directories(${CMAKE_CURRENT_BINARY_DIR})

#set(SOURCE_FILES src/ambient_sound_display.cpp src/ambient_sound_visual.cpp)

set(HEADER_FILES
  src/polygon_array_display.h
)

if(rviz_QT_VERSION VERSION_LESS "5")
  qt4_wrap_cpp(MOC_FILES ${HEADER_FILES})
else()
  qt5_wrap_cpp(MOC_FILES ${HEADER_FILES})
endif()

set(SOURCE_FILES
  src/polygon_array_display.cpp
  ${MOC_FILES}
)
# if(jsk_recognition_msgs_VERSION VERSION_LESS 1.2.15)
#   message(WARNING "jsk_recognition_msgs ${jsk_recognition_msgs_VERSION} < 1.2.15 does not support human_skeleton_array_display.cpp")
#   list(REMOVE_ITEM HEADER_FILES src/human_skeleton_array_display.h)
#   list(REMOVE_ITEM SOURCE_FILES src/human_skeleton_array_display.cpp)
#   list(REMOVE_ITEM SOURCE_FILES  ${PROJECT_BINARY_DIR}/src/moc_human_skeleton_array_display.cxx)
# endif()

add_library(jsk_rviz_plugins_fork ${SOURCE_FILES} ${UIC_FILES})
if(rviz_QT_VERSION VERSION_LESS "5")
  target_link_libraries(jsk_rviz_plugins_fork ${QT_LIBRARIES} ${catkin_LIBRARIES} ${rviz_DEFAULT_PLUGIN_LIBRARIES})
else()
  target_link_libraries(jsk_rviz_plugins_fork Qt5::Widgets ${catkin_LIBRARIES} ${rviz_DEFAULT_PLUGIN_LIBRARIES})
endif()
add_dependencies(jsk_rviz_plugins_fork
#   jsk_footstep_msgs_generate_messages_cpp
#   jsk_gui_msgs_generate_messages_cpp
#   jsk_hark_msgs_generate_messages_cpp
#   jsk_recognition_msgs_generate_messages_cpp
#   people_msgs_generate_messages_cpp
#   view_controller_msgs_generate_messages_cpp
  ${PROJECT_NAME}_generate_messages_cpp
  ${catkin_EXPORTED_TARGETS})

# if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
#   set_target_properties(jsk_rviz_plugins_fork PROPERTIES LINK_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -z defs")
# endif()

# From https://github.com/jsk-ros-pkg/jsk_recognition/pull/2345
# Install header files directly into ${CATKIN_PACKAGE_INCLUDE_DESTINATION}.
# If the package has setup.py and modules under src/${PROJECT_NAME}/,
# install python executables directly into ${CATKIN_PACKAGE_BIN_DESTINATION}.
# However, if it doesn't have setup.py, directories including python executables
# should be installed recursively into ${CATKIN_PACKAGE_SHARE_DESTINATION}.
# Also, other directories like 'launch' or 'sample' must be installed
# recursively into ${CATKIN_PACKAGE_SHARE_DESTINATION}.
# Be careful that 'launch' and 'launch/' are different: the former is directory
# and the latter is each content.
install(TARGETS jsk_rviz_plugins_fork
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
)

# install(DIRECTORY scripts/
#   DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
#   USE_SOURCE_PERMISSIONS
# )

install(FILES plugin_description.xml
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)
# install(DIRECTORY cfg config 
#   DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
#   USE_SOURCE_PERMISSIONS
# )

# install(DIRECTORY src/
#   DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
#   FILES_MATCHING PATTERN "*.h"
# )

# if (CATKIN_ENABLE_TESTING)
#   # download and install sample data
#   add_custom_target(${PROJECT_NAME}_install_sample_data COMMAND python$ENV{ROS_PYTHON_VERSION} ${PROJECT_SOURCE_DIR}/test/install_sample_data.py)
#   add_dependencies(tests ${PROJECT_NAME}_install_sample_data)
#   # run rostest for melodic+
#   find_package(rostest REQUIRED)
#   if("$ENV{ROS_DISTRO}" STRGREATER "indigo") # support from kinetic
#     file(GLOB TEST_FILES test/*.test test/*.xml)
#     foreach(TEST_FILE ${TEST_FILES})
#       # if we failed to install sample data, skip testing face_detect and normal
#       if((NOT EXISTS "${PROJECT_SOURCE_DIR}/test/data/2017-06-20-12-00-00_people_images_in_lab.bag") AND
# 	  ((${TEST_FILE} STREQUAL "${PROJECT_SOURCE_DIR}/test/face_detector.xml") OR
# 	    (${TEST_FILE} STREQUAL "${PROJECT_SOURCE_DIR}/test/normal.xml")))
# 	message(STATUS "Skip test for ${TEST_FILE} due to missing bag file")
# 	continue()
#       endif()
#       message(STATUS "Run test for ${TEST_FILE}")
#       add_rostest(${TEST_FILE})
#     endforeach()
#   endif()
# endif()
