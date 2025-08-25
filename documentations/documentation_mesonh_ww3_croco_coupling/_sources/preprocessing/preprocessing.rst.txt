.. _pre-processing:

Pre-processing
*********************************************************

During the pre-processing steps, you will choose the horizontal and vertical grids and prepare the initial and boundary conditions for the different models.

.. note::

   I recommend performing the steps :ref:`preprocessing_MNH`, :ref:`preprocessing_WW3`, and :ref:`preprocessing_OASIS` on the machine where you will run the simulation
   Depending on your machine and external access, I recommend doing the step :ref:`preprocessing_CROCO` remotely and transferring the files.

.. _python_requirements:

Requirements
====================================

This documentation has been tested with following python's environment :

.. code-block:: yaml

   name: environment
   channels:
     - conda-forge
   dependencies:
     - python=3.9
     - setuptools>60,<=74
     - numpy=1.23.5
     - netcdf4=1.6.2
     - matplotlib=3.7.2
     - cartopy=0.21.0
     - scipy=1.9.3
     - wxpython=4.2.0
     - traits=6.4.1
     - traitsui=7.4.2
     - dask=2022.11.1
     - geopandas=0.12.1
     - regionmask=0.9.0
     - pyinterp=2022.10.1
     - gcc>=8.3.0
     - gfortran>=8.3.0
     - netcdf-fortran=4.6.0
     - cdsapi
     - pyqt=5.15.9
     - copernicusmarine=2.0.1
     - xarray=2024.3.0

.. note::

   You can copy/paste these lines in a file called :file:`environment.yml` or find this file in following git repository.

To install it with :command:`conda`, do

.. code-block:: bash
   
   mamba env create -f environment.yml

.. note::

   Insallation of python's environment has to be done one time.

To activate it, do

.. code-block:: bash

   mamba activate environment
   
To deactivate it, do

.. code-block:: bash

   mamba deactivate

.. _get_config_template:

Get config template
====================================

All the files used in this section are available on a public Git repository. To download it, do :

.. code-block:: bash

   git clone https://github.com/JorisPianezze/config_tmpl_for_mesonh_ww3_croco_coupling.git

To be consistent with models directory :ref:`installation`, I recommend to rename this directory as :

.. code-block:: bash

   mv config_tmpl_for_mesonh_ww3_croco_coupling config_YOURCASESTUDY

.. note::

   Depending on the machine you are using, you may have to put this directory on the work directory.

A lot of files are needed for a coupled ocean-wave-atmosphere simulation, so I recommend organising your folders as following the structure of the config_YOURCASESTUDY directory :

.. code-block:: bash

   1_input_MNH          ! scripts and files for Meso-NH
   2_input_WW3          ! scripts and files for WW3
   3_input_CROCO        ! scripts and files for CROCO
   4_input_OASIS        ! scripts and files for OASIS
   A1_frc_MNH           ! example of a MNH forced simulation
   A2_frc_WW3_spinup    ! spinup simulation for WW3
   B1_cpl_MNH_CROCO     ! example of a MNH/CROCO coupled simulation
   C1_cpl_MNH_WW3_CROCO ! example of a MNH/WW3/CROCO coupled simulation
   README.md            ! readme

.. _preprocessing_MNH:

Meso-NH
====================================

In this section, we will use the scripts located in the 1_input_MNH folder of the Git repository (cf. :ref:`get_config_template` section).

.. note::

   The initial and boundary conditions for Meso-NH can be created where you want. On the machine on which you are running the coupled simulation or elsewhere.

To prepare the initial and boundary conditions for Meso-NH, it is necessary to use (at least), the following executables :

* PREP_PGD is used to prepare the physiographic data file: definition of the horizontal grid (number of points and grid resolution), type of coverage, etc ...

* PREP_REAL_CASE is used to prepare the initial and boundary conditions for the atmospheric fields (wind speed, potential temperature, mixing ratio, etc.).

The ocean-wave-atmosphere coupling only has an impact on the simulation (MESONH program). The initial and coupling files for Meso-NH are therefore obtained in the same way as for a forced simulation.

