add_library(base_linear_system SHARED base_linear_system.cpp)
cuda_add_library(base_linear_system_CUDA SHARED base_linear_system.cu)
target_link_libraries(base_linear_system base_linear_system_CUDA)

add_library(schur_linear_system SHARED schur_linear_system.cpp)
cuda_add_library(schur_linear_system_CUDA SHARED schur_linear_system.cu)
target_link_libraries(schur_linear_system schur_linear_system_CUDA base_linear_system)

add_library(LM_linear_system SHARED LM_linear_system.cpp)
cuda_add_library(LM_linear_system_CUDA SHARED LM_linear_system.cu)
target_link_libraries(LM_linear_system LM_linear_system_CUDA base_linear_system)

add_library(schur_LM_linear_system SHARED schur_LM_linear_system.cpp)
cuda_add_library(schur_LM_linear_system_CUDA SHARED schur_LM_linear_system.cu)
target_link_libraries(schur_LM_linear_system schur_LM_linear_system_CUDA)
target_link_libraries(schur_LM_linear_system LM_linear_system schur_linear_system nccl)