
if(Matlab_FOUND)
    if(BUILD_TESTING)
        add_subdirectory(tests)
    endif()
endif()

# Install.
# --------
install(DIRECTORY examples/ DESTINATION "${OPENSIM_INSTALL_MATLABEXDIR}/Examples")
install(DIRECTORY Hopper_Device DESTINATION "${OPENSIM_INSTALL_MATLABEXDIR}")
install(DIRECTORY Utilities DESTINATION "${OPENSIM_INSTALL_MATLABEXDIR}")
install(DIRECTORY OpenSenseExample DESTINATION "${OPENSIM_INSTALL_MATLABEXDIR}")
install(DIRECTORY Dynamic_Walking_Tutorials/Dynamic_Walker_Builder
        DESTINATION "${OPENSIM_INSTALL_MATLABEXDIR}")
install(DIRECTORY Dynamic_Walking_Tutorials/Dynamic_Walker_Challenge
        DESTINATION "${OPENSIM_INSTALL_MATLABEXDIR}"
        # These files have not yet been updated for 4.0:
        PATTERN "DesignMainStarterWithControls.m" EXCLUDE
        PATTERN "AddCoordinateActuator.m" EXCLUDE
        PATTERN "AddMillardMuscle.m" EXCLUDE
        PATTERN "AddPathActuator.m" EXCLUDE
        PATTERN "IntegrateOpenSimPlant.m" EXCLUDE
        PATTERN "Main_WalkerForwardSim.m" EXCLUDE
        PATTERN "Main_WalkerForwardSimWithControls.m" EXCLUDE
        PATTERN "OpenSimPlantControlsFunction.m" EXCLUDE
        PATTERN "OpenSimPlantFunction.m" EXCLUDE
        PATTERN "Simulation_using_Matlab_Integration.m" EXCLUDE)



# Remove parts from the hopper device example that the tutorial audience is
# expected to fill in.
# Anything matching `% ANSWER{ ... % }` is cut from the file.
foreach(fname RunHopper RunHopperWithDevice)
    set(file_contents "")
    file(STRINGS Hopper_Device/${fname}_answers.m file_contents_answers)
    list(LENGTH file_contents_answers num_lines)
    set(line_num 0)
    while(${line_num} LESS ${num_lines})
        list(GET file_contents_answers ${line_num} current_line)
        if("${current_line}" MATCHES "% ANSWER{")
            while(NOT "${current_line}" MATCHES "% }")
                math(EXPR line_num "${line_num} + 1")
                list(GET file_contents_answers ${line_num} current_line)
            endwhile()
            # Add an empty line in place of the ANSWERS block.
            set(file_contents "${file_contents}\n")
        else()
            set(file_contents "${file_contents}${current_line}\n")
        endif()
        math(EXPR line_num "${line_num} + 1")
    endwhile()
    file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/Hopper_Device/${fname}.m"
               "${file_contents}")
    install(FILES       "${CMAKE_CURRENT_BINARY_DIR}/Hopper_Device/${fname}.m"
            DESTINATION "${OPENSIM_INSTALL_MATLABEXDIR}/Hopper_Device")
endforeach()

# The configureOpenSim.m script contains paths into the OpenSim installation
# that may be different on different platforms, so we configure it with CMake
# variables.
file(RELATIVE_PATH matlabexdir_to_install_dir
    "${CMAKE_INSTALL_PREFIX}/${OPENSIM_INSTALL_MATLABEXDIR}"
    "${CMAKE_INSTALL_PREFIX}")
configure_file(configureOpenSim.m.in
    "${CMAKE_CURRENT_BINARY_DIR}/configureOpenSim.m" @ONLY)
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/configureOpenSim.m"
        DESTINATION "${OPENSIM_INSTALL_MATLABEXDIR}")