Prepare the horizontal grid : PREP_PGD
---------------------------------------

First you need to specify PREP_PGD_FILES directory. This directory contains the files needed to define the type of cover (ECOCLIMAP, SAND_HWSD_MOY, ...) and the orography (gtopo30).
If you don't have it you can use the script get_pgd_files to download them.

These files can also be downloaded on `Meso-NH <http://mesonh.aero.obs-mip.fr/mesonh56/Download>`_ website.

.. note:: For belenos users, these files are here /home/cnrm_other/ge/mrmh/rodierq/SAVE/mesonh

You can now define your horizontal grid in the namelist PRE_PGD1.nam. An example is located in the 1_input_mnh directory.
The number of points and the size of the grid are defined by the variables NIMAX, NJMAX, XDX and XDY in the namelist PRE_PGD1.nam.

Note that it is not possible to use the coupling with TEB, so it is essential not to activate it when creating the PGD and to transform the city points into land:

.. code-block:: fortran

   &NAM_PGD_SCHEMES CNATURE = 'ISBA',
                    CSEA    = 'SEAFLX',
                    CTOWN   = 'NONE',
                    CWATER  = 'WATFLX' /

   &NAM_PGD_ARRANGE_COVER LTOWN_TO_ROCK=.TRUE. /

.. warning:: Test with TEB, it has to works in MNH570 !

Add bathymetry to create WW3 grid :
.. code-block:: fortran

   &NAM_SEABATHY YSEABATHY         = 'etopo2.nc',
                 YSEABATHYFILETYPE = 'NETCDF',
                 YNCVARNAME        = 'topo' /

To launch the PREP_PGD executable, you need to adapt the run_prep_pgd.sh script and run it :

.. code-block:: bash

   sbatch run_prep_pgd.sh

or 

.. code-block:: bash

   ./run_prep_pgd.sh

At the end of execution you should read in the OUTPUT_LISTING0 file: ''PREP_PGD ends successfully''.
At the end of this stage you will find two files in your directory: netcdf file with the fields corresponding to your domain and a descriptive file containing the namelists used (in some cases this file is empty).

Prepare the vertical grid, IC and LBC : PREP_REAL_CASE
-------------------------------------------------------------------------------------------

To prepare the initial and boundary conditions, you first need to download the large-scale forcing fields you will be using: AROME, ARPEGE, IFS or GFS.
An example of a ERA5 request via CDSAPI can be found in the 1_input_mnh folder (extract_era5_instantaneous_fields_for_mesonh_and_abl1d.py).

To extract ERA5 fields for Meso-NH, do :

.. code-block:: python

   python extract_era5_instantaneous_fields_for_mesonh_and_abl1d.py

Once the forcing fields have been extracted, you need to modify the namelist PRE_REAL1.nam_tmpl and, in particular, enter the vertical grid you want.
You then need to adapt the paths in the run_prep_real_case.sh script to define your environment, then run it one of the following command (depending of your system) :

.. code-block:: bash

   sbatch run_prep_real_case.sh

or 

.. code-block:: bash

   ./run_prep_real_case.sh

If, at the end of the OUTPUT_LISTING0 file, you have ''PREP_REAL_CASE ends correctly'', then this stage has gone well.
At the end of this stage you will find two files: a netcdf file with the fields corresponding to the atmospheric fields interpolated on your grid and a descriptive file containing the namelists used (in some cases this file is empty).

For the purposes of ocean-wave-atmosphere coupling, it is necessary to create a rstrt_MNH.nc file which will allow the model to be started and restarted (:ref:`RST LAG_concept`).
To do this, you must put the name of the netcdf file corresponding to the initial time in the create_restart_file_from_PREP_REAL_CASE.py script, then run the python script :

.. code-block:: bash

   python create_restart_file_from_PREP_REAL_CASE.py

At the end of this step, the rstrt_MNH.nc file should have been created.

Preparing the namelist for the coupled simulation : EXSEG1.nam
----------------------------------------------------------------------------

