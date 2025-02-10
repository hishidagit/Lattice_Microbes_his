add_library(lmmain STATIC "${CMAKE_CURRENT_LIST_DIR}/common.cpp") 
target_include_directories(lmmain PRIVATE lmcore)
target_link_libraries(lmmain PUBLIC lmcore)

add_executable(lm "${CMAKE_CURRENT_LIST_DIR}/MainSA.cpp") 
target_link_libraries(lm PRIVATE lmmain)

add_executable(lm_setdm "${CMAKE_CURRENT_LIST_DIR}/lm_setdm.cpp")
target_link_libraries(lm_setdm PUBLIC lmcore)

add_executable(lm_setp "${CMAKE_CURRENT_LIST_DIR}/lm_setp.cpp")
target_link_libraries(lm_setp PUBLIC lmcore)

add_executable(lm_setrm "${CMAKE_CURRENT_LIST_DIR}/lm_setrm.cpp")
target_link_libraries(lm_setrm PUBLIC lmcore)

install(TARGETS lm lm_setdm lm_setp lm_setrm RUNTIME DESTINATION bin)

if (OPT_MPI)
     add_executable(mpilm "${CMAKE_CURRENT_LIST_DIR}/MainMPI.cpp")
     target_link_libraries(mpilm PUBLIC lmcore)
     install(TARGETS mpilm RUNTIME DESTINATION bin)
endif ()

# if (TARGET pythonlm)
#     message(STATUS "Target pythonlm is available.")
# else()
#     message(FATAL_ERROR "Target pythonlm is not available. Check add_subdirectory order.")
# endif()


if (OPT_PYTHON)
     add_executable(lm_python "${CMAKE_CURRENT_LIST_DIR}/lm_python.cpp" "${CMAKE_BINARY_DIR}/lm_module_pack.h")
     # target_include_directories(lm_python PRIVATE ${SWIG_MODULE_pythonlmstatic_REAL_NAME})
     # target_link_libraries(lm_python PRIVATE ${SWIG_MODULE_pythonlmstatic_REAL_NAME})
     ## target_include_directories(lm_python PRIVATE ${SWIG_MODULE_pythonlm_REAL_NAME})
     ## target_link_libraries(lm_python PRIVATE ${SWIG_MODULE_pythonlm_REAL_NAME})
     
     #target_include_directories(lm_python PRIVATE ${SWIG_MODULE_pythonlm_REAL_NAME} ${PYTHON_INCLUDE_DIRS})
     target_include_directories(lm_python PRIVATE 
          ${SWIG_MODULE_pythonlm_REAL_NAME} 
          ${PYTHON_INCLUDE_DIRS}
          ${CMAKE_SOURCE_DIR}            # プロジェクトのトップディレクトリを追加
          ${CMAKE_SOURCE_DIR}/core        # 必要に応じて "core" ディレクトリも追加
          ${CMAKE_BINARY_DIR}             # 生成されたファイルがあるビルドディレクトリを追加
     )
     # # もし既にファイルが生成されているなら（または後で生成されることを保証する必要があります）
     # add_library(PythonLM SHARED IMPORTED)
     # set_target_properties(PythonLM PROPERTIES
     # IMPORTED_LOCATION "${CMAKE_BINARY_DIR}/_lm.cpython-37m-x86_64-linux-gnu.so"
     # )
     # インポートライブラリターゲットの定義
     add_library(PYTHONLM SHARED IMPORTED)
     set_target_properties(PYTHONLM PROPERTIES
     IMPORTED_LOCATION "${CMAKE_BINARY_DIR}/_lm.cpython-37m-x86_64-linux-gnu.so"
     )
     target_link_libraries(lm_python PRIVATE 
          # _pythonlm
          PYTHONLM
          lmcore
          lmmain
          # ${SWIG_MODULE_pythonlm_REAL_NAME}
          # ${CMAKE_BINARY_DIR}             # 生成されたファイルがあるビルドディレクトリを追加
          ${PYTHON_LIBRARIES}
          ${Protobuf_LIBRARIES}
          ${HDF5_LIBRARIES}
     )
     # 代わりに、必要があればビルド順序を保証
     add_dependencies(lm_python _pythonlm)

     # target_link_libraries(lm_python PRIVATE ${PYTHON_LIBRARIES})
   

     install(TARGETS lm_python RUNTIME DESTINATION bin)
endif ()
