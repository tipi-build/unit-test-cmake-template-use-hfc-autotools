cmake_minimum_required(VERSION 3.27.6)
project(print_something)

# make cache variables for install destinations
include(GNUInstallDirs)
include(FetchContent)


# specify the C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

set(HFC_BASE_DIR "")
if(HERMETIC_FETCHCONTENT_SOURCE_CACHE_DIR)
  set(HFC_BASE_DIR "${HERMETIC_FETCHCONTENT_SOURCE_CACHE_DIR}/")
else()
  set(HFC_BASE_DIR "${CMAKE_CURRENT_LIST_DIR}/")
endif()

# Provisioning hfc if not already provisioned by parent project / proxy toolchain
include(HermeticFetchContent OPTIONAL RESULT_VARIABLE hfc_included) 
if(NOT hfc_included)
  set(hfc_REPOSITORY https://github.com/tipi-build/hfc)
  set(hfc_REVISION a56de21daf429cf54d9c087524086277480a4eaf) # release v1.0.4

  FetchContent_Populate(
    hermetic_fetchcontent
    GIT_REPOSITORY ${hfc_REPOSITORY}
    GIT_TAG ${hfc_REVISION}
    SOURCE_DIR "${HFC_BASE_DIR}/thirdparty/cache/hfc-${hfc_REVISION}/src"
    SUBBUILD_DIR "${HFC_BASE_DIR}/thirdparty/cache/hfc-${hfc_REVISION}/subbuild"
    BINARY_DIR "${HFC_BASE_DIR}/thirdparty/cache/hfc-${hfc_REVISION}/bin"
  )

  FetchContent_GetProperties(hermetic_fetchcontent)

  message(STATUS "Hermetic FetchContent ${hfc_REVISION} available at '${hermetic_fetchcontent_SOURCE_DIR}'")
  list(APPEND CMAKE_MODULE_PATH "${hermetic_fetchcontent_SOURCE_DIR}/cmake")
  include(HermeticFetchContent)
endif()

FetchContent_Declare(
  Iconv
  GIT_REPOSITORY "https://github.com/tipi-build/unittest-autotools-sample.git"
  GIT_TAG "ad80b024eeda8f4c0a96eedf669dc453ed33a094"
)

FetchContent_MakeHermetic(
  Iconv
  HERMETIC_CMAKE_EXPORT_LIBRARY_DECLARATION 
  [=[
    add_library(Iconv::Iconv STATIC IMPORTED)
    set_property(TARGET Iconv::Iconv PROPERTY IMPORTED_LOCATION "@HFC_PREFIX_PLACEHOLDER@/lib/libiconv.a")    
    set_property(TARGET Iconv::Iconv PROPERTY INTERFACE_INCLUDE_DIRECTORIES @HFC_PREFIX_PLACEHOLDER@/include)
  ]=]
  HERMETIC_BUILD_SYSTEM autotools
)

HermeticFetchContent_MakeAvailableAtConfigureTime("Iconv")


add_library(print_hello_tipi STATIC print_hello_tipi.cxx)

target_include_directories(print_hello_tipi
                           PUBLIC
                           "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>"
                           "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>"
)
target_link_libraries(print_hello_tipi PRIVATE Iconv::Iconv)


install(TARGETS print_hello_tipi
        EXPORT print_hello_tipiTargets
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

install(FILES print_hello_tipi.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
install(EXPORT print_hello_tipiTargets
        FILE print_hello_tipiTargets.cmake
        NAMESPACE print_hello_tipi::
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/print_hello_tipi
)







##"lib.h", "MathFunctionscbrt.h"
