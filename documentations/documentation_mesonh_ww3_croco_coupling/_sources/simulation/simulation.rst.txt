.. _RST Simulation:

Simulation
*********************************************************

Run a coupled simulation
------------------------------------

Depending on whether you want to run a coupled OA simulation or a coupled Meso-NH/WW3/CROCO simulation, you need to go to the B1_cpl_mnh_croco or C1_cpl_mnh_ww3_croco directory, respectively. In these directories, you need to change the path and run the script using the following command :

.. code-block:: bash

   sbatch run_cpl_mnh_ww3_croco.sub

or :

.. code-block:: bash

   ./run_cpl_mnh_ww3_croco.sub

At the end of this simulation, you should read ''(oasis_terminate) SUCCESSFUL RUN'' at the end of the debug.root.XX files. The standard outputs for Meso-NH/SurfEx, WW3 and CROCO are the same as in the forced simulations.

Outputs of a coupled simulation
------------------------------------

At the end of your simulation, a large number of files are created:

* Meso-NH /SurfEx output: standard output (the same as for a forced simulation)

* WW3 output: standard output (same as for a forced simulation)

* CROCO output: standard output (same as a forced simulation)

* OASIS output: debug files, etc.

Changing the coupling time step
------------------------------------

To change the coupling time step, you need to modify several namelists. The Meso-NH namelist (EXSEG1.nam) :

.. code-block:: fortran

   XTSTEP_CPL_WAVE = 600.0
   XTSTEP_CPL_SEA = 600.0
   
that of OASIS :

.. code-block:: fortran

   MNH__U10 WW3__U10 1 600 1 rst_A.nc EXPORTED

Coupling time step problem:
The XTSTEP in the MNH EXSEG must be identical to the LAG variable in the namcouple.
If this is not the case, the model may stop at the beginning of the simulation (e.g. at 600s). 

.. note:: the coupling time step must be a multiple of the time steps of all the models.
