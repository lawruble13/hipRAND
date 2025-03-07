# Fortran Wrapper

if(HIP_COMPILER STREQUAL "nvcc")
    configure_file(
        "${CMAKE_CURRENT_SOURCE_DIR}/hip/hip_nvcc_m.f90"
        "${PROJECT_BINARY_DIR}/src/fortran/hip_m.f90"
        COPYONLY
    )
else()
    configure_file(
        "${CMAKE_CURRENT_SOURCE_DIR}/hip/hip_hcc_m.f90"
        "${PROJECT_BINARY_DIR}/src/fortran/hip_m.f90"
        COPYONLY
    )
endif()

set(CMAKE_Fortran_MODULE_DIRECTORY ${PROJECT_BINARY_DIR}/library/fortran)

# hipRAND Fortran wrapper
set(FORTRAN_HIPRAND_SRCS
    ${CMAKE_CURRENT_SOURCE_DIR}/hiprand_m.f90
    ${PROJECT_BINARY_DIR}/src/fortran/hip_m.f90
)
add_library(hiprand_fortran STATIC "${FORTRAN_HIPRAND_SRCS}")
set_target_properties(hiprand_fortran PROPERTIES LINKER_LANGUAGE Fortran)
if(HIP_COMPILER STREQUAL "nvcc")
    target_link_libraries(hiprand_fortran
        PRIVATE
            hiprand
            ${CUDA_LIBRARIES}
            ${CUDA_curand_LIBRARY}
    )
else()
    target_link_libraries(hiprand_fortran
        PRIVATE
            hiprand
            roc::rocrand
            "-L${HIP_ROOT_DIR}/lib -lhip_hcc -Wl,-rpath,${HIP_ROOT_DIR}/lib"
    )
endif()

# Install Fortran sources
rocm_install(
    FILES ${FORTRAN_HIPRAND_SRCS}
    DESTINATION hiprand/src/fortran
    PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ
)
