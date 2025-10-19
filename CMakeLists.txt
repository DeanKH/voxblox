cmake_minimum_required(VERSION 3.22)
project(voxblox)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# ##############################################################################
# PROTOBUF #
# ##############################################################################
# General idea: first check if we have protobuf catkin, then use that. Otherwise
# use system protobuf.
find_package(Eigen3 REQUIRED)
find_package(Protobuf REQUIRED)
if(NOT Protobuf_FOUND)
  message(FATAL "Protobuf not found")
  # find_package(Protobuf REQUIRED)
endif()
set(PROTO_DEFNS proto/voxblox/Block.proto proto/voxblox/Layer.proto)
set(BASE_PATH "proto")

protobuf_generate(OUT_VAR voxblox_proto_sources LANGUAGE cpp PROTOS
                  ${PROTO_DEFNS})

set("voxblox_SRCS"
    src/alignment/icp.cc
    src/core/block.cc
    src/core/esdf_map.cc
    src/core/tsdf_map.cc
    src/integrator/esdf_integrator.cc
    src/integrator/esdf_occ_integrator.cc
    src/integrator/integrator_utils.cc
    src/integrator/intensity_integrator.cc
    src/integrator/tsdf_integrator.cc
    src/io/mesh_ply.cc
    src/io/sdf_ply.cc
    src/mesh/marching_cubes.cc
    src/simulation/objects.cc
    src/simulation/simulation_world.cc
    src/utils/camera_model.cc
    src/utils/evaluation_utils.cc
    src/utils/layer_utils.cc
    src/utils/neighbor_tools.cc
    src/utils/protobuf_utils.cc
    src/utils/timing.cc
    src/utils/voxel_utils.cc
    ${voxblox_proto_sources})

add_library(${PROJECT_NAME} SHARED ${voxblox_SRCS})
target_include_directories(
  ${PROJECT_NAME}
  PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
         $<INSTALL_INTERFACE:include> ${Protobuf_INCLUDE_DIRS}
         ${EIGEN3_INCLUDE_DIRS} ${CMAKE_CURRENT_BINARY_DIR}/proto)

# target_link_libraries(${PROJECT_NAME} ${PROJECT_NAME}_proto)
