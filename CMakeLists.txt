# CMake support for LZ4
#
# To the extent possible under law, the author(s) have dedicated all
# copyright and related and neighboring rights to this software to
# the public domain worldwide. This software is distributed without
# any warranty.
#
# For details, see <http://creativecommons.org/publicdomain/zero/1.0/>.
#
# LZ4's CMake support is maintained by Evan Nemerson; when filing
# bugs please mention @nemequ to make sure I see it.

cmake_minimum_required(VERSION 3.2)

project(lz4)

add_library(lz4 STATIC
    src/SampleHeader.cpp
    src/SampleHeader.h
)

install(TARGETS lz4
    EXPORT lz4
)
