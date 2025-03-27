cmake_minimum_required(VERSION 3.16)

project(EEPROM VERSION 0.1 LANGUAGES CXX)

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CMAKE_PREFIX_PATH "C:/Qt/6.9.0")
set(MINGW_PATH "C:/Qt/Tools/mingw1310_64")

find_package(Qt6 REQUIRED COMPONENTS
        Core
        Gui
        Widgets
        SerialPort)

file(GLOB SRC_FILES *.cpp *.qrc)
file(GLOB INC_FILES *.h)


set(PROJECT_SOURCES
        ${SRC_FILES}
        ${INC_FILES}
)

qt_add_executable(EEPROM
        MANUAL_FINALIZATION
        ${PROJECT_SOURCES}
    )

target_link_libraries(EEPROM PRIVATE Qt6::Core Qt6::Gui Qt6::Widgets Qt6::SerialPort)

include(GNUInstallDirs)
install(TARGETS EEPROM
    BUNDLE DESTINATION .
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

qt_finalize_executable(EEPROM)

if (WIN32)
    set(QT_INSTALL_PATH "${CMAKE_PREFIX_PATH}")
    if (NOT EXISTS "${QT_INSTALL_PATH}/bin")
        set(QT_INSTALL_PATH "${QT_INSTALL_PATH}/..")
        if (NOT EXISTS "${QT_INSTALL_PATH}/bin")
            set(QT_INSTALL_PATH "${QT_INSTALL_PATH}/..")
        endif ()
    endif ()
    if (EXISTS "${QT_INSTALL_PATH}/plugins/platforms/qwindows.dll")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E make_directory
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/platforms/")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy
                "${QT_INSTALL_PATH}/plugins/platforms/qwindows.dll"
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/platforms/")
    endif ()
    foreach (QT_LIB Core Gui Widgets SerialPort)
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy
                "${QT_INSTALL_PATH}/bin/Qt6${QT_LIB}.dll"
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>")
    endforeach (QT_LIB)
    foreach (MINGW_DLL libgcc_s_seh-1 libstdc++-6 libwinpthread-1)
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy
                "${MINGW_PATH}/bin/${MINGW_DLL}.dll"
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>")
    endforeach (MINGW_DLL)
endif ()
