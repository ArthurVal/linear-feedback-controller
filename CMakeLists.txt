cmake_minimum_required(VERSION 3.22.1)

#
# Project definition
#
project(linear_feedback_controller LANGUAGES CXX)

#
# Options
#
option(BUILD_SHARED_LIBS "Build using shared libraries" ON)

# Add global mandatory warnings
# (from https://github.com/cpp-best-practices/cppbestpractices/blob/master/02-Use_the_Tools_Available.md)
if(CMAKE_CXX_COMPILER_ID MATCHES "(GNU|Clang)")
  add_compile_options(
    # -Werror                     # Consider warnings as errors

    -Wall -Wextra               # All reasonable warnings
    -pedantic                   # NON ISO C/C++ warnings
    -Wshadow                    # Warns on shadowing
    -Wnon-virtual-dtor          # Warns on virtual classes without virtual dtor

    # -Wold-style-cast            # Warns for c-style casts
    # -Wcast-align                # Warns for potential performance problem casts
    # -Wunused                    # Warns on anything being unused
    # -Woverloaded-virtual        # Warns if you overload (not override) a
    #                             # virtual function
    # -Wconversion                # Warns on type conversions that may lose data
    # -Wsign-conversion           # (Clang all versions, GCC >= 4.3) Warns on sign
    #                             # conversions
    # -Wmisleading-indentation    # (only in GCC >= 6.0) Warns if indentation
    #                             # implies blocks where blocks do not exist
    # -Wduplicated-cond           # (only in GCC >= 6.0) Warns if if / else chain
    #                             # has duplicated conditions
    # -Wduplicated-branches       # (only in GCC >= 7.0) warn if if / else
    #                             # branches have duplicated code
    # -Wlogical-op                # (only in GCC) warn about logical operations
    #                             # being used where bitwise were probably wanted
    # -Wnull-dereference          # (only in GCC >= 6.0) warn if a null
    #                             # dereference is detected
    # -Wuseless-cast              # (only in GCC >= 4.8) warn if you perform a
    #                             # cast to the same type
    # -Wdouble-promotion          # (GCC >= 4.6, Clang >= 3.8) warn if float is
    #                             #  implicitly promoted to double
    # -Wformat=2                  # Warns on security issues around functions that
    #                             # format output (i.e., printf)
    # -Wlifetime                  # (only special branch of Clang currently) shows
    #                             # object lifetime issues
    # -Wimplicit-fallthrough      # Warns when case statements fall-through.
    #                             # (Included with -Wextra in GCC, not in clang)
  )

elseif(CMAKE_CXX_COMPILER_ID EQUALS "MSVC")
  add_compile_options(
    # /WX     # Consider warnings as errors

    /W4     # All reasonable warnings
    /W14640 # Enable warning on thread unsafe static member initialization

    # /w14242 # 'identfier': conversion from 'type1' to 'type1', possible loss
    #         # of data
    # /w14254 # 'operator': conversion from 'type1:field_bits' to
    #         # 'type2:field_bits', possible loss of data
    # /w14263 # 'function': member function does not override any base class
    #         # virtual member function
    # /w14265 # 'classname': class has virtual functions, but destructor is not
    #         # virtual instances of this class may not be destructed correctly
    # /w14287 # 'operator': unsigned/negative constant mismatch
    # /we4289 # nonstandard extension used: 'variable': loop control variable
    #         # declared in the for-loop is used outside the for-loop scope
    # /w14296 # 'operator': expression is always 'boolean_value'
    # /w14311 # 'variable': pointer truncation from 'type1' to 'type2'
    # /w14545 # expression before comma evaluates to a function which is missing
    #         # an argument list
    # /w14546 # function call before comma missing argument list
    # /w14547 # 'operator': operator before comma has no effect; expected operator
    #         # with side-effect
    # /w14549 # 'operator': operator before comma has no effect; did you intend
    #         # 'operator'?
    # /w14555 # expression has no effect; expected expression with side-effect
    # /w14619 # pragma warning: there is no warning number 'number'
    # /w14826 # Conversion from 'type1' to 'type_2' is sign-extended. This may
    #         # cause unexpected runtime behavior.
    # /w14905 # wide string literal cast to 'LPSTR'
    # /w14906 # string literal cast to 'LPWSTR'
    # /w14928 # illegal copy-initialization; more than one user-defined conversion
    #         # has been implicitly applied
  )

