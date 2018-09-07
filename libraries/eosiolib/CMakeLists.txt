file(GLOB HEADERS "*.hpp"
                  "*.h")

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/core_symbol.hpp.in ${CMAKE_CURRENT_BINARY_DIR}/core_symbol.hpp)

add_library(eosio 
            eosiolib.cpp 
            ${HEADERS})

target_include_directories(eosio PUBLIC 
                                 "$<BUILD_INTERFACE:${STANDARD_INCLUDES}>"
                                 $<INSTALL_INTERFACE:${CMAKE_INSTALL_FULL_INCLUDEDIR}>)

target_link_libraries( eosio c c++ )

install(TARGETS eosio EXPORT EosioLib
   LIBRARY DESTINATION ./lib 
   ARCHIVE DESTINATION ./lib)

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/../eosiolib DESTINATION ./include FILES_MATCHING PATTERN "*.h" PATTERN "*.hpp")
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/core_symbol.hpp DESTINATION ./include/eosiolib)
