cmake_minimum_required(VERSION 2.8)
project(MODEL_ACQUISITION)

IF (NOT CMAKE_BUILD_TYPE)
    SET(CMAKE_BUILD_TYPE Release)
ENDIF ()

MESSAGE("Build type: " ${CMAKE_BUILD_TYPE})

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}  -Wall  -O3 -march=native ")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -O3 -march=native")

# Check C++11 or C++0x support
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
if (COMPILER_SUPPORTS_CXX11)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
    add_definitions(-DCOMPILEDWITHC11)
    message(STATUS "Using flag -std=c++11.")
elseif (COMPILER_SUPPORTS_CXX0X)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
    add_definitions(-DCOMPILEDWITHC0X)
    message(STATUS "Using flag -std=c++0x.")
else ()
    message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif ()

LIST(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake_modules)

find_package(CUDA REQUIRED)
SET(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS};-O3 -use_fast_math)
include_directories(${CUDA_INCLUDE_DIRS})
find_package(PCL 1.8 REQUIRED)
include_directories(${PCL_INCLUDE_DIRS})
link_directories(${PCL_LIBRARY_DIRS})
add_definitions(${PCL_DEFINITIONS})
find_package(OpenCV 3.0 QUIET)
if (NOT OpenCV_FOUND)
    find_package(OpenCV 2.4.3 QUIET)
    if (NOT OpenCV_FOUND)
        message(FATAL_ERROR "OpenCV > 2.4.3 not found.")
    endif ()
endif ()

find_package(Eigen3 3.1.0 REQUIRED)
find_package(Pangolin REQUIRED)
find_package(libfreenect REQUIRED)


include_directories(
        ${PROJECT_SOURCE_DIR}
        ${PROJECT_SOURCE_DIR}/include
        ${EIGEN3_INCLUDE_DIR}
        ${Pangolin_INCLUDE_DIRS}
        ${PROJECT_SOURCE_DIR}/Thirdparty/TSDF/include
)

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/lib)

add_library(${PROJECT_NAME} SHARED
        src/ORBSLAMSystem.cpp
        src/Tracking.cc
        src/LocalMapping.cc
        src/LoopClosing.cc
        src/ORBextractor.cc
        src/ORBmatcher.cc
        src/FrameDrawer.cc
        src/Converter.cc
        src/MapPoint.cc
        src/KeyFrame.cc
        src/Map.cc
        src/MapDrawer.cc
        src/Optimizer.cc
        src/PnPsolver.cc
        src/Frame.cc
        src/KeyFrameDatabase.cc
        src/Sim3Solver.cc
        src/Initializer.cc
        src/Viewer.cc
        src/Octree.cpp
        src/PointCloudGenerator.cpp
        )

target_link_libraries(${PROJECT_NAME}
        ${OpenCV_LIBS}
        ${EIGEN3_LIBS}
        ${Pangolin_LIBRARIES}
        ${PROJECT_SOURCE_DIR}/Thirdparty/DBoW2/lib/libDBoW2.so
        ${PROJECT_SOURCE_DIR}/Thirdparty/g2o/lib/libg2o.so
        ${PROJECT_SOURCE_DIR}/Thirdparty/TSDF/lib/libtsdf_cuda.so
        )

# Build examples
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/Examples/RGB-D)

add_executable(rgbd_kinect
        Examples/RGB-D/rgbd_kinect.cpp)

target_link_libraries(rgbd_kinect
        ${PROJECT_NAME}
        ${PCL_LIBRARIES}
        ${LIBUSB1_LIBRARIES}
        freenect
        freenect_sync
        ${PROJECT_SOURCE_DIR}/Thirdparty/TSDF/lib/libtsdf_cuda.so
        ${CUDA_LIBRARIES})

add_executable(rgbd_kinect_gl
        Examples/RGB-D/rgbd_kinect_gl.cpp)

target_link_libraries(rgbd_kinect_gl
        ${PROJECT_NAME}
        ${PCL_LIBRARIES}
        ${LIBUSB1_LIBRARIES}
        freenect
        freenect_sync
        ${PROJECT_SOURCE_DIR}/Thirdparty/TSDF/lib/libtsdf_cuda.so
        ${CUDA_LIBRARIES} glut GL GLU GLEW)

add_executable(rgbd_realsense
        Examples/RGB-D/rgbd_realsense.cpp
        src/BridgeRSR200.cpp)

target_link_libraries(rgbd_realsense
        ${PROJECT_NAME}
        ${PCL_LIBRARIES}
        ${LIBUSB1_LIBRARIES}
        realsense
        ${PROJECT_SOURCE_DIR}/Thirdparty/TSDF/lib/libtsdf_cuda.so
        ${CUDA_LIBRARIES})

add_executable(rgbd_realsense_gl
        Examples/RGB-D/rgbd_realsense_gl.cpp
        src/BridgeRSR200.cpp)

target_link_libraries(rgbd_realsense_gl
        ${PROJECT_NAME}
        ${PCL_LIBRARIES}
        ${LIBUSB1_LIBRARIES}
        realsense
        ${PROJECT_SOURCE_DIR}/Thirdparty/TSDF/lib/libtsdf_cuda.so
        ${CUDA_LIBRARIES} glut GL GLU GLEW)

