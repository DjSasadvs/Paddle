set(OUTPUT_DIR
    "${CMAKE_CURRENT_BINARY_DIR}/build")

file(GLOB TRAINER_PY_FILES . ./paddle/trainer/*.py)
file(GLOB HELPERS_PY_FILES . ./paddle/trainer_config_helpers/*.py)
file(GLOB UTILS_PY_FILES . ./paddle/utils/*.py)
file(GLOB_RECURSE V2_PY_FILES ./paddle/v2/ *.py)

set(PY_FILES paddle/__init__.py
  ${TRAINER_PY_FILES}
  ${HELPERS_PY_FILES}
  ${UTILS_PY_FILES}
  ${V2_PY_FILES})

add_custom_target(copy_paddle_master)

SET(COPY_PADDLE_MASTER "")
if(WITH_GOLANG)
  SET(COPY_PADDLE_MASTER "copy_paddle_master")
  add_custom_command(TARGET ${COPY_PADDLE_MASTER}
    COMMAND cp ${paddle_master_LIB_PATH} ${PROJ_ROOT}/python/paddle/v2/master/
    )
  add_dependencies(copy_paddle_master paddle_master)
endif(WITH_GOLANG)

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/setup.py.in
    ${CMAKE_CURRENT_BINARY_DIR}/setup.py)

add_custom_command(OUTPUT ${OUTPUT_DIR}/.timestamp
    COMMAND env ${py_env} ${PYTHON_EXECUTABLE} setup.py bdist_wheel
    COMMAND ${CMAKE_COMMAND} -E touch ${OUTPUT_DIR}/.timestamp
    DEPENDS gen_proto_py framework_py_proto ${PY_FILES} ${external_project_dependencies} ${COPY_PADDLE_MASTER})

add_custom_target(paddle_python ALL DEPENDS
    ${OUTPUT_DIR}/.timestamp)

set(PADDLE_PYTHON_PACKAGE_DIR ${CMAKE_CURRENT_BINARY_DIR}/dist/)

if (WITH_TESTING)
  add_subdirectory(paddle/trainer_config_helpers/tests)
  if (WITH_SWIG_PY)
    # enable v2 API unittest only when paddle swig api is compiled
    add_subdirectory(paddle/v2/tests)
    add_subdirectory(paddle/v2/reader/tests)
    add_subdirectory(paddle/v2/plot/tests)
    add_subdirectory(paddle/v2/framework/tests)
  endif()
endif()
install(DIRECTORY ${PADDLE_PYTHON_PACKAGE_DIR}
    DESTINATION opt/paddle/share/wheels
)
