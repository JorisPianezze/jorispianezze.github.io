.. _compilation:

Compilation
*********************************************************

First you need to compile :ref:`compilation_NETCDF` and :ref:`compilation_OASIS` libraries because :ref:`compilation_MESONH`, :ref:`compilation_CROCO` and :ref:`compilation_WW3` will use these libraries during their compilation.

.. note::

   For :ref:`compilation_CROCO`'s compilation you need to know the numbers of horizontal and vertical points of your domain before compile it.

.. _compilation_NETCDF:

NetCDF
====================================

You can compile NetCDF library using the script :

.. code-block:: bash

   cd models_YOURCASESTUDY/libraries/
   sbatch install_aer_hdf_netcdf_libraries.sub

To test compilation of the different libraries, do :

.. code-block:: bash

   ./build_netcdf-X.X.X/bin/nf-config --all

And you need to see :

.. code-block:: bash

   This netCDF-Fortran X.X.X has been built with the following features:

     --cc        -> XXX
     --cflags    -> XXX

     --fc        -> XXX
     --fflags    -> XXX
     --flibs     -> XXX
     --has-f90   -> XXX
     --has-f03   -> XXX

     --has-nc2   -> XXX
     --has-nc4   -> XXX

     --prefix    -> XXX
     --includedir-> XXX
     --version   -> netCDF-Fortran X.X.X

.. _compilation_OASIS:

OASIS
====================================

To compile OASIS you need to adapt the make.inc and make.${machine} path. 

.. warning:: 

   * You need to put absolute path !
   * make.inc call make.${machine} file
   * To know environment variable ${machine} look at your environment.sh file

Then you can compile OASIS :

.. code-block:: bash
  
   cd models_YOURCASESTUDY/
   source environment.sh
   cd oasis3-mct_5.2/util/make_dir/
   make realclean -f TopMakefileOasis3
   make -f TopMakefileOasis3

.. _compilation_MESONH:

Meso-NH
====================================

Before compiling Meso-NH, you need to change MNHENV in the :file:`MNH/src/configure` file by the path of your environment.sh file :

