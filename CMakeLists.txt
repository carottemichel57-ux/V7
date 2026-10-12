cmake_minimum_required(VERSION 3.5)

if(NOT DEFINED CMAKE_TOOLCHAIN_FILE)
  if(DEFINED ENV{VITASDK})
    set(CMAKE_TOOLCHAIN_FILE "$ENV{VITASDK}/share/vita.toolchain.cmake" CACHE PATH "toolchain file")
  else()
    message(FATAL_ERROR "VITASDK n'est pas défini")
  endif()
endif()

project(vitacraft C)
include("$ENV{VITASDK}/share/vita.cmake" REQUIRED)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wl,-q -O2 -std=gnu99")

add_executable(vitacraft main.c)

# Les fonctions sceIo* de l'API utilisateur sont exportées par Iofilemgr.
# Certains VitaSDK historiques n'installent pas ce stub : ne l'ajouter que s'il existe.
set(EXTRA_STUBS "")
if(EXISTS "$ENV{VITASDK}/arm-vita-eabi/lib/libSceIofilemgr_stub.a")
  list(APPEND EXTRA_STUBS SceIofilemgr_stub)
endif()

target_link_libraries(vitacraft
  taihen_stub
  SceLibKernel_stub
  SceProcessmgr_stub
  SceDisplay_stub
  SceCtrl_stub
  ${EXTRA_STUBS}
)

set_target_properties(vitacraft PROPERTIES LINK_FLAGS "-nostartfiles")

vita_create_self(vitacraft.suprx vitacraft CONFIG exports.yml UNSAFE)
