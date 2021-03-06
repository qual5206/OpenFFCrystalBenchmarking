# Minimization of Crystallography Open Database Crystal Structures with OpenFF

This repository contains code for downloading a set of crystal structures
from the Crystallography Open Database (COD) (http://www.crystallography.net/cod/) and performing an energy minimization using
the OpenMM (https://openmm.org/) molecular simulation tookit and OpenFF (https://openforcefield.org/).

## Usage

Project was coded for Python 3.9. Refer to `requirements.txt` file for required packages.

## Current Workflow

1. Download list of desired structures from COD using list of COD IDs in `COD_ID_List.txt` and `COD_import.py` script. This downloads CIF files from COD and converts to PDB format using pybel. These are output in the `data/CIF` and `data/PDB` directories.
2. Create supercell from PDB file using PyMol `supercell` function. This cell is created to be large enought to satisfy that periodic boundary conditions are greater than 1.8nm. This is done in the `data/supercell_generation.py` script and output in the `data/PDB_supercell` directory.
3. Paramaterize using openFF and build MD simulation in openMM. Perform energy minimization and record the initial and final energy values as well as the RMSD between initial and final state.
   1. `main.py` will perform an energy minimization which includes custom functions to minimize the unit cell as well as positions. This script will also output initial and final box vectors.
   2. `main_no_box_minimization.py` will perform only energy minimization with respect to position using the built-in OpenMM energy minimization. 

Results that are output in the `data` directory:
1. `minimzation_results.csv` is a csv file with COD ID #, initial and final energies, and RMSD between those states. If box vector minimization is performed, the initial and final box vectors are also output.
2. `minimization_results.pkl` is a pickled file of the underlying Pandas DataFrame for the above data.
3. `rmsd_values.txt` is a tab separated text data file that only reports COD ID and RMSD values.
4. `initial_states` and `final_states` directories contain the .xml state files from OpenMM saved before and after energy minimzation.
5. `dminimized_PDB_supercell` directory contains the PDB files of the minimized supercell system.

## Examples

The `examples` folder contains scripts for a reduced subset of 10 COD ID values with the expected outputs stored in the `expected_data` directory. For all, ensure the working directory is set to `/examples`.
1. Run `example_COD_import.py`.
2. Run `example_supercell_generation.py`.
3. Run `example_main.py`.
4. Compare the generated data in `example_data` to `expected_data`.

## Current Issues

`COD_ID_List.txt` is not curated of entries that have data issues (non-matching coordinates between CIF and SMILES
and entries without SMILES strings). These entries currently produce errors that are logged in `errors.log`
during execution.

The current energy minimization algorithm fails for many systems due to periodic cell conditions not being
in a reduced format (http://docs.openmm.org/7.0.0/userguide/theory.html#periodic-boundary-conditions).

The energy minimization algorithm also rarely tries to reduce the periodic boundary conditions to less than 0.9 nm
which is half the non-bonded cutoff distance which causes an error.