.. code-block:: bash

   export MNHENV=${MNHENV:-"
          source /XXX/models_YOURCASESTUDY/environment.sh
   "}

Then to compile Meso-NH's package, do :

.. code-block:: bash

   cd models_YOURCASESTUDY/MNH-V5-7-2/src/
   
   export VER_MPI=MPIAUTO
   export VER_CDF=CDFPERSO
   export VER_OASIS=OASISPERSO

   ./configure
   
and then, depending of the architecture, compile interactively :

.. code-block:: bash

   . ../conf/profile_mesonh
   make   
   make installmaster

or submit a job (take care of the name of profile_mesonh in the job). By example, for belenos, do :

.. code-block:: bash

   sbatch job_make_mesonh_BullX_belenos
  
.. note::
   
   * By setting VER_CDF=CDFPERSO and VER_OASIS=OASISPERSO, compilation of Meso-NH will use NETCDF_CONFIG must point to the nf-config file. OASISDIR must point to the OASIS build directory. NETCDF_CONFIG and OASISDIR environment variables are defined in environment.sh file.

   * If you chose VER_CDF=CDFAUTO and VER_OASIS=OASISAUTO, you will install and compile NetCDF and OASIS libraries provided in the Meso-NH's package. These libraries will be located in src/LIB directory.
   
   * The :command:`make` command calls different files: Makefile (compile libraries), Rules.\$ARCH\$(F).mk (options of compilation), Makefile.MESONH.mk (link to the compiled libraries)
   
Many programs have been compiled. In this documentation, we will focus on those listed in the following table. These programs can be utilized with or without activating OASIS coupling. The selection of Meso-NH's parameterizations is managed through the use of namelists; therefore, it is not necessary to recompile Meso-NH if you wish to change parameterizations, unlike WW3 and CROCO.

.. table:: List of Meso-NH's executables used in this documentation. IC corresponds to Initial Condition and LBC to Lateral Boundary Condition.

   +--------------------+--------------------------------------+-----------------+
   | Executable name    + Function                             + Namelist        |
   +====================+======================================+=================+
   | PREP_PGD           | Prepare horizontal grid              | PRE_PGD1.nam    |
   +--------------------+--------------------------------------+-----------------+
   | PREP_REAL_CASE     | Prepare verticale grid and IC / LBC  | PRE_REAL1.nam   |
   +--------------------+--------------------------------------+-----------------+
   | MESONH             | Run simulation                       | EXSEG1.nam      |
   +--------------------+--------------------------------------+-----------------+
   | DIAG               | Compute diagnostics after simulation | DIAG1.nam       |
   +--------------------+--------------------------------------+-----------------+    
   

.. _compilation_CROCO:

CROCO
====================================

Whatever the computers, you can follow the same procedure to compile CROCO.

First, you need to go to the directory :

.. code-block::

   cd models_CASESTUDY/croco-v2.1.0/exe_${machine}

.. note::

   * To know environment variable ${machine} look at your environment.sh file

In this directoy, there are :

* clean.sh: a script to clean the compiled directory

* jobcomp : a script to launch the compilation adapted to your machine.

* MY_SRC : a folder containing the configuration files cppdefs.h and param.h files.

In the cppdefs.h file, you can choose whether or not to pair with the following cpp keys: 

.. code-block:: fortran

   # define OA_COUPLING
   # define OW_COUPLING

The cpp keys OA_COUPLING and OW_COUPLING are used to activate coupling with an atmospheric model and a wave model, respectively. Next you need to give a name to your configuration, which will be used in the param.h file:

.. code-block:: fortran

   /* Configuration Name */
   # define YOURCASESTUDY

In param.h, you also must indicate the number of points you have for the CROCO domain.

.. tip:: To find out these points, run a ncdump -h command on the file rstrt_SAVE.nc for example, see the section :ref:`preprocessing_MNH`.

.. code-block:: fortran

   # elif defined  YOURCASESTUDY
         parameter (LLm0=38,  MMm0=38,  N=32)     ! YOURCASESTUDY

The number of cores used (NNODES) during the simulation must be entered in the param.h file and the code must be recompiled each time the number of cores used is changed:

.. code-block:: fortran

   #ifdef MPI
   integer NP_XI, NP_ETA, NNODES
   parameter (NP_XI=1, NP_ETA=1,  NNODES=NP_XI*NP_ETA)
   parameter (NPP=1)
   parameter (NSUB_X=1, NSUB_E=1)

NP_XI and NP_ETA correspond to the number of cores in longitude and latitude respectively.

Unlike WW3 and Meso-NH, there is no program for preparing initial conditions or lateral boundary conditions: these steps are carried out using croco_pytools.

To compile CROCO run the following command:

.. code-block:: bash

   source ../../environment.sh
   ./jobcomp

.. note:: Before compiling you need to check and eventually change SOURCE=../OCEAN directory in the beginning of jobcomp script.

Only one compiled program is used in this documentation (cf following table).
This executable depends on the settings activated, so if you want to change the settings you have to recompile CROCO: it is therefore necessary to copy the exe compilation folder so as not to overwrite the previously compiled executables.
In the croco folder, I therefore have two folders exe_YOURCCONFIG_Xcore_CPLOA and exe_YOURCONFIG_Xcore_CPLOA_CPLOW containing the executables for the coupled ocean-atmosphere and ocean-wave-atmosphere simulations, respectively.

.. table:: Executable CROCO compiled and used in this documentation.

   +--------------------+--------------------------------------+-----------------+
   | Executable name    + Function                             + Namelist        |
   +====================+======================================+=================+
   | croco              | Run simulation                       | croco.in        |
   +--------------------+--------------------------------------+-----------------+
   
.. _compilation_WW3:

WW3
====================================

The steps for compiling WW3 are detailed in the WW3 manual (ww3_dir/manual/), but they are also outlined here for enhanced clarity.
Regardless of the computer you are using, you can follow the same procedure to compile WW3. Next, you need to prepare the link and comp files required for compiling WW3 by using the following command:

.. code-block:: bash

   cd models_YOURCASESTUDY
   source environment.sh
   cd WW3-v7.14/model/bin/
   ./w3_setup .. -c ${machine} -s OASACM -q

At this stage, the :file:`comp` and :file:`link` files are created and the :file:`switch_OASACM` is copied into the :file:`switch` file.
Note that it is in the switch file that you must define the numerical schema, physical parameters, ... you want to use. The OASACM and OASOCM switches are used to activate coupling with an atmospheric model and an ocean model respectively (see WW3 manual).

For the aim of this documentation you need to create two new swicthes switch_OASACM_OASOCM and switch_NOOASIS by example. The first one will have an additional key OASOCM and the second one will not have COU OASIS OASACM keys. switch_NOOASIS will be used to create executables for WW3 spinup simulation.

Next, you can choose to compile following WW3 programs with the w3_make command:

.. code-block:: bash
 
   ./w3_make ww3_grid ww3_bound ww3_bounc ww3_prnc ww3_shel ww3_ounf

To recompile everything you need to do :

.. code-block:: bash

   ./w3_clean
   ./w3_new
   ./w3_make

A list of programs have been compiled, and for this documentation, we will use those listed in the following table.

.. table:: List of executables WW3 compiled and used in this documentation. IC corresponds to Initial Condition and LBC to Lateral Boundary Condition.

   +--------------------+--------------------------------------+-----------------+
   | Executable name    + Function                             + Namelist        |
   +====================+======================================+=================+
   | ww3_grid           | Prepare grid                         | ww3_grid.nml    |
   +--------------------+--------------------------------------+-----------------+
   | ww3_bound          | Prepare LBC (read txt files)         | ww3_bound.nml   |
   | ww3_bounc          | Prepare LBC (read nc files)          | ww3_bounc.nml   |
   +--------------------+--------------------------------------+-----------------+
   | ww3_prnc           | Prepare forcing (wind in our case)   | ww3_prnc.nml    |
   +--------------------+--------------------------------------+-----------------+
   | ww3_shel           | Run simulation                       | ww3_shel.nml    |
   +--------------------+--------------------------------------+-----------------+    
   | ww3_ounf           | Create netcdf file                   | ww3_ounf.nml    |
   +--------------------+--------------------------------------+-----------------+

.. note::

   * These executables, located in the exe folder, depend on the activated settings, so any changes to the settings require recompiling WW3. Therefore, it is essential to duplicate the exe folder to avoid overwriting previously compiled executables.

   * For this tutorial, you need to compile WW3 twice: once with the OASACM_OASOCM switch and once with the NOOASIS switch (necessary only for the spinup simulation).

   * In WW3/models/, you need to have two exe directories: one named exe_NOOASIS and the other exe_OASACM_OASOCM. These contain the executables for the spinup simulation and the coupled ocean-wave-atmosphere simulation, respectively :

   .. code-block:: bash

      ./w3_setup .. -c ${machine} -s NOOASIS -q
      ...
      ./w3_setup .. -c ${machine} -s OASACM_OASOCM -q


   * To check that WW3 has been correctly compiled, a number of tests are available in the WW3/regtests/ folder.

