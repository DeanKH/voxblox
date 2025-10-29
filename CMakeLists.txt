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

protobuf_generate(OUT_VAR voxblox_proto_files LANGUAGE cpp PROTOS
                  ${PROTO_DEFNS})

set(voxblox_proto_headers "")
set(voxblox_proto_cc_sources "")

foreach(proto_file ${voxblox_proto_files})
  get_filename_component(file_ext ${proto_file} EXT)
  if(file_ext STREQUAL ".pb.h")
    list(APPEND voxblox_proto_headers ${proto_file})
  elseif(file_ext STREQUAL ".pb.cc")
    list(APPEND voxblox_proto_cc_sources ${proto_file})
  endif()
endforeach()

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
    ${voxblox_proto_cc_sources})

add_library(${PROJECT_NAME} SHARED ${voxblox_SRCS})
target_include_directories(
  ${PROJECT_NAME}
  PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
         $<INSTALL_INTERFACE:include> ${Protobuf_INCLUDE_DIRS}
         ${EIGEN3_INCLUDE_DIRS} $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>)

add_library(voxblox::${PROJECT_NAME} ALIAS ${PROJECT_NAME})

# Link required libraries
target_link_libraries(${PROJECT_NAME} PUBLIC ${Protobuf_LIBRARIES}
                                             Eigen3::Eigen)

# Set target properties
set_target_properties(
  ${PROJECT_NAME}
  PROPERTIES VERSION 1.0.0
             SOVERSION 1
             EXPORT_NAME ${PROJECT_NAME})

# ##############################################################################
# INSTALLATION #
# ##############################################################################
include(CMakePackageConfigHelpers)

# # Install the library
install(
  TARGETS ${PROJECT_NAME}
  EXPORT voxbloxTargets
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

# # Install headers
install(
  DIRECTORY include/
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
  FILES_MATCHING
  PATTERN "*.h"
  PATTERN "*.hpp")

# # Install generated protobuf headers
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/proto/voxblox/Block.pb.h
              ${CMAKE_CURRENT_BINARY_DIR}/proto/voxblox/Layer.pb.h
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/proto/voxblox)

# Create package config file
configure_package_config_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/cmake/voxbloxConfig.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/voxbloxConfig.cmake
  INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/voxblox)

# # Install package config files
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/voxbloxConfig.cmake
              ${CMAKE_CURRENT_BINARY_DIR}/voxbloxConfigVersion.cmake
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/voxblox)

# Export targets
install(
  EXPORT voxbloxTargets
  FILE voxbloxTargets.cmake
  NAMESPACE voxblox::
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/voxblox)
