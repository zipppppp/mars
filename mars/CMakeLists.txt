cmake_minimum_required (VERSION 3.6)

set(CMAKE_INSTALL_PREFIX "${CMAKE_BINARY_DIR}" CACHE PATH "Installation directory" FORCE)
message(STATUS "CMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}")

add_subdirectory(comm comm)
add_subdirectory(boost boost)
add_subdirectory(app app)
add_subdirectory(baseevent baseevent)
add_subdirectory(log xlog)
add_subdirectory(sdt sdt)
add_subdirectory(stn stn)


project (mars)

include(comm/CMakeUtils.txt)

include_directories(.)
include_directories(..)

set(SELF_LIBS_OUT ${CMAKE_SYSTEM_NAME}.out)

if(ANDROID)

    find_library(log-lib log)
    find_library(z-lib z)

    link_directories(app baseevent log sdt stn comm boost)

    # marsxlog
    set(SELF_LIB_NAME marsxlog)

    file(GLOB SELF_SRC_FILES libraries/mars_android_sdk/jni/JNI_OnLoad.cc
            libraries/mars_xlog_sdk/jni/import.cc)

    add_library(${SELF_LIB_NAME} SHARED ${SELF_SRC_FILES})
    install(TARGETS ${SELF_LIB_NAME} LIBRARY DESTINATION ${SELF_LIBS_OUT} ARCHIVE DESTINATION ${SELF_LIBS_OUT})

    get_filename_component(EXPORT_XLOG_EXP_FILE log/jni/export.exp ABSOLUTE)
    target_link_libraries(${SELF_LIB_NAME} "-Wl,--gc-sections -Wl,--version-script='${EXPORT_XLOG_EXP_FILE}'"
                            ${log-lib}
                            ${z-lib}
                            xlog
                            mars-boost
                            comm)


    set(SELF_LIB_NAME marsstn)
    get_filename_component(EXPORT_EXP_FILE libraries/mars_android_sdk/jni/export.exp ABSOLUTE)

    file(GLOB SELF_SRC_FILES libraries/mars_android_sdk/jni/*.cc)
    set (CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,--version-script='${EXPORT_EXP_FILE}'")

    add_library(${SELF_LIB_NAME} SHARED ${SELF_SRC_FILES})


    install(TARGETS ${SELF_LIB_NAME} LIBRARY DESTINATION ${SELF_LIBS_OUT} ARCHIVE DESTINATION ${SELF_LIBS_OUT})

    link_directories(${SELF_LIBS_OUT})

    target_link_libraries(${SELF_LIB_NAME}
                            ${log-lib}
                            stn
                            sdt
                            app
                            baseevent
                            comm
                            mars-boost
                            marsxlog)
elseif(APPLE)

endif()