The final step in preparing the Meso-NH configuration is to adapt the EXSEG1.nam namelist to a coupled simulation.
In the case of this documentation, you do not have to modify the namelists present in the 1_input_mnh folder. The namelists EXSEG1.nam_B1_cpl_mnh_croco and EXSEG1.nam_C1_cpl_mnh_ww3_croco are already adapted to a coupled ocean-atmosphere and ocean-waves-atmosphere simulation, respectively.

Only the namelists to be added in the event of coupling are presented here (the other namelists are identical to a forced simulation):

.. code-block:: fortran

   &NAM_OASIS LOASIS = .TRUE..,
              LOASIS_GRID = .TRUE.,
              CMODEL_NAME = 'mesonh' /
           
   &NAM_SFX_WAVE_CPL XTSTEP_CPL_WAVE = 600.0,
                     CWAVE_U10 = 'MNH__U10',
                     CWAVE_V10 = 'MNH__V10',
                     CWAVE_CHA = 'MNH__CHA',
                     CWAVE_UCU = 'MNH_WSSU',
                     CWAVE_VCU = 'MNH_WSSV',
                     CWAVE_HS = 'MNH___HS',
                     CWAVE_TP = 'MNH___TP' /

   &NAM_SFX_SEA_CPL XTSTEP_CPL_SEA = 600.0,
                    CSEA_FWSU = 'MNH_TAUX',
                    CSEA_FWSV = 'MNH_TAUY',
                    CSEA_HEAT = 'MNH_HEAT',
                    CSEA_SNET = 'MNH_SNET',
                    CSEA_WIND = ' ',
                    CSEA_FWSM = ' ',
                    CSEA_EVAP = ' ',
                    CSEA_RAIN = ' ',
                    CSEA_SNOW = ' ',
                    CSEA_WATF = 'MNH_EVPR',
                    CSEA_PRES = ' ',
                    CSEA_SST = 'MNH__SST',
                    CSEA_UCU = ' ',
                    CSEA_VCU = ' ' /

.. note:: See :ref:`RST details_MNH` section to have detailled information about available coupling fields.

The strings must be the same as those defined in the namcouple file (Section :ref:`preprocessing_OASIS`). If you do not wish to use the coupling with a wave model, you must set XTSTEP_CPL_WAVE=-1.0 or delete the namelist NAM_SFX_WAVE_CPL.

List of necessary files for MNH : 1_input_mnh
-----------------------------------------------------------------------

.. code-block:: bash

   1_input_mnh
      | -- PGD.des/PGD.nc         ! from PREP_PGD program
      | -- PRC.des/PRC.nc         ! from PREP_REAL_CASE program      
      | -- rstrt_MNH.nc           ! from python script
      | -- EXSEG1.nam             ! namelist for the coupled simulation

.. _preprocessing_CROCO:

CROCO
====================================

.. warning:: A new python preprocessing tool for CROCO is under development. I will update this section when the first release of this tool will be available.

In this section, we will use the scripts located in the 3_input_croco directory.

Download croco_pytools
-----------------------------------------------------------------------

To get croco_pytools, do:

.. code-block:: bash

   cd 3_input_croco
   git clone git@gitlab.inria.fr:croco-ocean/croco_pytools.git

Create the horizontal grid
-----------------------------------------------------------------------

To create the horizontal grid for CROCO, go to PREPRO directory. You need to compile and modify path in make_grid.py script.

.. code-block:: bash

   cd PREPRO
   make
   python make_grid.py

.. note:: I needed to change NETCDFLIB line in Makedefs by NETCDFLIB= -L/usr/lib/x86_64-linux-gnu -lnetcdff -lnetcdf

.. note:: You can get coastlines data here :

.. code-block:: bash

   wget https://www.ngdc.noaa.gov/mgg/shorelines/data/gshhs/latest/gshhg-shp-2.3.7.zip
   
.. note:: etopo2.nc can be downloaded here : https://www.ncei.noaa.gov/products/etopo-global-relief-model

