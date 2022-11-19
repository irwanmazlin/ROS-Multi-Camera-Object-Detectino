cmake_minimum_required(VERSION 2.8.3)
project(detection)


find_package(InferenceEngine 2021.4)
if (NOT InferenceEngine_FOUND)
    message(FATAL_ERROR "")
endif()

set(OpenCV_DIR $ENV{OpenCV_DIR})
find_package(catkin REQUIRED COMPONENTS
  message_generation roscpp OpenCV InferenceEngine sensor_msgs roslaunch roslint topic_tools)


find_package(catkin REQUIRED COMPONENTS
  roscpp
)


# Flags
if(UNIX OR APPLE)
  # Linker flags.
  if( ${CMAKE_CXX_COMPILER_ID} STREQUAL "GNU" OR ${CMAKE_CXX_COMPILER_ID} STREQUAL "Intel")
    # GCC specific flags. ICC is compatible with them.
    set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -z noexecstack -z relro -z now")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -z noexecstack -z relro -z now")
  elseif(${CMAKE_CXX_COMPILER_ID} STREQUAL "Clang")
    # In Clang, -z flags are not compatible, they need to be passed to linker via -Wl.
    set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,-z,noexecstack -Wl,-z,relro -Wl,-z,now")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,-z,noexecstack -Wl,-z,relro -Wl,-z,now")
  endif()

  # Compiler flags.
  if( ${CMAKE_CXX_COMPILER_ID} STREQUAL "GNU")
    # GCC specific flags.
    if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER 4.9 OR CMAKE_CXX_COMPILER_VERSION VERSION_EQUAL 4.9)
      set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIE -fstack-protector-strong")
    else()
      set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIE -fstack-protector")
    endif()
  elseif(${CMAKE_CXX_COMPILER_ID} STREQUAL "Clang")
    # Clang is compatbile with some of the flags.
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIE -fstack-protector")
  elseif(${CMAKE_CXX_COMPILER_ID} STREQUAL "Intel" )
    # Same as above, with exception that ICC compilation crashes with -fPIE option, even
    # though it uses -pie linker option that require -fPIE during compilation. Checksec
    # shows that it generates correct PIE anyway if only -pie is provided.
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fstack-protector")
  endif()

  # Generic flags.
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC -fno-operator-names -Wformat -Wformat-security -Wall")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

 # Add OpenMP support
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fopenmp")
endif()

# Add x86 intrinsic compiler support
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -march=native")
execute_process(
    COMMAND bash -c "lscpu | grep -qi flags | grep -qi flags | grep -qi f16c"
    RESULT_VARIABLE SUPPORT_F16C)
if (SUPPORT_F16C EQUAL 0)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -mf16c")
  add_definitions(-DSUPPORT_MF16C)
endif()

execute_process(
    COMMAND bash -c "lscpu | grep -qi flags | grep -qi flags | grep -qi sse4_1"
    RESULT_VARIABLE SUPPORT_SSE41)
if (SUPPORT_SSE41 EQUAL 0)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -msse4.1")
endif()

## Generate messages in the 'msg' folder
add_message_files(FILES
  ObjectBoxList.msg
  ObjectBox.msg
  Objects.msg
  Object.msg
)

## Generate added messages and services with any dependencies listed here
generate_messages(DEPENDENCIES
  std_msgs
)


catkin_package(
  CATKIN_DEPENDS
  message_runtime
#  INCLUDE_DIRS include
#  LIBRARIES ros_openvino
#  CATKIN_DEPENDS other_catkin_pkg
#  DEPENDS system_lib
)

###########
## Build ##
###########

## Specify additional locations of header files
## Your package locations should be listed before other locations
include_directories(
  include ${catkin_INCLUDE_DIRS}
  ${InferenceEngine_INCLUDE_DIRS}
  ${InferenceEngine_INCLUDE_DIRS}/../samples
  ${InferenceEngine_DIR}/../src
  ${OpenCV_INCLUDE_DIRS}
# include
# ${catkin_INCLUDE_DIRS}
)

## Declare a C++ library







# add_executable(object_detection
#   src/object_detection.cpp
# )

add_executable(front
  src/front.cpp
)

# add_executable(frontleft
#   src/frontleft.cpp
# )

# add_dependencies(object_detection ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

add_dependencies(front ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

# add_dependencies(frontleft ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})



# target_link_libraries(object_detection
#   ${catkin_LIBRARIES}
#   ${InferenceEngine_LIBRARIES}
#   ${OpenCV_LIBRARIES}
# )
target_link_libraries(front
  ${catkin_LIBRARIES}
  ${InferenceEngine_LIBRARIES}
  ${OpenCV_LIBRARIES}
)

# target_link_libraries(frontleft
#   ${catkin_LIBRARIES}
#   ${InferenceEngine_LIBRARIES}
#   ${OpenCV_LIBRARIES}
# )

# install(TARGETS object_detection
#     ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#     LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#     RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
# )

install(TARGETS front
    ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
    LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
    RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)
# install(TARGETS frontleft
#     ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#     LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#     RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
# )


install(DIRECTORY launch/
   DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/launch )

install(DIRECTORY models/
   DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/models )
   
install(DIRECTORY msg/
   DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/msg )
   
install(DIRECTORY rviz/
   DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/rviz )
   
install(DIRECTORY src/
   DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/src )