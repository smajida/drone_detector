cmake_minimum_required(VERSION 3.1)
project(drone_detector)

FILE(GLOB_RECURSE HEADER_FILES include *.h)
FILE(GLOB_RECURSE OPENCL_FILES include *.cl)
FILE(GLOB_RECURSE LAUNCHER_FILES include *.launch)
FILE(GLOB_RECURSE YAML_FILES include *.yaml)

## Find catkin macros and libraries
## if COMPONENTS list like find_package(catkin REQUIRED COMPONENTS xyz)
## is used, also find other catkin packages
find_package(catkin REQUIRED COMPONENTS
  cv_bridge
  image_transport
  roscpp
  sensor_msgs
  std_msgs
  tf
)

find_package(Eigen3)
if(NOT EIGEN3_FOUND)
  # Fallback to cmake_modules
  find_package(cmake_modules REQUIRED)
  find_package(Eigen REQUIRED)
  set(EIGEN3_INCLUDE_DIRS ${EIGEN_INCLUDE_DIRS})
  set(EIGEN3_LIBRARIES ${EIGEN_LIBRARIES})
endif()

find_package(OpenCV REQUIRED)
find_package(OpenCL REQUIRED)
message("${OpenCL_INCLUDE_DIRS} ${OpenCL_LIBRARIES}")
################################################
## Declare ROS messages, services and actions ##
################################################
## Generate messages in the 'msg' folder
# add_message_files(
#   FILES
#   cross_pos.msg
# )

## Generate added messages and services with any dependencies listed here
# generate_messages(
#   DEPENDENCIES
#   std_msgs
# )

################################################
## Declare ROS dynamic reconfigure parameters ##
################################################

## Generate dynamic reconfigure parameters in the 'cfg' folder
# generate_dynamic_reconfigure_options(
#   cfg/DynReconf1.cfg
#   cfg/DynReconf2.cfg
# )

###################################
## catkin specific configuration ##
###################################

catkin_package(
   INCLUDE_DIRS include
   LIBRARIES ${PROJECT_NAME} SparseOptFlowOcl
   CATKIN_DEPENDS cv_bridge image_transport roscpp sensor_msgs std_msgs message_runtime
#  DEPENDS system_lib
   DEPENDS Eigen
)


###########
## Build ##
###########

## Specify additional locations of header files
## Your package locations should be listed before other locations
# include_directories(include)
include_directories(
  include
  ${catkin_INCLUDE_DIRS}
  ${OpenCV_INCLUDE_DIRS}
  ${OpenCL_INCLUDE_DIRS}
  ${EIGEN3_INCLUDE_DIRS}
)

## Declare a C++ library
add_library(SparseOptFlowOcl src/SparseOptFlowOcl.cpp)
target_link_libraries(SparseOptFlowOcl ${OpenCV_LIBRARIES} ${OpenCL_LIBRARIES} ${catkin_LIBRARIES})

## Declare a C++ executable
add_executable(drone_detector_node src/drone_detector.cpp)


## Add cmake target dependencies of the executable
## same as for the library above
add_dependencies(drone_detector_node ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

add_custom_target(headers SOURCES ${HEADER_FILES} ${OPENCL_FILES} ${LAUNCHER_FILES} ${YAML_FILES})
configure_file(src/FastSpacedBMMethod.cl ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_BIN_DESTINATION}/FastSpacedBMMethod.cl COPYONLY)

## Specify libraries to link a library or executable target against
target_link_libraries(drone_detector_node
   ${catkin_LIBRARIES}
   ${OpenCV_LIBRARIES}
   ${EIGEN3_LIBRARIES}

   SparseOptFlowOcl
)
# if(CUDA_FOUND)
#   target_link_libraries(optic_flow_node
#   fsbmof
#   ${CUDA_LIBRARIES}
#   )
# endif()   

