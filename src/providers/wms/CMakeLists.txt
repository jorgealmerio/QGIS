set (WMS_SRCS
  qgswmscapabilities.cpp
  qgswmsprovider.cpp
  qgswmsconnection.cpp
  qgswmsdataitems.cpp
  qgsxyzconnection.cpp
)

if (WITH_GUI)
  set(WMS_GUI_SRCS
    qgswmsprovidergui.cpp
    qgswmssourceselect.cpp
    qgswmsdataitemguiproviders.cpp
    qgstilescalewidget.cpp
    qgswmtsdimensions.cpp
    qgsxyzconnectiondialog.cpp
    qgsxyzsourceselect.cpp
    qgsxyzsourcewidget.cpp
  )
endif()


# static library
add_library(wmsprovider_a STATIC ${WMS_SRCS})

if (WITH_GUI)
  add_library(wmsprovider_gui_a STATIC ${WMS_GUI_SRCS})

  add_dependencies(wmsprovider_a ui)
  add_dependencies(wmsprovider_gui_a ui)
endif()

target_link_libraries(wmsprovider_a
  qgis_core
  ${Qt5Xml_LIBRARIES}
  ${Qt5Core_LIBRARIES}
  ${Qt5Svg_LIBRARIES}
  ${Qt5Network_LIBRARIES}
  ${Qt5Sql_LIBRARIES}
  ${Qt5Concurrent_LIBRARIES}
)
target_compile_definitions(wmsprovider_a PRIVATE "-DQT_NO_FOREACH")

if (WITH_GUI)
  target_link_libraries(wmsprovider_gui_a
    qgis_gui
    wmsprovider_a
    ${Qt5Xml_LIBRARIES}
    ${Qt5Core_LIBRARIES}
    ${Qt5Svg_LIBRARIES}
    ${Qt5Network_LIBRARIES}
    ${Qt5Sql_LIBRARIES}
    ${Qt5Concurrent_LIBRARIES}
    ${Qt5Gui_LIBRARIES}
    ${Qt5Widgets_LIBRARIES}
  )
  target_compile_definitions(wmsprovider_gui_a PRIVATE "-DQT_NO_FOREACH")
endif()

if (FORCE_STATIC_PROVIDERS)
  # for (external) mobile apps to be able to pick up provider for linking
  install (TARGETS wmsprovider_a ARCHIVE DESTINATION ${QGIS_PLUGIN_DIR})
  if (WITH_GUI)
    install (TARGETS wmsprovider_gui_a ARCHIVE DESTINATION ${QGIS_PLUGIN_DIR})
  endif()
else()
  # dynamically loaded module
  add_library(wmsprovider MODULE ${WMS_SRCS} ${WMS_GUI_SRCS})

  target_link_libraries(wmsprovider
    qgis_core
    ${GDAL_LIBRARY}  # for OGR_G_CreateGeometryFromJson()
  )

  if (WITH_GUI)
    target_link_libraries (wmsprovider
      qgis_gui
    )
    add_dependencies(wmsprovider ui)
  endif()

  target_compile_definitions(wmsprovider PRIVATE "-DQT_NO_FOREACH")

  install (TARGETS wmsprovider
    RUNTIME DESTINATION ${QGIS_PLUGIN_DIR}
    LIBRARY DESTINATION ${QGIS_PLUGIN_DIR})

endif()
