cmake_minimum_required(VERSION 3.13)

project(pihpsdr VERSION 1.0.1)

if (MSVC)
    set(CMAKE_CXX_FLAGS "-O2 /std:c++17 /EHsc /W3")

    get_filename_component(SDKPATH ${CMAKE_LINKER} DIRECTORY)
    find_program(LIBEXE lib HINTS ${SDKPATH} REQUIRED)

    # External Project FFTW on Windows
    if(${CMAKE_EXE_LINKER_FLAGS} MATCHES "X86")
        SET(FFTW_URL "ftp://ftp.fftw.org/pub/fftw/fftw-3.3.5-dll32.zip")
        SET(ARCH x86)
        SET(HASH 29882a43033c9393479a4df52a2e9120589c06a2b724155b1a682747fa3e57d4)
    else()
        SET(FFTW_URL "ftp://ftp.fftw.org/pub/fftw/fftw-3.3.5-dll64.zip")
        SET(ARCH x64)
        SET(HASH cfd88dc0e8d7001115ea79e069a2c695d52c8947f5b4f3b7ac54a192756f439f)
    endif()

    include(ExternalProject)
    ExternalProject_Add(
        LIBFFTW
        URL ${FFTW_URL}
        URL_HASH SHA256=${HASH}
        BUILD_IN_SOURCE TRUE
        CONFIGURE_COMMAND   ""
        BUILD_COMMAND       ${LIBEXE} /def:./libfftw3f-3.def /MACHINE:${ARCH} /OUT:./fftw3f-3.lib
        INSTALL_COMMAND     ""
    )
    ExternalProject_Get_Property(LIBFFTW SOURCE_DIR)
    SET(LIBFFTW_INCLUDE_DIRS ${SOURCE_DIR})
    SET(LIBFFTW_LIBRARY_DIRS ${SOURCE_DIR})
    SET(LIBFFTW_LIBRARIES fftw3f-3)

    add_subdirectory(ExtIO_sddc)
else()
    if (USE_DEBUG_ASAN)
        set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -fsanitize=address")
        set(CMAKE_CXX_FLAGS "-O0 -std=c++17 -Wall -Werror")
    else()
        set(CMAKE_CXX_FLAGS "-O3 -std=c++17 -Wall -Werror")
    endif(USE_DEBUG_ASAN)
    #add_compile_options(-Wall -Wextra -pedantic)
    include(FindPkgConfig)
    pkg_check_modules(LIBUSB REQUIRED libusb-1.0)
    pkg_check_modules(LIBFFTW REQUIRED fftw3)
    pkg_check_modules(LIBGTK  REQUIRED gtk+-3.0)
    pkg_check_modules(LIBALSA REQUIRED alsa)
endif (MSVC)

execute_process(COMMAND bash -c "git --no-pager show --date=short --format=\"%ai\" --name-only | head -n 1"
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
    OUTPUT_STRIP_TRAILING_WHITESPACE
    OUTPUT_VARIABLE GIT_DATE
)

execute_process(COMMAND git describe --abbrev=0 --tags
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
    OUTPUT_STRIP_TRAILING_WHITESPACE
    OUTPUT_VARIABLE GIT_VERSION
)

