cmake_minimum_required(VERSION 3.5)

if(NOT DEFINED ENV{PAF_SAMPLE_NAME})
  message(FATAL_ERROR "Please define PAF_SAMPLE_NAME")
endif()

if(NOT DEFINED CMAKE_TOOLCHAIN_FILE)
  if(DEFINED ENV{VITASDK})
    set(CMAKE_TOOLCHAIN_FILE "$ENV{VITASDK}/share/vita.toolchain.cmake" CACHE PATH "toolchain file")
  else()
    message(FATAL_ERROR "Please define VITASDK to point to your SDK path!")
  endif()
endif()

project(paf_sample)
include("${VITASDK}/share/vita.cmake" REQUIRED)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wl,-q -Wall -O2 -fno-inline -fno-builtin -fshort-wchar")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-rtti -fno-exceptions -Wl,-q -Wall -O2 -fno-builtin -fshort-wchar -Wno-unused-function -Wno-sign-compare")

set(VITA_MAKE_FSELF_FLAGS "${VITA_MAKE_FSELF_FLAGS} -a 0x2F00000000000101")

enable_language(CXX ASM)

link_directories(
  ${CMAKE_CURRENT_SOURCE_DIR}/../vitasdk
)

include_directories(
)

add_executable(${PROJECT_NAME}
  ${CMAKE_CURRENT_SOURCE_DIR}/main.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/../samples/$ENV{PAF_SAMPLE_NAME}/src/paf_sample.cpp
)

target_link_libraries(${PROJECT_NAME}
  SceAppMgr_stub
  SceKernelThreadMgr_stub
  SceSysmodule_stub
  SceCtrl_stub
  SceLibKernel_stub
  SceIofilemgr_stub
  SceProcessmgr_stub
  SceNet_stub
  SceNetCtl_stub
  SceSsl_stub
  SceHttp_stub
  ScePaf_stub_weak
  ScePafWebMapView_stub_weak
  SceWebUIPlugin_stub_weak
)

set_target_properties(${PROJECT_NAME}
  PROPERTIES LINK_FLAGS "-nostartfiles -nostdlib"
  COMPILE_FLAGS "'-D__declspec(__v__)=/**/'"
)

vita_create_self(${PROJECT_NAME}.self ${PROJECT_NAME}
  CONFIG exports.yml
  UNSAFE
  STRIPPED
  REL_OPTIMIZE
)

vita_create_vpk(paf_sample.vpk "PAFS00001" ${PROJECT_NAME}.self
  VERSION "01.00"
  NAME "paf sample"
  FILE ${CMAKE_CURRENT_SOURCE_DIR}/../samples/$ENV{PAF_SAMPLE_NAME}/sample_plugin.rco sample_plugin.rco
)

vita_create_stubs(stubs_${PROJECT_NAME} ${PROJECT_NAME} exports.yml)
