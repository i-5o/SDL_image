add_executable(testimage main.c)

set(ALL_TESTS
    testimage
)
set(RESOURCE_FILES
    palette.bmp
    palette.gif
    sample.avif
    sample.bmp
    sample.cur
    sample.ico
    sample.jpg
    sample.jxl
    sample.png
    sample.pnm
    sample.qoi
    sample.tif
    sample.webp
    sample.xcf
    sample.xpm
    svg.bmp
    svg.svg
    svg64.bmp
)

set(TESTS_ENVIRONMENT
    "SDL_TEST_SRCDIR=${CMAKE_CURRENT_SOURCE_DIR}"
    "SDL_TEST_BUILDDIR=${CMAKE_CURRENT_BINARY_DIR}"
    "SDL_VIDEODRIVER=dummy"
)

foreach(prog ${ALL_TESTS})
    target_compile_definitions(${prog} PRIVATE $<TARGET_PROPERTY:SDL2_image,COMPILE_DEFINITIONS>)
    target_link_libraries(${prog} PRIVATE SDL2_image::${sdl2_image_export_name})
    if (TARGET SDL2::SDL2main)
        target_link_libraries(${prog} PRIVATE SDL2::SDL2main)
    endif()
    target_link_libraries(${prog} PRIVATE SDL2_test)
    if (BUILD_SHARED_LIBS)
        target_link_libraries(${prog} PRIVATE SDL2::SDL2)
    else()
        target_link_libraries(${prog} PRIVATE SDL2::SDL2-static)
    endif()

    add_test(
        NAME ${prog}
        COMMAND ${prog}
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
    )
    set_tests_properties(
        ${prog}
        PROPERTIES ENVIRONMENT "${TESTS_ENVIRONMENT}"
    )
    if(INSTALL_TESTS)
        set(exe ${prog})
        set(installedtestsdir "${CMAKE_INSTALL_FULL_LIBEXECDIR}/installed-tests/${CMAKE_PROJECT_NAME}")
        configure_file(template.test.in "${exe}.test" @ONLY)
        install(
            FILES "${CMAKE_CURRENT_BINARY_DIR}/${exe}.test"
            DESTINATION "${CMAKE_INSTALL_DATADIR}/installed-tests/${CMAKE_PROJECT_NAME}"
        )
    endif()
endforeach()

if(INSTALL_TESTS)
    install(
        TARGETS ${ALL_TESTS}
        DESTINATION "${CMAKE_INSTALL_LIBEXECDIR}/installed-tests/${CMAKE_PROJECT_NAME}"
    )
    install(
        FILES ${RESOURCE_FILES}
        DESTINATION "${CMAKE_INSTALL_LIBEXECDIR}/installed-tests/${CMAKE_PROJECT_NAME}"
    )
endif()