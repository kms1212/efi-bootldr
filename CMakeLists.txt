cmake_minimum_required(VERSION 3.12)

project(root)

add_custom_target(gnu-efi ALL 
                  make
                  WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}/gnu-efi-code
                  COMMENT "Building gnu-efi..."
                  VERBATIM)

add_subdirectory(bootldr)
