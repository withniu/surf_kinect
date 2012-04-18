cmake_minimum_required(VERSION 2.4.6)
include($ENV{ROS_ROOT}/core/rosbuild/rosbuild.cmake)

# Set the build type.  Options are:
#  Coverage       : w/ debug symbols, w/o optimization, w/ code-coverage
#  Debug          : w/ debug symbols, w/o optimization
#  Release        : w/o debug symbols, w/ optimization
#  RelWithDebInfo : w/ debug symbols, w/ optimization
#  MinSizeRel     : w/o debug symbols, w/ optimization, stripped binaries
#set(ROS_BUILD_TYPE RelWithDebInfo)

rosbuild_init()

#set the default path for built executables to the "bin" directory
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)
#set the default path for built libraries to the "lib" directory
set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)

#uncomment if you have defined messages
#rosbuild_genmsg()
#uncomment if you have defined services
#rosbuild_gensrv()

#common commands for building c++ executables and libraries
#rosbuild_add_library(${PROJECT_NAME} src/example.cpp)
#target_link_libraries(${PROJECT_NAME} another_library)
#rosbuild_add_boost_directories()
#rosbuild_link_boost(${PROJECT_NAME} thread)
#rosbuild_add_executable(example examples/example.cpp)
#target_link_libraries(example ${PROJECT_NAME})





PROJECT (SURFGPU)

INCLUDE(${CMAKE_SOURCE_DIR}/src/CMake/cuda/FindCUDA.cmake)
include(${CMAKE_SOURCE_DIR}/src/CMake/FindCUDPP.cmake)
find_package(OpenCV REQUIRED)

#set(CUDA_BUILD_CUBIN ON)
set(CUDA_BUILD_EMULATION OFF)
set(CUDA_NVCC_FLAGS "-arch=sm_11")

option(CUDA_24BIT_IMUL "Use 24bit integer multiplication for IMUL macros" ON)
if(CUDA_24BIT_IMUL)
	add_definitions(-DCUDA_24BIT_IMUL)
endif()

option(SURF_MATCH_SIMPLE "Use simple SURF descriptor matching" OFF)
if(SURF_MATCH_SIMPLE)
	add_definitions(-DSURF_MATCH_SIMPLE)
endif()

# Needed for M_PI and friends on Windows
if(WIN32)
	add_definitions(-D_USE_MATH_DEFINES)
endif()

INCLUDE_DIRECTORIES(
${CMAKE_SOURCE_DIR}
${CUDA_TOOLKIT_INCLUDE}
${OpenCV_INCLUDE_DIR}
${CUDA_INCLUDE_DIRS}
${CUDPP_INCLUDE_DIR}
)

CUDA_INCLUDE_DIRECTORIES(
${CMAKE_SOURCE_DIR}
${CUDA_TOOLKIT_INCLUDE}
${OpenCV_INCLUDE_DIR}
${CUDA_INCLUDE_DIRS}
)

LINK_DIRECTORIES(
	${CUDA_SDK_LIB_DIR}
)

set_source_files_properties(src/cuda/buildSURFDescriptorsGPU.cu PROPERTIES
	COMPILE_FLAGS "--maxrregcount 18")
CUDA_ADD_EXECUTABLE(surf
	src/fasthessian.cpp
	src/integral.cpp
	src/ipoint.cpp
#	main.cpp
	src/main_kinect.cpp
	src/surf.cpp
	src/utils.cpp
	src/cuda/buildSURFDescriptorsGPU.cu
	src/cuda/convertRGB2GrayFloatGPU.cu
	src/cuda/convertRGBA2GrayFloatGPU.cu
	src/cuda/detectIpointOrientationsGPU.cu
	src/cuda/getMatchesGPU.cpp
	src/cuda/helper_funcs.cpp
	src/cuda/buildDetGPU.cu
	src/cuda/nonMaxGPU.cu
	src/cuda/matchSURFKeypointsGPU.cu
	src/cuda/normalizeSURFDescriptorsGPU.cu
	src/cuda/strerror_cudpp.cpp
	src/cuda/transposeGPU.cu
)

TARGET_LINK_LIBRARIES(surf
${CUDA_LIBRARIES}
${OpenCV_LIBS}
${CUDA_CUDA_LIBRARY}
${CUDA_CUDART_LIBRARY}
${CUDPP_LIBRARY}
)

