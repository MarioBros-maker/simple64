cmake_minimum_required(VERSION 3.20)

project(mupen64plus LANGUAGES C)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

include(CheckIPOSupported)
check_ipo_supported(RESULT ENABLE_IPO)
if(ENABLE_IPO)
    message("Interprocedural optimizations enabled")
endif(ENABLE_IPO)

if(ENABLE_IPO AND (CMAKE_BUILD_TYPE STREQUAL "Release"))
    set(CMAKE_INTERPROCEDURAL_OPTIMIZATION ON)
endif()

find_package(PNG REQUIRED)
find_package(ZLIB REQUIRED)
find_package(SDL2 REQUIRED)
find_package(PkgConfig REQUIRED)
pkg_check_modules(SDL_NET REQUIRED SDL2_net)

include_directories(
    src
    subprojects/md5
    subprojects/xxhash
    subprojects/minizip
    subprojects/xdelta
    ${SDL2_INCLUDE_DIRS}
)

add_definitions(-DM64P_PARALLEL -DM64P_NETPLAY -DNO_ASM)

add_library(mupen64plus SHARED
    src/api/callbacks.c
    src/api/common.c
    src/api/config.c
    src/api/debugger.c
    src/api/frontend.c
    src/api/vidext.c
    src/backends/api/video_capture_backend.c
    src/backends/plugins_compat/audio_plugin_compat.c
    src/backends/plugins_compat/input_plugin_compat.c
    src/backends/clock_ctime_plus_delta.c
    src/backends/dummy_video_capture.c
    src/backends/file_storage.c
    src/device/cart/cart.c
    src/device/cart/af_rtc.c
    src/device/cart/cart_rom.c
    src/device/cart/eeprom.c
    src/device/cart/flashram.c
    src/device/cart/is_viewer.c
    src/device/cart/sram.c
    src/device/controllers/game_controller.c
    src/device/controllers/vru_controller.c
    src/device/controllers/paks/biopak.c
    src/device/controllers/paks/mempak.c
    src/device/controllers/paks/rumblepak.c
    src/device/controllers/paks/transferpak.c
    src/device/dd/dd_controller.c
    src/device/dd/disk.c
    src/device/device.c
    src/device/gb/gb_cart.c
    src/device/gb/mbc3_rtc.c
    src/device/gb/m64282fp.c
    src/device/memory/memory.c
    src/device/pif/bootrom_hle.c
    src/device/pif/cic.c
    src/device/pif/n64_cic_nus_6105.c
    src/device/pif/pif.c
    src/device/r4300/cached_interp.c
    src/device/r4300/cp0.c
    src/device/r4300/cp1.c
    src/device/r4300/idec.c
    src/device/r4300/interrupt.c
    src/device/r4300/pure_interp.c
    src/device/r4300/r4300_core.c
    src/device/r4300/dcache.c
    src/device/r4300/icache.c
    src/device/r4300/tlb.c
    src/device/rcp/ai/ai_controller.c
    src/device/rcp/mi/mi_controller.c
    src/device/rcp/pi/pi_controller.c
    src/device/rcp/rdp/fb.c
    src/device/rcp/rdp/rdp_core.c
    src/device/rcp/ri/ri_controller.c
    src/device/rcp/rsp/rsp_core.c
    src/device/rcp/si/si_controller.c
    src/device/rcp/vi/vi_controller.c
    src/device/rdram/rdram.c
    src/main/main.c
    src/main/util.c
    src/main/cheat.c
    src/main/eventloop.c
    src/main/rom.c
    src/main/savestates.c
    src/main/screenshot.c
    src/main/sdl_key_converter.c
    src/main/workqueue.c
    src/plugin/plugin.c
    src/plugin/dummy_video.c
    src/plugin/dummy_audio.c
    src/plugin/dummy_input.c
    src/plugin/dummy_rsp.c
    src/main/netplay.c
    subprojects/md5/md5.c
    subprojects/minizip/ioapi.c
    subprojects/minizip/unzip.c
    subprojects/minizip/zip.c
)

if (WIN32)
    target_sources(mupen64plus PRIVATE
        src/osal/dynamiclib_win32.c
        src/osal/files_win32.c
    )
endif (WIN32)

if (UNIX)
    target_sources(mupen64plus PRIVATE
        src/osal/dynamiclib_unix.c
        src/osal/files_unix.c
    )
endif (UNIX)

target_compile_options(mupen64plus PRIVATE -march=x86-64-v3)
target_link_options(mupen64plus PRIVATE -march=x86-64-v3)
target_link_libraries(mupen64plus PRIVATE SDL2 ${SDL_NET_LIBRARIES} ZLIB::ZLIB PNG::PNG)