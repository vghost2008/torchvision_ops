cmake_minimum_required(VERSION 3.12)
set(CMAKE_PREFIX_PATH /home/wj/ai/software/libtorch)
project(visionop)
set(CMAKE_CXX_STANDARD 14)
file(STRINGS version.txt TORCHVISION_VERSION)

option(WITH_CUDA "Enable CUDA support" OFF)

if(WITH_CUDA)
  enable_language(CUDA)
  add_definitions(-D__CUDA_NO_HALF_OPERATORS__)
  add_definitions(-DWITH_CUDA)
  set(CMAKE_CUDA_FLAGS "${CMAKE_CUDA_FLAGS} --expt-relaxed-constexpr")
endif()


find_package(Torch REQUIRED)
#find_package(PNG REQUIRED)
#find_package(JPEG REQUIRED)

function(CUDA_CONVERT_FLAGS EXISTING_TARGET)
    get_property(old_flags TARGET ${EXISTING_TARGET} PROPERTY INTERFACE_COMPILE_OPTIONS)
    if(NOT "${old_flags}" STREQUAL "")
        string(REPLACE ";" "," CUDA_flags "${old_flags}")
        set_property(TARGET ${EXISTING_TARGET} PROPERTY INTERFACE_COMPILE_OPTIONS
            "$<$<BUILD_INTERFACE:$<COMPILE_LANGUAGE:CXX>>:${old_flags}>$<$<BUILD_INTERFACE:$<COMPILE_LANGUAGE:CUDA>>:-Xcompiler=${CUDA_flags}>"
            )
    endif()
endfunction()

if(MSVC)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /wd4819")
  if(WITH_CUDA)
    set(CMAKE_CUDA_FLAGS "${CMAKE_CUDA_FLAGS} -Xcompiler=/wd4819")
    foreach(diag cc_clobber_ignored integer_sign_change useless_using_declaration
      set_but_not_used field_without_dll_interface
      base_class_has_different_dll_interface
      dll_interface_conflict_none_assumed
      dll_interface_conflict_dllexport_assumed
      implicit_return_from_non_void_function
      unsigned_compare_with_zero
      declared_but_not_referenced
      bad_friend_decl)
      string(APPEND CMAKE_CUDA_FLAGS " -Xcudafe --diag_suppress=${diag}")
    endforeach()
    CUDA_CONVERT_FLAGS(torch_cpu)
    if(TARGET torch_cuda)
      CUDA_CONVERT_FLAGS(torch_cuda)
    endif()
    if(TARGET torch_cuda_cu)
      CUDA_CONVERT_FLAGS(torch_cuda_cu)
    endif()
    if(TARGET torch_cuda_cpp)
      CUDA_CONVERT_FLAGS(torch_cuda_cpp)
    endif()
  endif()
endif()

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

list(APPEND ALLOW_LISTED ops .)
if(WITH_CUDA)
    list(APPEND ALLOW_LISTED ./ops/cuda ./ops/autocast)
endif()

FOREACH(DIR ${ALLOW_LISTED})
    file(GLOB ALL_SOURCES ${ALL_SOURCES} ${DIR}/*.*)
ENDFOREACH()

add_library(${PROJECT_NAME} SHARED ${ALL_SOURCES})
#add_library(${PROJECT_NAME} STATIC ${ALL_SOURCES})
target_link_libraries(${PROJECT_NAME} PRIVATE ${TORCH_LIBRARIES})
set_target_properties(${PROJECT_NAME} PROPERTIES
  EXPORT_NAME TorchVision
  INSTALL_RPATH ${TORCH_INSTALL_PREFIX}/lib)
include_directories(ops)

