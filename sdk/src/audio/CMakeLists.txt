# rexaudio - Audio subsystem library
# Audio processing and XMA decoding for Xbox 360 emulation

add_library(rexaudio OBJECT
    
    audio_driver.cpp
    audio_system.cpp
    downmix.cpp
    xma_context.cpp
    xma_decoder.cpp
    xma_register_file.cpp
    # NOP backend (always available)
    nop/nop_audio_system.cpp
)
add_library(rex::audio ALIAS rexaudio)

# XMA sample conversion uses _mm_shuffle_epi8 on x86 hosts (SSSE3).
if((CMAKE_SYSTEM_PROCESSOR MATCHES "x86_64|AMD64|amd64|i.86") AND NOT MSVC)
    target_compile_options(rexaudio PRIVATE -mssse3)
endif()
if(REXGLUE_PLATFORM_SWITCH)
    # libnx audout backend
    target_sources(rexaudio PRIVATE switch/switch_audio_system.cpp)
else()
    # SDL backend
    target_sources(rexaudio PRIVATE sdl/sdl_audio_system.cpp sdl/sdl_audio_driver.cpp)
endif()

rex_target_pch(rexaudio
    <mutex>
    <rex/math.h>
    <rex/string.h>
    <rex/memory.h>
    <rex/filesystem.h>
    <rex/thread.h>
    <rex/thread/mutex.h>
    <rex/ppc/context.h>
    <rex/system/kernel_state.h>
    <rex/system/xthread.h>
)

target_include_directories(rexaudio PUBLIC
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

target_link_libraries(rexaudio
    PUBLIC rexcore
    PRIVATE libavcodec libavutil
)
if(NOT REXGLUE_PLATFORM_SWITCH)
    target_link_libraries(rexaudio PUBLIC SDL3::SDL3)
endif()
