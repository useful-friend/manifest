# Top-level CMake for cpp_sensor_platform.
# Build all: cmake -B build -S . && cmake --build build
#
# Or build each sensor separately:
#   cd sensors/cts-gen3 && mkdir build && cd build && cmake .. && make
#   cd sensors/radar    && mkdir build && cd build && cmake .. && make
#   cd fusion          && mkdir build && cd build && cmake .. && make

cmake_minimum_required(VERSION 3.16)
project(sensor_platform LANGUAGES CXX VERSION 0.1.0)

# Build sensors and fusion
add_subdirectory(hal)
add_subdirectory(transport)
add_subdirectory(sensors/cts-gen3)
# add_subdirectory(sensors/radar)
# add_subdirectory(fusion)