else()
  message(WARNING
    "Couldn't set mandatory warnings: "
    "Unknown CXX_COMPILER_ID \"${CMAKE_CXX_COMPILER_ID}\""
  )

endif()

#
# Handle dependencies by reading the package.xml
#
find_package(ament_cmake_auto REQUIRED)
ament_auto_find_build_dependencies()

#
# Generate the ROS2 parameters interface automatically.
#
include(cmake/sec_generate_parameter_library.cmake)
# Main node params
sec_generate_parameter_library_markdown(${PROJECT_NAME}_parameters_doc
                                        src/linear_feedback_controller.yaml)
sec_generate_parameter_library(
  generated_parameters # Lib name
  ${PROJECT_NAME}_parameters # CMake target name for the parameter library.
  src/linear_feedback_controller.yaml # Path to input yaml file
)
add_dependencies(${PROJECT_NAME}_parameters ${PROJECT_NAME}_parameters_doc)
# Joint estimator params
sec_generate_parameter_library_markdown(joint_state_estimator_parameters_doc
                                        src/linear_feedback_controller.yaml)
sec_generate_parameter_library(
  joint_state_estimator_generated_parameters # Lib name
  joint_state_estimator_parameters # CMake target name for the parameter
                                   # library.
  src/joint_state_estimator.yaml # Path to input yaml file
)
add_dependencies(joint_state_estimator_parameters
                 joint_state_estimator_parameters_doc)

#
# Main Library
#
set(${PROJECT_NAME}_headers
    include/${PROJECT_NAME}/averaging_filter.hpp
    include/${PROJECT_NAME}/averaging_filter.hxx
    include/${PROJECT_NAME}/joint_state_estimator.hpp
    include/${PROJECT_NAME}/lf_controller.hpp
    include/${PROJECT_NAME}/linear_feedback_controller.hpp
    include/${PROJECT_NAME}/linear_feedback_controller_ros.hpp
    include/${PROJECT_NAME}/min_jerk.hpp
    include/${PROJECT_NAME}/pd_controller.hpp
    include/${PROJECT_NAME}/robot_model_builder.hpp)
set(${PROJECT_NAME}_sources
    src/joint_state_estimator.cpp #
    src/lf_controller.cpp #
    src/linear_feedback_controller.cpp #
    src/linear_feedback_controller_ros.cpp #
    src/min_jerk.cpp #
    src/pd_controller.cpp #
    src/robot_model_builder.cpp)
ament_auto_add_library(${PROJECT_NAME} ${${PROJECT_NAME}_sources}
                       ${${PROJECT_NAME}_headers})
ament_target_dependencies(${PROJECT_NAME} Eigen3)
target_link_libraries(${PROJECT_NAME} pinocchio::pinocchio)
target_link_libraries(${PROJECT_NAME} ${PROJECT_NAME}_parameters)

#
# Unit tests
#
include(CTest)
if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_auto_find_test_dependencies()

  ament_auto_add_gtest(test_averaging_filter tests/test_averaging_filter.cpp)
  target_link_libraries(test_averaging_filter ${PROJECT_NAME})

  ament_auto_add_gtest(test_min_jerk tests/test_min_jerk.cpp)
  target_link_libraries(test_min_jerk ${PROJECT_NAME})

  ament_auto_add_gtest(test_robot_model_builder
                       tests/test_robot_model_builder.cpp)
  target_link_libraries(test_robot_model_builder ${PROJECT_NAME}
                        example-robot-data::example-robot-data)
endif()

#
# Export plugins
#
pluginlib_export_plugin_description_file(controller_interface
                                         controller_plugins.xml)

#
# Installation
#
install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/doc
        DESTINATION share/${PROJECT_NAME})
install(
  PROGRAMS tests/pd_plus_controller.py
  DESTINATION lib/${PROJECT_NAME}
  RENAME pd_plus_controller)
install(DIRECTORY config DESTINATION share/${PROJECT_NAME})
install(DIRECTORY launch DESTINATION share/${PROJECT_NAME})
ament_export_libraries(${PROJECT_NAME})
ament_auto_package()