Create the IC and LBC
-----------------------------------------------------------------------

To initialise and force CROCO to the lateral boundary conditions, we use the Mercator-Ocean data that can be downloaded from the CMEMS servers. For that you need to create an account on `CMEMS <http://marine.copernicus.eu/services-portfolio/access-to-products/>`_ servers. After creating your account you have to install motu client. If you are using conda, you can install it using the following commands:

.. code-block:: bash

   conda install motuclient
   conda activate env_motuclient

.. note:: A conda environment environment_motuclient.yml is available in the 3_input_croco directory to install motuclien in a conda container.

Then yo can download GLORYS by using this script (you need to put your username and password and chose the date to extract) :

.. code-block:: bash

   ./download_glorys.sh

At the end of this extraction, you will have a netcdf file with the fields corresponding to those chosen in the extraction script. If you use the script in this documentation directly, the extracted file is called raw_motu_mercator_Y2020M09.nc.

To prepare the IC you have to adapt the path in the make_ini.py script and do :

.. code-block:: bash

   python make_ini.py

At the end of this step, you will have a netcdf file called croco_ini.nc.

To prepare the LBC you have to adapt the path in the make_bry.py script and do :

.. code-block:: bash

   python make_bry.py

At the end of this step, you will have a netcdf file called croco_bry.nc.

Once the times have been checked, you need to transfer these files to belenos and go to 3_input_croco to run the create_restart_file_from_ini_CROCO_file.py script to create the rstrt_CROCO.nc file.

.. code-block:: bash

   python create_restart_file_from_ini_CROCO_file.py

Prepare the namelist for the coupled simulation : croco.in
-----------------------------------------------------------------------

No information about OASIS must be defined in the namelist croco.in (located in the 3_input_croco directory) : CROCO reads the namcouple file to access the corresponding information. To modify this namelist, please refer to the CROCO documentation.

List of necessary files for CROCO : 3_input_croco
-----------------------------------------------------------------------

.. code-block:: bash

   3_input_croco
      | -- croco_grd.nc
      | -- croco_ini.nc
      | -- croco_bry.nc  
      | -- rstrt_CROCO.nc
      | -- croco.in


.. _preprocessing_WW3:

WW3
====================================

In this section, we will use scripts located in the 2_input_ww3 folder.

Create the files containing the grid
----------------------------------------

In order to have equivalent grids (coverage + resolution), it is possible to create the WW3 grid directly from the Meso-NH grid.
This is the approach I have followed in this documentation. To do this, you need to adapt the path in the python script create_grid_for_ww3_from_pgd.py and then run it via : 

.. code-block:: bash
 
   cd grid/
   python create_grid_for_ww3_from_pgd.py

.. note:: Several files are generated in inp format.

Prepare the grid : ww3_grid
----------------------------------------

Once the files defining the grid for WW3 have been created, you need to use the ww3_grid program. This program uses the namelist ww3_grid.nml (you can find an example in 2_input_ww3).
In this namelist you must add the names of the files created previously, for example :

.. code-block:: fortran

   &CURV_NML
      CURV%NX               =  40
      CURV%NY               =  40
      CURV%XCOORD%SF        =  1.0
      CURV%XCOORD%OFF       =  0.0
      CURV%XCOORD%FILENAME  =  'grid/WW3_IROISE_DX5KM.lon.inp'
      CURV%YCOORD%SF        =  1.0
      CURV%YCOORD%OFF       =  0.0
      CURV%YCOORD%FILENAME  =  'grid/WW3_IROISE_DX5KM.lat.inp'
   /

.. note:: You also have to modify bathy, obstruction and mask section of the ww3_grid.nml namelist.

You can then modify the paths in the run_ww3_grid.sh script and run it (depending of your system) :

.. code-block:: bash

   sbatch run_ww3_grid.sh

or

.. code-block:: bash

   ./run_ww3_grid.sh