SET(WDSP_SRCS 
    wdsp/amd.c
    wdsp/ammod.c
    wdsp/amsq.c
    wdsp/analyzer.c
    wdsp/anf.c
    wdsp/anr.c
    wdsp/bandpass.c
    wdsp/calcc.c
    wdsp/calculus.c
    wdsp/cblock.c
    wdsp/cfcomp.c
    wdsp/cfir.c
    wdsp/channel.c
    wdsp/comm.c
    wdsp/compress.c
    wdsp/delay.c
    wdsp/dexp.c
    wdsp/div.c
    wdsp/eer.c
    wdsp/emnr.c
    wdsp/emph.c
    wdsp/eq.c
    wdsp/fcurve.c
    wdsp/fir.c
    wdsp/firmin.c
    wdsp/fmd.c
    wdsp/fmmod.c
    wdsp/fmsq.c
    wdsp/gain.c
    wdsp/gen.c
    wdsp/icfir.c
    wdsp/iir.c
    wdsp/iobuffs.c
    wdsp/iqc.c
    wdsp/linux_port.c
    wdsp/lmath.c
    wdsp/main.c
    wdsp/meter.c
    wdsp/meterlog10.c
    wdsp/nbp.c
    wdsp/nob.c
    wdsp/nobII.c
    wdsp/osctrl.c
    wdsp/patchpanel.c
    wdsp/resample.c
    wdsp/rmatch.c
    wdsp/RXA.c
    wdsp/sender.c
    wdsp/shift.c
    wdsp/siphon.c
    wdsp/slew.c
    wdsp/snb.c
    wdsp/syncbuffs.c
    wdsp/TXA.c
    wdsp/utilities.c
    wdsp/varsamp.c
    wdsp/version.c
    wdsp/wcpAGC.c
    wdsp/wisdom.c
)

add_executable(pihpsdr
    ${WDSP_SRCS}
    ${SOAPY_SOURCES}
    audio.c
    band.c
    discovered.c
    discovery.c
    filter.c
    main.c
    new_menu.c
    about_menu.c
    exit_menu.c
    radio_menu.c
    rx_menu.c
    ant_menu.c
    display_menu.c
    dsp_menu.c
    pa_menu.c
    cw_menu.c
    oc_menu.c
    xvtr_menu.c
    equalizer_menu.c
    step_menu.c
    meter_menu.c
    band_menu.c
    bandstack_menu.c
    mode_menu.c
    filter_menu.c
    noise_menu.c
    agc_menu.c
    vox_menu.c
    fft_menu.c
    diversity_menu.c
    tx_menu.c
    vfo_menu.c
    test_menu.c
    meter.c
    mode.c
    old_discovery.c
    new_discovery.c
    old_protocol.c
    new_protocol.c
    new_protocol_programmer.c
    rx_panadapter.c
    tx_panadapter.c
    property.c
    radio.c
    receiver.c
    rigctl.c
    rigctl_menu.c
    toolbar.c
    transmitter.c
    zoompan.c
    sliders.c
    version.c
    vfo.c
    waterfall.c
    button_text.c
    vox.c
    update.c
    store.c
    store_menu.c
    memory.c
    led.c
    ext.c
    error_handler.c
    cwramp.c
    protocols.c
)
include_directories(${LIBFFTW_INCLUDE_DIRS})
include_directories(${LIBGTK_INCLUDE_DIRS})
include_directories("${CMAKE_SOURCE_DIR}")
include_directories("${CMAKE_SOURCE_DIR}/wdsp")

#add_definitions(-DCLIENT_SERVER)
add_definitions(-DFREQUENCY_INFO)
add_definitions(-DGIT_DATE="${GIT_DATE}")
add_definitions(-DGIT_VERSION="${GIT_VERSION}")

target_link_directories(pihpsdr PUBLIC "${LIBFFTW_LIBRARY_DIRS}")
target_link_libraries(pihpsdr PUBLIC ${LIBFFTW_LIBRARIES})

target_link_directories(pihpsdr PUBLIC "${LIBGTK_LIBRARY_DIRS}")
target_link_libraries(pihpsdr PUBLIC ${LIBGTK_LIBRARIES})

target_link_directories(pihpsdr PUBLIC "${LIBALSA_LIBRARY_DIRS}")
target_link_libraries(pihpsdr PUBLIC ${LIBALSA_LIBRARIES})

target_link_libraries(pihpsdr PUBLIC m)
target_link_libraries(pihpsdr PUBLIC pthread)

add_executable(hpsdrsim hpsdrsim.c newhpsdrsim.c)
target_link_libraries(hpsdrsim PUBLIC m)
target_link_libraries(hpsdrsim PUBLIC pthread)