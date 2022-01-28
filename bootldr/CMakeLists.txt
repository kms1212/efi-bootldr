project(efi-bootloader)

file(GLOB EFI_SOURCES "${CMAKE_SOURCE_DIR}/bootldr/src/*")

execute_process(COMMAND uname -m OUTPUT_VARIABLE SYS_ARCH)
string(REPLACE "\n" " " SYS_ARCH ${SYS_ARCH})
string(STRIP ${SYS_ARCH} SYS_ARCH)

set(GNUEFI "${CMAKE_SOURCE_DIR}/gnu-efi-code")
set(CFLAGS -I ${GNUEFI}/inc -fPIC -fno-stack-protector -fshort-wchar -mno-red-zone -DEFI_FUNCTION_WRAPPER -Wall -Werror)
set(LDFLAGS -nostdlib -znocombreloc -shared -Bsymbolic -T ${GNUEFI}/gnuefi/elf_${SYS_ARCH}_efi.lds -L ${GNUEFI})
set(OCFLAGS -j .text -j .sdata -j .data -j .dynamic -j .dynsym -j .rel -j .rela -j .rel.* -j .rela.* -j .reloc --target efi-app-${SYS_ARCH} --subsystem=10)

add_custom_target(efibl ALL
                  DEPENDS gnu-efi
                  WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
                  COMMENT "Building efibl..."
                  VERBATIM)
foreach(SRCFILE ${EFI_SOURCES})
    get_filename_component(OBJFILE_CMD ${SRCFILE} NAME)
    add_custom_command(TARGET efibl PRE_BUILD
                       COMMAND ${CMAKE_C_COMPILER} ${CFLAGS} -c ${SRCFILE} -o ${CMAKE_CURRENT_BINARY_DIR}/${OBJFILE_CMD}.o
                       COMMENT "Building C Object ${SRCFILE}"
                       VERBATIM)
    set(OBJFILE "${OBJFILE} ${CMAKE_CURRENT_BINARY_DIR}/${OBJFILE_CMD}.o")
endforeach()
set(OBJFILE "${OBJFILE} ${GNUEFI}/${SYS_ARCH}/gnuefi/crt0-efi-${SYS_ARCH}.o")
separate_arguments(OBJFILE)
add_custom_command(TARGET efibl POST_BUILD
                   COMMAND ${CMAKE_LINKER} ${LDFLAGS} ${OBJFILE} -o ${CMAKE_CURRENT_BINARY_DIR}/efibl.so
                   COMMENT "Linking temporary shared library efibl.so"
                   VERBATIM)
add_custom_command(TARGET efibl POST_BUILD
                   COMMAND ${CMAKE_OBJCOPY} ${OCFLAGS} ${CMAKE_CURRENT_BINARY_DIR}/efibl.so ${CMAKE_CURRENT_BINARY_DIR}/efibl.efi
                   COMMENT "Creating efi application efibl.efi"
                   VERBATIM)

add_dependencies(efibl gnu-efi)