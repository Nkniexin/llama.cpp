add_library(llava OBJECT
            llava.cpp
            llava.h
            clip.cpp
            clip.h
            )

target_link_libraries(llava PRIVATE ggml llama ${CMAKE_THREAD_LIBS_INIT})

target_include_directories(llava PUBLIC .)
target_include_directories(llava PUBLIC ../..)
target_include_directories(llava PUBLIC ../../common)

target_compile_features(llava PRIVATE cxx_std_11)

add_library(llava_static STATIC $<TARGET_OBJECTS:llava>)
if (BUILD_SHARED_LIBS)
    set_target_properties(llava PROPERTIES POSITION_INDEPENDENT_CODE ON)
    target_compile_definitions(llava PRIVATE LLAMA_SHARED LLAMA_BUILD)
    add_library(llava_shared SHARED $<TARGET_OBJECTS:llava>)
    target_link_libraries(llava_shared PRIVATE ggml llama ${CMAKE_THREAD_LIBS_INIT})
    install(TARGETS llava_shared LIBRARY)
endif()

if (NOT MSVC)
    target_compile_options(llava PRIVATE -Wno-cast-qual) # stb_image.h
endif()

if(TARGET BUILD_INFO)
    add_dependencies(llava BUILD_INFO)
endif()

set(TARGET llama-llava-cli)
add_executable(${TARGET} llava-cli.cpp)
set_target_properties(${TARGET} PROPERTIES OUTPUT_NAME llama-llava-cli)
install(TARGETS ${TARGET} RUNTIME)
target_link_libraries(${TARGET} PRIVATE common llava ${CMAKE_THREAD_LIBS_INIT})
target_compile_features(${TARGET} PRIVATE cxx_std_11)

# 设置 FFmpeg路径
set(FFMPEG_DIR "C:/ffmpeg-master-latest-win64-gpl-shared")

# 指定包含目录
include_directories("${FFMPEG_DIR}/include")

# 指定链接库路径
link_directories("${FFMPEG_DIR}/lib")

# 查找 FFmpeg 库
find_library(AVCODEC_LIBRARY avcodec HINTS "${FFMPEG_DIR}/lib")
find_library(AVFORMAT_LIBRARY avformat HINTS "${FFMPEG_DIR}/lib")
find_library(AVUTIL_LIBRARY avutil HINTS "${FFMPEG_DIR}/lib")
find_library(SWSCALE_LIBRARY swscale HINTS "${FFMPEG_DIR}/lib")

set(TARGET llama-minicpmv-cli)
add_executable(${TARGET} minicpmv-cli.cpp)
set_target_properties(${TARGET} PROPERTIES OUTPUT_NAME llama-minicpmv-cli)
install(TARGETS ${TARGET} RUNTIME)
target_link_libraries(${TARGET} PRIVATE common llava ${CMAKE_THREAD_LIBS_INIT})
#链接到 FFmpeg库
target_link_libraries(${TARGET} PRIVATE ${AVCODEC_LIBRARY} ${AVFORMAT_LIBRARY} ${AVUTIL_LIBRARY} ${SWSCALE_LIBRARY})
target_compile_features(${TARGET} PRIVATE cxx_std_11)