At the end of this stage, you should find ''End of program'' at the end of the ww3_grid.out file, with no errors.
This program has generated 3 files: mod_def.ww3, mask.ww3 and mapsta.ww3. These files will be used by the other programs; they correspond to the WW3 grid in binary format.
Depending on the wind-wave parameterisation you are using, a ST4TABUHF2.bin file may also be generated at this stage: this file is also required by the other programs.

Prpare the forcing (wind) for spinup simulation : ww3_prnc
------------------------------------------------------------

Unlike Meso-NH/SurfEx and CROCO, a spinup simulation is required to prepare the initial condition for WW3. To prepare this simulation, you need the wind speed at 10m to force the wave model.
To be consistent with the atmospheric fields used to initialise and force the Meso-NH lateral boundary condition, we will use the 10m wind speed from the ERA5.

To extract the 10m wind speed from the ECMWF servers, you can use the CDSAPI requests located in the 2_input_ww3/era5 directory and run it via :

.. code-block:: bash

   python extract_era5_instantaneous_fields_for_ww3.py

.. note:: Extracted ERA5 files also contains wave spectra. These spectra will be used in the next section to prepare LBC for WW3.

Once the grib files have been extracted, you need to use the grib_to_netcdf_uv10.sh script located in the 2_input_ww3/wind directory to convert the grib files into a netcdf file in a WW3 readable format.
This script uses functions contained in the eccodes library. You may need to install it. To run it, simply do :

.. code-block:: bash

   ./grib_to_netcdf_uv10.sh

At the end of these steps, you will have a netcdf file wind.nc containing the 10 m wind speed fields for the period you are interested in.

Next, you can run the script run_ww3_prnc.sh to prepare the forcing file for WW3 using :

.. code-block:: bash

   sbatch run_ww3_prnc.sh 

or :

.. code-block:: bash

   ./run_ww3_prnc.sh 

At the end of this step, the wind.ww3 file is created. This file has to be used in the spinup simulation.

Prepare the LBC : ww3_bound
-----------------------------------------------------------------------

.. warning:: To be update with ERA5.

To force the WW3 simulations at the LBC for the spinup and the coupled simulations, we need the wave energy spectra.
These spectra can be found in the CDSAPI requests located in the era5 directory.
Once the ECMWF wave energy spectra have been extracted, you need to convert the grib files into spec files that can be read by ww3 programs.
The scripts needed for this conversion are located here: 2_input_ww3/spec.

First of all, you need to fill in the list_points file with the latitude and longitude of the points you need for your lateral boundary conditions.
Depending on the size and location of your domain, I recommend that you use at least 1 degree spacing between points. You should not have duplicate points and the first digit should be 000000.
The longitude and latitude you choose must be a multiple of the ecmwf grid.
No spatial interpolation is performed and must be expanded. To create this list of points you can use the python script create_list_points.py located in the folder 2_input_ww3/spec.

.. code-block:: bash

   python create_list_points.py

This python script reads a netcdf file from a WW3 simulation and extracts the points at the LBC of the domain.

Once the list_points file has been created, you can compile the fortran :

.. code-block:: bash

   ./compile_codes.sh

Two Fortran programs are compiled: 

* the extract_spectra_for_ww3.f90 program. This program is an adaptation of the script given by Jean Bidlot to the ECMWF. It converts the grib files containing the spectra into txt files.

* the spectra_from_mfwam_to_ww3_spectra.f program. This program reads the txt files created by the previous program and creates spec files that can be read by the ww3_bound program.

After compiling the fortran program, you need to create a directory called ''spectre'' where the files will be created. Then you need to adapt the date according to your directories in the file run_codes.sh and run it.

.. code-block:: bash

   ./run_codes.sh

At the end of this program, files are created in the spectrum directory (ww3.X.spec): one file for a lateral boundary condition point with all the required time steps in these files.

Once the .spec files have been created, you need to create the namelist ww3_bound.inp using the script create_ww3_bound.inp.py : 

.. code-block:: bash

   python create_ww3_bound.inp.py

You can then run the ww3_bound program to prepare the lateral boundary conditions for WW3.
To do this, go back to the 2_input_ww3 directory and modify the paths in the run_ww3_bound.sh script and run it via :

