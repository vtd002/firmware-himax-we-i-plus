cmake_minimum_required(VERSION 3.12)

# set source directory name as the project name
get_filename_component(ProjectId ${CMAKE_CURRENT_SOURCE_DIR} NAME)
string(REPLACE " " "_" ProjectId ${ProjectId})
project(${ProjectId})


# add the sources
file(GLOB EI_SDK "edge-impulse-sdk/porting/himax/*.cpp"
                 "ingestion-sdk-platform/himax/*.cpp"
                 "ingestion-sdk-platform/himax/*.cpp"
                 "ingestion-sdk-c/*.cpp"
                 "repl/*.cpp"
                 "repl/*.c"
                 "sensors/*.cpp"
                 "QCBOR/src/*.c"
                 "mbedtls_hmac_sha256_sw/mbedtls/src/*.c"
                 "CMSIS/DSP/Source/TransformFunctions/*fft*.c"
                 "CMSIS/DSP/Source/CommonTables/*.c"
                 "CMSIS/DSP/Source/TransformFunctions/*bit*.c")

file(GLOB FW_SDK "firmware-sdk/jpeg/*.cpp"
                  "firmware-sdk/at_base64_lib.cpp")

file(GLOB TF "edge-impulse-sdk/tensorflow/lite/micro/*.cc"
             "edge-impulse-sdk/tensorflow/lite/micro/memory_planner/*.cc"
             "edge-impulse-sdk/tensorflow/lite/micro/kernels/*.cc"
             "edge-impulse-sdk/tensorflow/lite/core/api/*.cc"
             "edge-impulse-sdk/tensorflow/lite/c/*.c"
             "edge-impulse-sdk/tensorflow/lite/kernels/internal/*.cc"
             "edge-impulse-sdk/tensorflow/lite/kernels/*.cc"
             "edge-impulse-sdk/tensorflow/lite/micro/kernels/arc_mli/*.cc"
             "tflite-model/*.cpp")

file(GLOB DSP "edge-impulse-sdk/dsp/kissfft/*.cpp"
            "edge-impulse-sdk/dsp/dct/*.cpp"
            "edge-impulse-sdk/dsp/image/*.cpp")

set(SOURCE_FILES ${EI_SDK} ${FW_SDK} ${TF} ${DSP})

# include directories
include_directories(./)
include_directories(edge-impulse-sdk/)
include_directories(edge-impulse-sdk/classifier)
include_directories(edge-impulse-sdk/porting)
include_directories(edge-impulse-sdk/dsp)
include_directories(edge-impulse-sdk/dsp/kissfft)
include_directories(edge-impulse-sdk/dsp/dct)
include_directories(edge-impulse-sdk/dsp/image)
include_directories(edge-impulse-sdk/third_party/flatbuffers/include/)
include_directories(edge-impulse-sdk/third_party/gemmlowp)
include_directories(edge-impulse-sdk/third_party/ruy)
include_directories(edge-impulse-sdk/CMSIS/DSP/Include/)
include_directories(edge-impulse-sdk/CMSIS/Core/Include/)
include_directories(ingestion-sdk-c)
include_directories(arc_mli_package/include)
include_directories(ingestion-sdk-c)
include_directories(ingestion-sdk-platform/himax)
include_directories(repl)
include_directories(sensors)
include_directories(QCBOR/inc)
include_directories(mbedtls_hmac_sha256_sw)
include_directories(firwmare-sdk)
include_directories(firwmare-sdk/jpeg)


# add the executable
add_executable(${PROJECT_NAME}.elf main.cc ei_run_impulse.cpp ${SOURCE_FILES})

target_link_libraries(${PROJECT_NAME}.elf ${MY_LINKER_FLAGS})

add_custom_target(image DEPENDS ${PROJECT_NAME}.elf)
add_custom_command(TARGET image USES_TERMINAL
    COMMAND cp ${PROJECT_NAME}.elf ../image_gen_linux_v3
    COMMAND cp ${PROJECT_NAME}.map ../image_gen_linux_v3
    COMMAND ${CMAKE_COMMAND} ARGS -E chdir ../image_gen_linux_v3 image_gen -e ${PROJECT_NAME}.elf -m ${PROJECT_NAME}.map -o out.img
    COMMAND ${CMAKE_COMMAND} ARGS -E chdir ../image_gen_linux_v3 rm ${PROJECT_NAME}.elf
    COMMAND ${CMAKE_COMMAND} ARGS -E chdir ../image_gen_linux_v3 rm ${PROJECT_NAME}.map)
