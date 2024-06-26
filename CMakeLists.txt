# CMake 3.15+ for Thrust/Cub support
cmake_minimum_required(VERSION 3.19 FATAL_ERROR)

set(CMAKE_TOOLCHAIN_FILE ${CMAKE_CURRENT_SOURCE_DIR}/vcpkg/scripts/buildsystems/vcpkg.cmake CACHE STRING "Vcpkg toolchain file")


project(MegBA VERSION 1.0.0 LANGUAGES CUDA CXX)



set(CMAKE_CXX_STANDARD 14)
set(CUDA_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED TRUE)
set(CMAKE_CXX_VISIBILITY_PRESET hidden)
set(CMAKE_VISIBILITY_INLINES_HIDDEN TRUE)
set(CMAKE_POSITION_INDEPENDENT_CODE TRUE)
set(CMAKE_CUDA_ARCHITECTURES "60;61;62;70;72;75;80;86;87")
set(CMAKE_CUDA_SEPARABLE_COMPILATION TRUE)
set(CMAKE_CUDA_RESOLVE_DEVICE_SYMBOLS TRUE)
set(CMAKE_CUDA_HOST_COMPILATION_CPP FALSE)
set(CMAKE_CUDA_PROPAGATE_HOST_FLAGS FALSE)


include(GenerateExportHeader)
include(GNUInstallDirs)

set(generated_headers "${CMAKE_CURRENT_BINARY_DIR}/generated_headers")
set(config_install_dir "${CMAKE_INSTALL_LIBDIR}/cmake/MegBA")
set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

# Configuration
set(version_config "${generated_dir}/MegBAConfigVersion.cmake")
set(project_config "${generated_dir}/MegBAConfig.cmake")
set(ALGO_EXPORT_NAME "Algo_Targets")
set(EDGE_EXPORT_NAME "Edge_Targets")
set(GEO_EXPORT_NAME "Geo_Targets")
set(LINEAR_SYSTEM_EXPORT_NAME "LinearSystem_Targets")
set(OPERATOR_EXPORT_NAME "Operator_Targets")
set(PROBLEM_EXPORT_NAME "Problem_Targets")
set(RESOURCE_EXPORT_NAME "Resource_Targets")
set(SOLVER_EXPORT_NAME "Solver_Targets")
set(VERTEX_EXPORT_NAME "Vertex_Targets")

option(MEGBA_ENABLE_NCCL "Enable NCCL" FALSE)
option(MEGBA_ENABLE_EXAMPLE "BUILD EXAMPLES" TRUE)

# CUDA
find_package(CUDAToolkit REQUIRED)
find_package(OpenMP REQUIRED)
find_package(Eigen3 CONFIG REQUIRED)
find_package(Threads REQUIRED)
if (${CMAKE_CUDA_COMPILER_VERSION} GREATER_EQUAL 11.5)
    set(Thrust_DIR /usr/local/cuda/targets/x86_64-linux/lib/cmake/thrust/)
    set(CUB_DIR /usr/local/cuda/targets/x86_64-linux/lib/cmake/cub/)
else ()
    set(Thrust_DIR /usr/local/cuda/include/thrust/cmake/)
    set(CUB_DIR /usr/local/cuda/include/cub/cmake/)
endif()

if (${CMAKE_BUILD_TYPE} MATCHES "Debug")
    add_definitions(-DDEBUG=1)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O0 -fPIC")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O0 -fPIC -fopenmp -pthread")
    set(CUDA_NVCC_FLAGS "${CUDA_NVCC_FLAGS} -O0 --extended-lambda")
else ()
    add_definitions(-DDEBUG=0)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O3 -fPIC")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3 -fPIC -fopenmp -pthread")
    set(CUDA_NVCC_FLAGS "${CUDA_NVCC_FLAGS} -O3 --extended-lambda")
endif ()

find_package(Thrust REQUIRED CONFIG)
thrust_create_target(Thrust)

add_subdirectory(src)

if(MEGBA_ENABLE_EXAMPLE)
add_subdirectory(examples)
endif()

include(CMakePackageConfigHelpers)

write_basic_package_version_file(
    "${version_config}" COMPATIBILITY SameMajorVersion
)

configure_package_config_file(
    "cmake/libraryConfig.cmake.in"
    "${project_config}"
    INSTALL_DESTINATION "${config_install_dir}"
)

install(
    FILES "${project_config}" "${version_config}"
    DESTINATION "${config_install_dir}"
)
install(DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/include/ 
        DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}")

set(CPACK_GENERATOR "7Z")
set(CPACK_PACKAGE_VERSION "${MegBA_VERSION}")
include(CPack)

include(FeatureSummary)
feature_summary(WHAT ALL)
