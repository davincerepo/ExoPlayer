#
# Copyright 2021 The Android Open Source Project
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

cmake_minimum_required(VERSION 3.21.0 FATAL_ERROR)
#message (FATAL_ERROR "${CMAKE_BUILD_TYPE}")
# Enable C++11 features.
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_BUILD_TYPE Release)

project(libffmpegJNI C CXX)

# Additional flags needed for "arm64-v8a" from NDK 23.1.7779620 and above.
# See https://github.com/google/ExoPlayer/issues/9933#issuecomment-1029775358.
if(${ANDROID_ABI} MATCHES "arm64-v8a")
    set(CMAKE_CXX_FLAGS "-Wl,-Bsymbolic")
endif()

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O3")

# libyuv
set(ndklibyuv_location "${CMAKE_CURRENT_SOURCE_DIR}/../../../ndklibyuv")
add_library(libyuv SHARED IMPORTED)
set_target_properties(libyuv PROPERTIES
    IMPORTED_LOCATION "${ndklibyuv_location}/buildout/intermediates/ndkBuild/release/obj/local/arm64-v8a/libyuv.so"
)
include_directories("${ndklibyuv_location}/src/main/jni/libyuv/include")

#libdav1d
set(libdav1d_location "${CMAKE_CURRENT_SOURCE_DIR}/dav1d")
add_library(libdav1d SHARED IMPORTED)
set_target_properties(libdav1d PROPERTIES
    IMPORTED_LOCATION "${libdav1d_location}/lib/libdav1d.so"
)

# ffmpeg
set(ffmpeg_location "${CMAKE_CURRENT_SOURCE_DIR}/ffmpeg")
set(ffmpeg_binaries "${ffmpeg_location}/android-libs/${ANDROID_ABI}")
foreach(ffmpeg_lib avutil swresample avcodec)
    set(ffmpeg_lib_filename lib${ffmpeg_lib}.so)
    set(ffmpeg_lib_file_path ${ffmpeg_binaries}/${ffmpeg_lib_filename})
    add_library(
            ${ffmpeg_lib}
            STATIC
            IMPORTED)
    set_target_properties(
            ${ffmpeg_lib} PROPERTIES
            IMPORTED_LOCATION
            ${ffmpeg_lib_file_path})
endforeach()

include_directories(${ffmpeg_location})
find_library(android_log_lib log)

add_library(ffmpeg_jni
        SHARED
        ffmpeg_jni.cc)

target_link_libraries(ffmpeg_jni
        PRIVATE android
        PRIVATE avutil
        PRIVATE swresample
        PRIVATE avcodec
        PRIVATE ${android_log_lib}
        PRIVATE libyuv
        PRIVATE libdav1d  
        )