.. code-block:: bash

   sbatch run_ww3_bound.sh 

or

.. code-block:: bash

   ./run_ww3_bound.sh 

At the end of this step, you should find ''Writing data for time:'' at the end of the ww3_bound.out file.
This program generated a file called nest.ww3 corresponding to the lateral boundary conditions for WW3. This file will be used in the spinup and coupling simulations.

Prepare the IC : spinup (ww3_shel)
-----------------------------------------------------------------------

To prepare the initial conditions for WW3, you need to run the WW3 forced simulation again, but now with the LBC file.

To do this, you can go to A2_frc_ww3_spinup folder, modify the script run_ww3_shel.sh and the namelist ww3_shel.nml_A2_frc_ww3_spinup located in the 2_input_ww3 directory according to the period you are interested in.
Simulations lasting a few days are sufficient to prepare your initial conditions. The length of time depends on your domain.

The first lines correspond to the date of the SPINUP.
The DOMAINSTART corresponds to the start of the spinup.
The DOMAINSTOP corresponds to the end of the spinup which corresponds to the start of the simulation.

.. code-block:: fortran

   &DOMAIN_NML
      DOMAIN%START   = '20220215 180000'
      DOMAIN%STOP    = '20220216 000000'
    /

Then in the OUTPUT_DATE_NML.
DATEFIELD corresponds to the output field you wish to write.
A field is output every 100s.
DATERESTART corresponds to the restart file which we are going to generate and which will be written at the end of the spinup.

.. code-block:: fortran

   &OUTPUT_DATE_NML
      DATE%FIELD          = '20220215 180000' '100' '20220215 000000'
      DATE%RESTART        = '20220215 180000' '600' '20220216 000000'
   /


To run the simulation, you can execute the script run_ww3_shel.sh via :

.. code-block:: bash

   sbatch run_ww3_shel.sh 

or

.. code-block:: bash

   ./run_ww3_shel.sh 

At the end of the spinup simulation, you should find ''End of program'' at the end of the ww3_shel.out file.
In your simulation directory, you should find files called restartXXX.ww3. If you are running a 7-day simulation, you should find 7 files.
The last file corresponds to the start date of your coupled simulation (the number of restart files is defined in the ww3_shel.nml name list). Another file called out_grd.ww3 is created and corresponds to the grid parameters.

To check the spinup and for the rest of this documentation, you need to convert the ww3 output files to netcdf using the ww3_ounf program and watch the time evolution of the significant wave height, for example.
You should see a signal propagating from the edges of the domain inwards and reaching the whole simulation domain.

To create the netcdf file, you need to modify the scripts run_ww3_ounf.sh and ww3_ounf.nml according to your path and configuration, then you can run the program run_ww3_ounf.sh via :

.. code-block:: bash

   sbatch run_ww3_ounf.sh 

or 

.. code-block:: bash

   ./run_ww3_ounf.sh 

At the end of this stage, a netcdf file is created containing all the required grid parameters.

If the spinpup is correct, you should use the script create_restart_file_from_WW3_file.py (located in 2_input_ww3) to create the file rstrt_WW3.nc. This script will read the netcdf file created previously, the fields inside the rstrt_WW3.nc file must be the time of the end of the spinup simulation.

.. code-block:: bash

   python create_restart_file_from_WW3_file.py

Prepare namelist for the coupled simulation : ww3_shel.nml
-----------------------------------------------------------------------

Finally, you need to prepare the namelist for the coupled run. For the purposes of this documentation, the namelist ww3_shel.nml_C1_cpl_mnh_ww3_croco is already adapted to a coupled ocean-wave-atmosphere simulation.
The information in this namelist concerning coupling is presented here:

