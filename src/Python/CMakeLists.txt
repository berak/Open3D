# Require Python.
find_package(PythonInterp REQUIRED)

set(PACKAGE_NAME open3d)

file(GLOB PY_ALL_HEADER_FILES "*.h")
file(GLOB PY_ALL_SOURCE_FILES "*.cpp")
file(GLOB PY_CORE_HEADER_FILES "Core/*.h")
source_group("Header Files\\Core" FILES ${PY_CORE_HEADER_FILES})
file(GLOB PY_CORE_SOURCE_FILES "Core/*.cpp")
source_group("Source Files\\Core" FILES ${PY_CORE_SOURCE_FILES})
file(GLOB PY_IO_HEADER_FILES "IO/*.h")
source_group("Header Files\\IO" FILES ${PY_IO_HEADER_FILES})
file(GLOB PY_IO_SOURCE_FILES "IO/*.cpp")
source_group("Source Files\\IO" FILES ${PY_IO_SOURCE_FILES})
file(GLOB PY_VISUALIZATION_HEADER_FILES "Visualization/*.h")
source_group("Header Files\\Visualization" FILES ${PY_VISUALIZATION_HEADER_FILES})
file(GLOB PY_VISUALIZATION_SOURCE_FILES "Visualization/*.cpp")
source_group("Source Files\\Visualization" FILES ${PY_VISUALIZATION_SOURCE_FILES})
pybind11_add_module(${PACKAGE_NAME}
    ${PY_ALL_HEADER_FILES} ${PY_ALL_SOURCE_FILES}
    ${PY_CORE_SOURCE_FILES} ${PY_CORE_HEADER_FILES}
    ${PY_IO_SOURCE_FILES} ${PY_IO_HEADER_FILES}
    ${PY_VISUALIZATION_SOURCE_FILES} ${PY_VISUALIZATION_HEADER_FILES}
)

target_link_libraries(${PACKAGE_NAME} PRIVATE ${CMAKE_PROJECT_NAME})

file(GLOB PY_HELPER_FILES "Helper/*.py")
add_custom_target(PythonHelper ALL SOURCES ${PY_HELPER_FILES})
set_target_properties(PythonHelper PROPERTIES FOLDER "Python")
add_custom_command(TARGET PythonHelper POST_BUILD
COMMAND ${CMAKE_COMMAND} -E copy_directory "${PROJECT_SOURCE_DIR}/Python/Helper/" "${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/${CMAKE_CFG_INTDIR}")
add_dependencies(${PACKAGE_NAME} PythonHelper)

if (BUILD_PYTHON_TESTS)
    file(GLOB PY_TEST_FILES "Test/*.py")
    add_custom_target(PythonTest ALL SOURCES ${PY_TEST_FILES})
    set_target_properties(PythonTest PROPERTIES FOLDER "Python")
    add_custom_command(TARGET PythonTest POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_directory "${PROJECT_SOURCE_DIR}/Python/Test/" "${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/${CMAKE_CFG_INTDIR}/Test")
    add_custom_command(TARGET PythonTest POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_directory "${PROJECT_SOURCE_DIR}/Test/TestData/" "${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/${CMAKE_CFG_INTDIR}/TestData")
    add_dependencies(${PACKAGE_NAME} PythonTest)
endif (BUILD_PYTHON_TESTS)

if (BUILD_PYTHON_TUTORIALS)
    file(GLOB PY_BASIC_TUTORIAL_FILES "Tutorial/Basic/*.py")
    source_group("Basic" FILES ${PY_BASIC_TUTORIAL_FILES})
    file(GLOB PY_ADVANCED_TUTORIAL_FILES "Tutorial/Advanced/*.py")
    source_group("Advanced" FILES ${PY_ADVANCED_TUTORIAL_FILES})
    file(GLOB PY_APPLICATION_TUTORIAL_FILES "Tutorial/Application/*.py")
    source_group("Application" FILES ${PY_APPLICATION_TUTORIAL_FILES})
    file(GLOB PY_RECONSTRUCTION_SYSTEM_TUTORIAL_FILES "Tutorial/ReconstructionSystem/*.py")
    source_group("ReconstructionSystem" FILES ${PY_RECONSTRUCTION_SYSTEM_TUTORIAL_FILES})
    add_custom_target(PythonTutorial ALL SOURCES ${PY_BASIC_TUTORIAL_FILES} ${PY_ADVANCED_TUTORIAL_FILES} ${PY_APPLICATION_TUTORIAL_FILES} ${PY_RECONSTRUCTION_SYSTEM_TUTORIAL_FILES})
    set_target_properties(PythonTutorial PROPERTIES FOLDER "Python")
    add_custom_command(TARGET PythonTutorial POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_directory "${PROJECT_SOURCE_DIR}/Python/Tutorial/" "${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/${CMAKE_CFG_INTDIR}/Tutorial")
    if (NOT BUILD_PYTHON_TESTS)
        add_custom_command(TARGET PythonTutorial POST_BUILD
            COMMAND ${CMAKE_COMMAND} -E copy_directory "${PROJECT_SOURCE_DIR}/Test/TestData/" "${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/${CMAKE_CFG_INTDIR}/TestData")
    endif (NOT BUILD_PYTHON_TESTS)
    add_dependencies(${PACKAGE_NAME} PythonTutorial)
endif (BUILD_PYTHON_TUTORIALS)

set_target_properties(${PACKAGE_NAME} PROPERTIES
                      FOLDER "Python"
                      LIBRARY_OUTPUT_DIRECTORY "${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/Python"
                      ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_ARCHIVE_OUTPUT_DIRECTORY}/Python")

# find the python site-packages location
execute_process(COMMAND ${PYTHON_EXECUTABLE} -m site --user-site OUTPUT_VARIABLE PYTHON_SITE_PACKAGES OUTPUT_STRIP_TRAILING_WHITESPACE)

# configure installer to copy the python module to the python site-packages
install(TARGETS ${PACKAGE_NAME} DESTINATION ${PYTHON_SITE_PACKAGES})
