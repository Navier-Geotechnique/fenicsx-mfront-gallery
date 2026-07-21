# Installing `FEniCSx` and `MFront` via Anaconda on Ubuntu

Both `FEniCSx` and `MFront` are now available through the [conda-forge](https://conda-forge.org/) channel:

- [`fenics-dolfinx`](https://anaconda.org/conda-forge/fenics-dolfinx)
- [`fenics-libdolfinx`](https://anaconda.org/conda-forge/fenics-libdolfinx)
- [`mfront`](https://anaconda.org/conda-forge/mfront)
- [`mgis` – MFrontGenericInterfaceSupport](https://anaconda.org/conda-forge/mgis)

An additional important package is [`dolfinx_materials`](https://github.com/bleyerj/dolfinx_materials).

**Note:** At the date of the last modification of this guide (16/07/2026), `dolfinx_materials` (v0.4.0) is designed for `FEniCSx` **0.10.x**, while the default version available via conda-forge is already 0.11.0 (or later). Installing 0.11 will break `dolfinx_materials` with `ImportError: cannot import name '_assign_block_data'`, so `fenics-dolfinx` **must be pinned to 0.10**. Note also that `fenics-dolfinx` is not compatible with Python 3.13+.

---

## Step 1 – Install Anaconda on Ubuntu

### Download Miniconda

You can download the installer from the [official Anaconda website](https://www.anaconda.com/download) or use the command below:

```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
```

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

### Create and activate a virtual environment, with a name of mfront (for example)

```bash
conda create -n mfront python=3.12
conda activate mfront
```

### Install additional useful packages

```bash
conda install -c conda-forge jupyterlab pandas matplotlib scipy meshio pyvista openpyxl trame trame-vtk trame-vuetify ipywidgets gmsh tqdm python-gmsh
```

### Download `dolfinx_materials`

Clone the repository now — its demos provide the `.mfront` files used to verify the MFront installation in the next step. The package itself will be installed later, once `FEniCSx` is available:

```bash
cd ~
git clone https://github.com/bleyerj/dolfinx_materials.git
```

### Install MFront and MGIS

```bash
conda install -c conda-forge mfront mgis
```

#### Configure `TFELHOME`

The conda-forge build of MFront ships with a **build-time placeholder path** baked in, which no longer exists after the package is relocated into your environment. As a result, `mfront --obuild` generates a `Makefile.mfront` pointing to a non-existent `include/` directory and compilation fails with:

```
fatal error: TFEL/Raise.hxx: No such file or directory
```

You can check this by running:

```bash
tfel-config --includes
# -I/home/conda/feedstock_root/build_artifacts/bld/rattler-build_mfront_.../placehold_placehold.../include
```

MFront resolves its installation prefix from the `TFELHOME` environment variable first, so setting it to `$CONDA_PREFIX` fixes the issue:

```bash
export TFELHOME="$CONDA_PREFIX"
tfel-config --includes
# -I/home/<user>/anaconda3/envs/mfront/include
```

To make this automatic, add an activation hook so that `TFELHOME` is set whenever the environment is activated and unset on deactivation. This keeps the variable scoped to this environment only and does not pollute `~/.bashrc` or other environments:

```bash
mkdir -p "$CONDA_PREFIX/etc/conda/activate.d" "$CONDA_PREFIX/etc/conda/deactivate.d"

cat > "$CONDA_PREFIX/etc/conda/activate.d/tfel.sh" << 'EOF'
export TFELHOME="$CONDA_PREFIX"
EOF

cat > "$CONDA_PREFIX/etc/conda/deactivate.d/tfel.sh" << 'EOF'
unset TFELHOME
EOF
```

Reactivate and verify:

```bash
conda deactivate
conda activate mfront
echo "$TFELHOME"     # should print the path of your conda environment
```


#### Verification

Compile the `StationaryHeatTransfer.mfront` behaviour shipped with the `dolfinx_materials` demos:

```bash
cd ~/dolfinx_materials/demos/mfront/heat_transfer/
rm -rf include src
mfront --obuild --interface=generic StationaryHeatTransfer.mfront
```

Removing the `include/` and `src/` directories beforehand is recommended, since MFront reuses previously generated files and a stale `Makefile.mfront` may still contain the wrong include paths.

The build succeeds if `src/libBehaviour.so` is generated. A warning about tangent operator blocks may appear; it is harmless here.

### Install FEniCSx

Pin the version to 0.10 for compatibility with `dolfinx_materials` (see the note at the top):

```bash
conda install -c conda-forge "fenics-dolfinx=0.10"
```

This installs the whole FEniCS stack consistently (`fenics-basix`, `fenics-ffcx`, `fenics-ufl`, `fenics-libdolfinx`) without touching `petsc4py`, MPI or NumPy.

### Install `dolfinx_materials`

Now that `FEniCSx` is available, install the package cloned earlier:

```bash
cd ~/dolfinx_materials
pip install .
```

Verify the installation:

```bash
python -c "from dolfinx_materials.solvers import NonlinearMaterialProblem; print('OK')"
```

If this raises `ImportError: cannot import name '_assign_block_data' from 'dolfinx.fem.petsc'`, your `fenics-dolfinx` is 0.11 or newer — downgrade it as described above.

---

## Step 3 – Run the Example Tests

### Nonlinear Heat Transfer

This example (from the demos of `dolfinx_materials`) solves a nonlinear steady-state heat transfer problem using an MFront material behavior.

The behaviour library has already been compiled during the verification step above. If needed, rebuild it with:

```bash
cd ~/dolfinx_materials/demos/mfront/heat_transfer/
mfront --obuild --interface=generic StationaryHeatTransfer.mfront
```

This generates the `include/` and `src/` directories.

#### Start JupyterLab

```bash
cd ~/dolfinx_materials/demos/mfront/heat_transfer/
jupyter lab &
```

The notebook `nonlinear_heat_transfer.ipynb` contains the full calculation setup.

**Note:** When defining the material, make sure to use the appropriate library extension: `.so` for Linux and `.dylib` for macOS.

```python
material = MFrontMaterial(
    os.path.join(current_path, "src/libBehaviour.so"), # Here
    "StationaryHeatTransfer",
    hypothesis="plane_strain",
)
```

## More demos are under development...