.. code-block:: fortran

   &INPUT_NML
      INPUT%FORCING%WATER_LEVELS  = 'C'
      INPUT%FORCING%CURRENTS      = 'C'
      INPUT%FORCING%WINDS         = 'C'
   /
   
   &OUTPUT_TYPE_NML
      TYPE%FIELD%LIST          = 'DPT WND HS CUR FP DIR SPR TWS CHA'
      TYPE%COUPLING%SENT       = 'T0M1 THM OHS ACHA AHS CUR FWS'
      TYPE%COUPLING%RECEIVED   = 'SSH CUR WND'
   /

Information to be entered in the namelist.
Start and end date of the simulation.
-DATEFIELD file output frequency.
-DATECOUPLING coupling frequency in seconds.
-DATERESTART restart file writing frequency.

.. code-block:: fortran

   &DOMAIN_NML
      DOMAIN%START   = '20220216 000000'
      DOMAIN%STOP    = '20220216 180000'
   /

   &OUTPUT_DATE_NML
      DATE%FIELD          = '20220216 000000'  '3600' '20220216 181000'
      DATE%COUPLING       = '20220216 000000'   '100' '20220216 181000'
      DATE%RESTART        = '20220216 000000' '86400' '20220216 181000'
   /

List of necessary files for WW3 : 2_input_ww3
-----------------------------------------------------------------------

.. code-block:: bash

   2_input_ww3
      | -- mapsta.ww3             ! from ww3_grid program
      | -- mask.ww3               ! from ww3_grid program
      | -- mod_def.ww3            ! from ww3_grid program
      | -- ST4ABUHF2.bin          ! from ww3_grid program
      | -- nest.ww3               ! from ww3_bound program
      | -- restart.ww3            ! from spinup simulation
      | -- rstrt_WW3.nc           ! from python script
      | -- ww3_shel.nml           ! namelist for the coupled simulation

.. _preprocessing_OASIS:

OASIS
====================================

Create remap files
-----------------------------------------------------------------------

First you need to get create_rmp_files tool from my public Git repository:

.. code-block:: bash

   git clone https://github.com/JorisPianezze/create_rmp_files_for_oasis.git


To create the weight files for coupling, use the python script located in the 4_input_oasis/create_rmp_files_for_oasis/data directory and run it:

.. code-block:: bash

   python create_grids_areas_and_masks_files_for_rmp_from_mnh_and_croco.py

3 netcdf files are created following execution of this script: areas.nc, grids.nc and masks.nc. These files contain information for the Meso-NH/SurfEx, WW3 and CROCO grids.

It is then necessary to go to the previous folder (4_input_oasis/create_rmp_files_for_oasis) and use the scripts run_testinterp.sh to create the weight files. The OASIS tools in this folder must be compiled first:

.. code-block:: bash

   make

You can then create the weight files using the following command:

.. code-block:: bash

   ./run_testinterp_atmt_ocnt.sh 2_1_1

At the end of the execution of this script a create_rmp_files_for_oasis_atmt_ocnt_bilinear/rundir_2_1_1 was created, this folder contains the file rmp_atmt_to_ocnt_DISTWGT_4.nc which corresponds to the file necessary for OASIS to carry out the spatial interpolation.
It is necessary to repeat the operation for each run.sh script.

Once the rmp.nc files have been created, go to the 4_input_oasis folder and run the create_link.sh script to create links to the rstrt_WW3.nc and rmp.nc files in this folder:

.. code-block:: bash

   ./create_link.sh

Prepare the namelist for the coupled simulation: namcouple
-----------------------------------------------------------------------

In this section, the namcouple namelist located in the 4_input_oasis folder will be used. For the purposes of this documentation, you do not need to modify the namelist; the following paragraphs are for your information.

The namcouple file is the OASIS coupler configuration file containing all the information relating to exchanges between models. It follows a precise syntax which is clearly explained in the OASIS3-MCT user manual. For example, it does not accept blank lines and there is no syntax check on this file: the slightest error means a crash without any indication.

List of necessary files in the directory 4_input_oasis
-----------------------------------------------------------------------

.. code-block:: bash

   4_input_oasis
      | -- rst_*.nc       ! restart files
      | -- rmp*.nc        ! remap files
      | -- namcouple      ! namelist for the coupled simulations
