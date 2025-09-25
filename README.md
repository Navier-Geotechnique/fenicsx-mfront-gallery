# Installing `FEniCSx` and `MFront` via Anaconda on Ubuntu

Both `FEniCSx` and `MFront` are now available through the [conda-forge](https://conda-forge.org/) channel:

- [`fenics-dolfinx`](https://anaconda.org/conda-forge/fenics-dolfinx)
- [`fenics-libdolfinx`](https://anaconda.org/conda-forge/fenics-libdolfinx)
- [`mfront`](https://anaconda.org/conda-forge/mfront)
- [`mgis` – MFrontGenericInterfaceSupport](https://anaconda.org/conda-forge/mgis)

An additional important package is [`dolfinx_materials`](https://github.com/bleyerj/dolfinx_materials).

**Note:** This package is designed for `FEniCSx` version 0.8.0, while the current `FEniCSx` version available via Conda is 0.9.0. As a result, some modifications are required to ensure compatibility.

The accompanying zip file includes key example scripts and test cases that will be used later.

---

## Step 1 – Install Anaconda on Ubuntu

### Download Miniconda

You can download the installer from the [official Anaconda website](https://www.anaconda.com/download) or use the command below:

```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
````

### Run the installer

```bash
bash Miniconda3-latest-Linux-x86_64.sh
```

### Activate Conda

Restart your terminal, or activate Conda manually:

```bash
source ~/.bashrc
```

---

## Step 2 – Install Required Packages

### Create and activate a virtual environment (optional)

```bash
conda create -n fenicsx
conda activate fenicsx
```

### Install additional useful packages

```bash
conda install -c conda-forge jupyterlab pandas matplotlib scipy meshio pyvista openpyxl trame trame-vtk trame-vuetify ipywidgets gmsh tqdm python-gmsh
```

### Install FEniCSx

```bash
conda install -c conda-forge fenics-dolfinx fenics-libdolfinx
```

### Install MFront and MGIS

```bash
conda install -c conda-forge mfront mgis
```

### Install the modified `dolfinx_materials`

Navigate to the directory where the modified version is located. This version has been adapted to ensure compatibility with the current `FEniCSx` version.

```bash
cd dolfinx_materials-0.3.0
pip install .
```

---

## Step 3 – Run the Example Tests

### Test 1 – Nonlinear Heat Transfer

This example (from demos of `dolfinx_materials`) solves a nonlinear steady-state heat transfer problem using an MFront material behavior.

#### Start JupyterLab

```bash
jupyter lab &
cd Files/Test_1_Nonlinear_Heat_Transfer/
```

#### Compile the MFront file

```bash
mfront --obuild --interface=generic StationaryHeatTransfer.mfront
```

This will generate `include/` and `src/` directories.
The notebook `nonlinear_heat_transfer.ipynb` contains the full calculation setup.

**Note:** When defining the material, make sure to use the appropriate library extension: `.so` for Linux and `.dylib` for macOS.

```python
material = MFrontMaterial(
    os.path.join(current_path, "src/libBehaviour.so"), # Here
    "StationaryHeatTransfer",
    hypothesis="plane_strain",
)
```

---

### Test 2 – Poroelastic Simulation

This example, developed by [Maxime Pierre](https://mhpierre.github.io/), simulates uniaxial loading on a poroelastic medium under plane strain conditions. The bottom boundary is fixed in vertical displacement, with drainage at the top.

#### Start JupyterLab

```bash
jupyter lab &
cd Test_2_Poroelasticity
```

#### Compile the MFront file

```bash
mfront --obuild --interface=generic Maxime_PoroElasticity.mfront
```

After running the Jupyter notebook, the simulation results (pore pressure, displacement, stress) are saved in the `results/` folder and can be visualized using ParaView.





## More demos are under development...