set( HEADERS
	BoundaryCondition.h
	PhysicalConstant.h
	Constrained.h
	conversion_rate.h
	DistributionInfo.h
	DUMUX.h
	Eclipse.h
	eos.h
	fem_ele.h
	fem_ele_std.h
	fem_ele_vec.h
	FEMCondition.h
	FEMEnums.h
	femlib.h
	files0.h
	GeoInfo.h
	InitialCondition.h
	LinearFunctionData.h
	mathlib.h
	matrix_class.h
	Output.h
	pcs_dm.h
	problem.h
	ProcessInfo.h
	prototyp.h
	rf_bc_new.h
	rf_fct.h
	rf_fluid_momentum.h
	rf_ic_new.h
	rf_kinreact.h
	rf_mfp_new.h
	rf_mmp_new.h
	rf_msp_new.h
	rf_node.h
	rf_num_new.h
	rf_out_new.h
	rf_pcs.h
	rf_random_walk.h
	rf_react.h
	rf_react_int.h
	rf_st_new.h
	rf_tim_new.h
	rfmat_cp.h
	solver.h
	SourceTerm.h
	SparseMatrixDOK.h
	Stiff_Bulirsch-Stoer.h
	tools.h
	vtk.h
)

set( SOURCES
	BoundaryCondition.cpp
	CAP_IO.cpp
	conversion_rate.cpp
	DistributionInfo.cpp
	DUMUX.cpp
	Eclipse.cpp
	eos.cpp
	fem_ele.cpp
	fem_ele_std.cpp
	fem_ele_std1.cpp
	fem_ele_std_tes.cpp
	fem_ele_std_tneq.cpp
	fem_ele_vec.cpp
	FEMCondition.cpp
	FEMEnums.cpp
	femlib.cpp
	files0.cpp
	GeoInfo.cpp
	InitialCondition.cpp
	LinearFunctionData.cpp
	mathlib.cpp
	matrix_class.cpp
	Output.cpp
	pcs_dm.cpp
	problem.cpp
	ProcessInfo.cpp
	rf_bc_new.cpp
	rf_fct.cpp
	rf_fluid_momentum.cpp
	rf_ic_new.cpp
	rf_kinreact.cpp
	rf_mfp_new.cpp
	rf_mmp_new.cpp
	rf_msp_new.cpp
	rf_node.cpp
	rf_num_new.cpp
	rf_out_new.cpp
	rf_pcs.cpp
	rf_random_walk.cpp
	rf_react.cpp
	rf_react_cap.cpp
	rf_react_int.cpp
	rf_st_new.cpp
	rf_tim_new.cpp
	rfmat_cp.cpp
	SourceTerm.cpp
	SparseMatrixDOK.cpp
	Stiff_Bulirsch-Stoer.cpp
	tools.cpp
	vtk.cpp
)

if(OGS_FEM_PETSC OR OGS_FEM_PETSC_GEMS)
	set( SOURCES ${SOURCES} rf_pcs1.cpp fct_mpi.h fct_mpi.cpp)
else()
	set( SOURCES ${SOURCES} par_ddc.h par_ddc.cpp)
	## New sparse class. WW
	if(OGS_USE_NEW_EQS)
		set( SOURCES ${SOURCES} equation_class.h equation_class.cpp )
		if(OGS_FEM_JFNK)
			set( SOURCES ${SOURCES} rf_pcs1.cpp )
		endif()
	else()
		set( SOURCES ${SOURCES} solver.h matrix_routines.h solver.cpp matrix_routines.cpp)
	endif()
endif()

if(OGS_FEM_BRNS)
	set( SOURCES ${SOURCES} rf_REACT_BRNS.h rf_REACT_BRNS.cpp )
endif()

if(OGS_FEM_GEMS OR OGS_FEM_PETSC_GEMS)
	set( SOURCES ${SOURCES} rf_REACT_GEM.h rf_REACT_GEM.cpp )
endif()


include_directories(
	${CMAKE_SOURCE_DIR}/Base
	${CMAKE_SOURCE_DIR}/FEM
	${CMAKE_SOURCE_DIR}/FileIO
	${CMAKE_SOURCE_DIR}/FileIO/FEMIO
	${CMAKE_SOURCE_DIR}/FileIO/MeshIO
	${CMAKE_SOURCE_DIR}/GCC
	${CMAKE_SOURCE_DIR}/GEM
	${CMAKE_SOURCE_DIR}/GEO
	${CMAKE_SOURCE_DIR}/MathLib
	${CMAKE_SOURCE_DIR}/MSH
)

if (LIS)
	include_directories(${LIS_INCLUDE_DIR})
endif ()

if (MKL)
	include_directories(${MKL_INCLUDES})
endif ()

if(OGS_FEM_PQC)
	include_directories(${CMAKE_SOURCE_DIR}/ThirdParty/PQC)
endif ()

if(OGS_FEM_IPQC)
	include_directories(${CMAKE_SOURCE_DIR}/ThirdParty/IPQC/src)
endif ()

# Create the library
add_library( FEM STATIC ${HEADERS} ${SOURCES} )

target_link_libraries(FEM
	Base
	FileIO
	GCC
	GEO
	MSH
)

if(OGS_FEM_PQC)
	target_link_libraries(FEM PQC)
endif()

if(PARALLEL_USE_MPI)
	target_link_libraries(FEM ${MPI_CXX_LIBRARIES})
endif()

if(OGS_FEM_MKL)
	target_link_libraries( FEM ${MKL_LIBRARIES} )
endif()

if (WIN32)
	if (OGS_FEM_GEMS)
		target_link_libraries( FEM GEM )
	endif ()
	if (LIS)
		target_link_libraries( FEM ${LIS_LIBRARIES} )
		set_target_properties( FEM PROPERTIES LINK_FLAGS "/NODEFAULTLIB:libcmt" )
	endif ()
	if (OGS_FEM_BRNS)
		target_link_libraries( FEM BrnsDll )
	endif ()
	if (OGS_FEM_CHEMAPP)
		target_link_libraries( FEM ca_vc_e )
	endif ()
endif ()

if (UNIX)
	if (OGS_FEM_BRNS)
		target_link_libraries( FEM ${CMAKE_DL_LIBS} ${LAPACK_LIBRARIES} )
	endif ()

	if (LIS)
		target_link_libraries( FEM ${LIS_LIBRARIES} )
	endif ()
endif ()

if (OGS_FEM_CHEMAPP)
	include_directories ( ${CMAKE_SOURCE_DIR}/EQL )
	target_link_libraries ( FEM EQL )
endif ()


if (OGS_FEM_CAP)
	include_directories (  ${CMAKE_SOURCE_DIR}/CMP )
	target_link_libraries (
		FEM
		GEO
		MSH
		CMP
	)
	if (UNIX)
		target_link_libraries (
			FEM
			ChemAppCS_64
			EChemAppS_64
		)
	endif ()
endif ()

if(OGS_USE_CVODE)
	target_link_libraries(FEM
		sundials_cvode_static
		sundials_nvecserial_static
	)
endif()
