# CMake snippet for building elf2cfetbl
#

project(CFS_TABLETOOL C)

add_executable(elf2cfetbl elf2cfetbl.c)

# Export relevant information so the parent script can invoke the tool
set(CFS_TABLETOOL_SCRIPT_DIR "${CMAKE_CURRENT_SOURCE_DIR}/scripts" CACHE INTERNAL "CFS table tool script directory")

add_custom_target(tabletool-execute
    COMMAND $(MAKE)
        CC="${CMAKE_C_COMPILER}"
        CFLAGS="${CMAKE_C_FLAGS}"

        # The CMake rules attempt to extract objects out of static archives
        # They are built using the toolchain-supplied ar binary, so we need
        # to use it to do extraction as opposed to the system ar binary
        AR="aarch64-elf-ar"

        TBLTOOL="$<TARGET_FILE:elf2cfetbl>"
        cfetables
    WORKING_DIRECTORY
        "${MISSION_BINARY_DIR}/tables"
    DEPENDS
        mission-cfetables
        elf2cfetbl
)

add_dependencies(mission-all tabletool-execute)
add_dependencies(mission-install tabletool-execute)
add_dependencies(mission-prebuild elf2cfetbl)

install(DIRECTORY ${CMAKE_BINARY_DIR}/tables/staging/ DESTINATION .)


install(TARGETS elf2cfetbl DESTINATION host)
