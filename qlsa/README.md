# Quantum Linear Systems Algorithm

A sample implementation of a Quantum Linear Systems Algorithm (QLSA), the [Harrow–Hassidim–Lloyd (HHL)](https://doi.org/10.1103/PhysRevLett.103.150502) algorithm, using [Qiskit's HHL implementation](https://learn.qiskit.org/course/ch-applications/solving-linear-systems-of-equations-using-hhl-and-its-qiskit-implementation). The implementation uses Python scripts and Jupyter notebooks that utilize Qiskit libraries. The codes are designed to run the QLSA circuit on different quantum hardwares:
1. [IBM quantum-computing](https://quantum-computing.ibm.com/)
2. [IQM](https://www.meetiqm.com/)

An application to fluid dynamics is also provided. The fluid dynamics use case follows the work of [Bharadwaj & Srinivasan (2020)](https://www.sto.nato.int/publications/STO%20Educational%20Notes/STO-EN-AVT-377/EN-AVT-377-01.pdf) and [Gopalakrishnan Meena et al. (2024)](https://doi.org/10.1063/5.0231929). 

## References
* A. W. Harrow, A. Hassidim, and S. Lloyd, "Quantum algorithm for linear systems of equations," [Phys. Rev. Lett. 103, 150502](https://doi.org/10.1103/PhysRevLett.103.150502) (2009).
* S. S. Bharadwaj and K. R. Sreenivasan, "Quantum computation of fluid dynamics," [arXiv:2007.09147](arXiv:2007.09147) (2020).
* M. Gopalakrishnan Meena, K. C. Gottiparthi, J. G. Lietz, A. Georgiadou, and E. A. Coello Pérez, "Solving the Hele-Shaw flow using the Harrow-Hassidim-Lloyd algorithm on superconducting devices: A study of efficiency and challenges," [Physics of Fluids, 36 (10): 101705](https://doi.org/10.1063/5.0231929), (2024). ([preprint](http://arxiv.org/abs/2409.10857), [code](https://doi.org/10.5281/zenodo.13738192) - the current repo is adapted from this code)
* [Qiskit - Getting started](https://qiskit.org/documentation/getting_started.html)
* [Qiskit on IQM](https://iqm-finland.github.io/qiskit-on-iqm/user_guide.html)

# Installation

**NOTE**: For OLCF training/competitions, skip to the [Run](#run) section. Custom conda envs are created for your use. The following instruction is for installing the softwares from scratch.

All developments were done on [OLCF Odo](https://docs.olcf.ornl.gov/systems/odo_user_guide.html) and macOS. Based on steps in [OLCF Docs](https://docs.olcf.ornl.gov/quantum/quantum_software/hybrid_hpc.html#qiskit).

<details><summary>Notes for OLCF Odo:</summary>

  * Make sure to install your custom env in either `/ccsopen/proj/[projid]` or `/gpfs/wolf2/olcf/[projid]/proj-shared` (recommended). This is required to seamlessly run the plotting routines on OLCF JupyterHub.
  * Follow steps in [OLCF Docs](https://docs.olcf.ornl.gov/quantum/quantum_software/hybrid_hpc.html#qiskit) to load the base conda env (DO NOT install Qiskit) 
    ```
    module load miniforge3
    ```
    * Use `source activate` and/or `source deactivate` instead of `conda activate` or `conda deactivate`.
  * Or the following for your own Miniconda installation
    ```
    wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
    bash Miniconda3-latest-Linux-x86_64.sh -b -p /gpfs/wolf2/olcf/[projid]/proj-shared/[userid]/software/miniconda3-odo
    source /gpfs/wolf2/olcf/[projid]/proj-shared/[userid]/software/miniconda3-odo/bin/activate
    ```
</details>

Since the [QLSA circuit generator](https://github.com/anedumla/quantum_linear_solvers) requires an older version of Qiskit (which includes `qiskit-terra`), we need to create two envs:
1. To generate the circuit using older version of Qiskit
2. To run the circuit using Qiskit 1.0

## 1. Install Libraries to Generate the HHL Circuit
1. Make custom conda env
      ```
      conda create --name qlsa-circuit python=3.11
      conda activate qlsa-circuit
      ```
2. Install Qiskit and [quantum linear solver](https://github.com/anedumla/quantum_linear_solvers) package
      ```
      pip install -r requirements_circuit.txt --no-cache-dir
      ```
3. [Optional but recommended] Test the quantum linear solver package: [`test_linear_solver.py`](test_linear_solver.py)
      ```
      python test_linear_solver.py -nq 2
      ```
      <details><summary>Sample output from the test code:</summary>
      
      ```
      Simulator: AerSimulator('aer_simulator')
      ======================
      Time elapsed for classical:  
      0 min 0.00 sec
      Time elapsed for HHL:  
      0 min 0.21 sec
      ======================
      HHL circuit:
            ┌──────────────┐┌──────┐        ┌─────────┐
      q9_0: ┤0             ├┤4     ├────────┤4        ├
            │  circuit-165 ││      │        │         │
      q9_1: ┤1             ├┤5     ├────────┤5        ├
            └──────────────┘│      │┌──────┐│         │
      q10_0: ───────────────┤0     ├┤3     ├┤0        ├
                            │  QPE ││      ││  QPE_dg │
      q10_1: ───────────────┤1     ├┤2     ├┤1        ├
                            │      ││      ││         │
      q10_2: ───────────────┤2     ├┤1 1/x ├┤2        ├
                            │      ││      ││         │
      q10_3: ───────────────┤3     ├┤0     ├┤3        ├
                            └──────┘│      │└─────────┘
      q11: ─────────────────────────┤4     ├───────────
                                    └──────┘                 
      ====================== 
      Euclidean norm classical:    
      1.237833351044751
      Euclidean norm HHL:        
      1.2099806231118977 (diff (%): 2.250e+00)
      ======================
      Classical solution vector:
      [1.14545455 0.43636364 0.16363636 0.05454545]
      HHL solution vector:
      [1.11266151 0.43866345 0.16004585 0.08942688]
      diff (%): 
      [ 2.86288363  0.52703993  2.1942013  63.94928497]
      ```
      </details>
4. Install GPU version of Aer simulator. Needs NVIDIA GPUs (skip for OLCF Odo or systems without NVIDIA GPUs):
      ```
      pip install qiskit-aer-gpu==0.14.2 --no-cache-dir
      ```
      * [Optional but recommended] Test the installation:
      ```
      python -c "from qiskit_aer import AerSimulator; simulator = AerSimulator(); print(simulator.available_devices())"
      ```
      * [Optional but recommended] Test the installation with the sample code provided: [`test_gpu.py`](test_gpu.py)

      ```
      python test_gpu.py -nq 2 --gpu
      ```
      <details><summary>Sample output from the test code:</summary>

      ```
      Simulator: aer_simulator_statevector_gpu
      N qubits: 2; GPU: True; multiple-GPU: False;
      Time elapsed 1:  0 min 0.49 sec
      Time elapsed 2:  0 min 0.01 sec
      ```
      </details>

## 2. Install Libraries for the Solver Used to Run the HHL Circuit

**NOTE:** Before installing or using the solver environment, deactivate (`conda deactivate`) the circuit generation env `qlsa-circuit`.

1. Make custom conda env
      ```
      conda create --name qlsa-solver python=3.11
      conda activate qlsa-solver
      ```
2. Install Qiskit and other packages
      ```
      pip install -r requirements_solver.txt --no-cache-dir
      ```
3. [Optional but recommended] Test Qiskit installation: [`test_qiskit_installation.py`](test_qiskit_installation.py)
      ```
      python test_qiskit_installation.py -backtyp ideal
      ```
      <details><summary>Sample output from the test code:</summary>

      ```
      Backend: QasmSimulator('qasm_simulator')

      Total count for 00 and 11 are: {'00': 494, '11': 506}
              ┌───┐      ░ ┌─┐   
         q_0: ┤ H ├──■───░─┤M├───
              └───┘┌─┴─┐ ░ └╥┘┌─┐
         q_1: ─────┤ X ├─░──╫─┤M├
                   └───┘ ░  ║ └╥┘
      meas: 2/══════════════╩══╩═
                            0  1 
      ```
      </details>
      
      * Change `-backtyp` for different backends. Make sure to test all backend options offered.
      * **NOTE:** To run using IBM Provider (or IQM Resonance), you need to add your IBM Quantum Computing (or IQM) API KEY and instance to the [`keys.sh`](keys.sh) file and source activate it.
4. Install GPU version of Aer simulator. Needs NVIDIA GPUs (skip for OLCF Odo or systems without NVIDIA GPUs):
      ```
      pip install qiskit-aer-gpu==0.15.1 --no-cache-dir
      ```
      * [Optional but recommended] Test the installation:
      ```
      python -c "from qiskit_aer import AerSimulator; simulator = AerSimulator(); print(simulator.available_devices())"
      ```
      * [Optional but recommended] Test the installation with the sample code provided: [`test_gpu.py`](test_gpu.py)

      ```
      python test_gpu.py -nq 2 --gpu
      ```
      <details><summary>Sample output from the test code:</summary>

      ```
      Simulator: aer_simulator_statevector_gpu
      N qubits: 2; GPU: True; multiple-GPU: False;
      Time elapsed 1:  0 min 0.49 sec
      Time elapsed 2:  0 min 0.01 sec
      ```
      </details>

# Run

The instructions below are mainly for running interactively on OLCF Odo. Use the batch script [`submit_odo.sh`](submit_odo.sh) to [submit a batch job on OLCF Odo](https://docs.olcf.ornl.gov/systems/frontier_user_guide.html#batch-scripts). The first time you run the Python scripts, it may take some time to load the libraries.

1. Start interactive job
    ```
    salloc -A trn037 -p batch -N 1 -t 1:00:00
    ```    
2. Load Python environment:
    * If you are using the real backends (need to connect externally to OLCF), you need to use the following [proxies for connecting outside OLCF](https://docs.olcf.ornl.gov/quantum/quantum_software/hybrid_hpc.html#batch-jobs)
      ```
      export all_proxy=socks://proxy.ccs.ornl.gov:3128/
      export ftp_proxy=ftp://proxy.ccs.ornl.gov:3128/
      export http_proxy=http://proxy.ccs.ornl.gov:3128/
      export https_proxy=http://proxy.ccs.ornl.gov:3128/
      export no_proxy='localhost,127.0.0.0/8,*.ccs.ornl.gov'
      ```
    * [Recommended for OLCF training/competition] You can either activate the pre-built env:
      ```
      source /gpfs/wolf2/olcf/trn037/proj-shared/81a/software/miniconda3-odo/bin/activate
      ```
      For circuit generation:
      ```
      conda activate /gpfs/wolf2/olcf/trn037/proj-shared/81a/software/miniconda3-odo/envs/qlsa-circuit 
      ```
      For circuit solver:
      ```
      conda activate /gpfs/wolf2/olcf/trn037/proj-shared/81a/software/miniconda3-odo/envs/qlsa-solver
      ```
    * Or activate your own coda envs. Follow env activation instructions from the [Installation](#installation) steps.
3. Run QLSA circuit generator script: [`circuit_HHL.py`](circuit_HHL.py)
    ```
    srun -N1 -n1 -c1 python circuit_HHL.py -case sample-tridiag -casefile input_vars.yaml --savedata
    ```
    * **NOTE:** Make sure to save the circuit.
    * Try different case settings in the case file [`input_vars.yaml`](input_vars.yaml).

4. Run the QLSA solver: [`solver.py`](solver.py)
    ```
    srun -N1 -n1 -c2 python solver.py -case sample-tridiag -casefile input_vars.yaml -s 1000
    ```
    * **NOTE:** Before running the code, deactivate the circuit generation env (`qlsa-circuit`) and activate the solver env (`qlsa-solver`).
    * Experiment with different parameters in the code.

## Running on real hardware

* Make sure to export key variables in your key file: `source keys.sh`
* On OLCF Odo's interactive or batch modes, need to export proxies to connect outside OLCF. See instructions above.
* Running on IQM: 
    * Currently, results are not returned when running on IQM for circuits with more than 2 qubits. The code returns an error.
    * Need to use a post-processing code to retrieve results from the IQM Resonance portal. See the code [solver_getjob.ipynb](solver_getjob.ipynb) below.
* Running on IBM:
    * Usually the queue wait times are long. So the code will not return any results.
    * Check the IBM Dashboard to see if the job has finished and use the [solver_getjob.ipynb](solver_getjob.ipynb) to see the results.

# Visualization

See the following Jupyter notebooks for:

* [solver_getjob.ipynb](solver_getjob.ipynb): for retrieving jobs from online portal of IBM and IQM.
* [plot_compare-backends.ipynb](plot_compare-backends.ipynb): visualizing the results from various backends.
* [plot_Hele-Shaw.ipynb](plot_Hele-Shaw.ipynb): visualizing the results for solving the 2D Hele-Shaw flow problem.

## Using JupyterLab

* Resource: [OLCF JupyterHub](https://docs.olcf.ornl.gov/services_and_applications/jupyter/overview.html#jupyter-at-olcf)
* For OLCF training/competitions, use [OLCF Open JupyterHub](https://jupyter-open.olcf.ornl.gov/).
* In your local machine, always install and start JupyterLab in your base conda env.
* Use custom kernels as needed. See below for how to install custom kernels.
* To import your custom conda env to JupyterLab, follow the steps below, which have been modified from the [OLCF JupyterHub docs](https://docs.olcf.ornl.gov/services_and_applications/jupyter/overview.html#example-creating-a-conda-environment-for-rapids):
  * Install JupyterLab in your custom conda env. Do the rest of the steps in your base env.
  * Follow steps 1-2: Open a Terminal on JupyterLab using the Launcher.
  * Skip step 3: You don't have to create your own custom conda env as you have already done this on OLCF Odo.
  * Follow step 4 (source activate your custom env) using the custom env you created. The custom env should be created in:
      *  `/ccsopen/proj/[projid]` or `/gpfs/wolf2/olcf/[projid]/proj-shared` (recommended) for OLCF Odo.
  * Follow step 5 (make your env visible in JupyterLab) using your desired env name: `python -m ipykernel install --user --name [env-name] --display-name [env-name]`. You may have to pip install the library `wcwidth` on the Jupyter terminal: `pip install wcwidth`
  * Finally refresh your page and the Launcher (and kernel selector for notebooks) will have your env.

# Cite this work

* Gopalakrishnan Meena, M., Gottiparthi, K. C., Lietz, J. G., Georgiadou, A., and Coello Pérez, E. A. (2024). Solving the Hele–Shaw flow using the Harrow–Hassidim–Lloyd algorithm on superconducting devices: A study of efficiency and challenges. [Physics of Fluids, 36(10).](https://doi.org/10.1063/5.0231929)
* Gopalakrishnan Meena, M., Gottiparthi, K., & Lietz, J. (2024). qlsa-hele-shaw: Solving the Hele-Shaw flow using the Harrow-Hassidim-Lloyd algorithm on superconducting devices. Zenodo. https://doi.org/10.5281/zenodo.13738192

# Authors

* Murali Gopalakrishnan Meena (Oak Ridge National Laboratory)
* Michael Sandoval (Oak Ridge National Laboratory)

Contact: gopalakrishm@ornl.gov