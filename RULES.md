# General Rules and Instructions

## Plagiarism Note and Late Policy
Copying code (either from other students or from external sources) is strictly
prohibited! We will be using automatic anti-plagiarism tools, and violating this rule will lead to expulsion from the class. Late submissions will
generally not be accepted. In case of severe illness or emergency, please notify
Daniele and provide a relevant medical certificate.

## Provided Libraries
For each assignment, you will use the geometry processing library [libigl](https://github.com/libigl/libigl/), which includes implementations of many of the algorithms presented in class.
In particular, you will be using the [python bindings of igl](https://libigl.github.io/libigl-python-bindings/).

The `libigl` library includes a set of tutorials, an introduction to which can be found in the two previous links. You are advised to look over the relevant tutorials before starting the implementation for the assignments; you are also encouraged to examine the source code of all the library functions that you use in your code to see how they were
implemented.

No libraries apart from `libigl`, `numpy`, `meshplot`, and `scipy` are permitted unless permission is granted in advance.

## Report

For every assignment, you should provide a report in the form of a runnable Jupiter notebook including results for all steps described in the assignment. Please also add illustrations and text as needed to explain what you did. In addition to the notebook, you should also attach an html version of the notebook, exported using ``nbconvert``.

## Installing igl in Python (in a venv)

*** Warning: You must use Python 3.12 or older; Python 3.13 is not yet supported by libigl ***

First set up a virtual environment
```bash
mkdir gp25
python3.12 -m venv gp25
source gp25/bin/activate
```

Then install `igl`, `meshplot`, and `jupiter` trough pip (do not use conda for igl, as it is outdated)
```bash
pip install libigl
pip install git+https://github.com/skoch9/meshplot/
pip install pythreejs
pip install notebook
```

### Using jupyter notebook
Launch the lab
```
gp25/bin/jupyter notebook
```

#### Toubleshooting

If you see
```
Renderer(camera=PerspectiveCamera(children=(DirectionalLight(color='white', intensity=0.6, position=(-0.016860…
<meshplot.Viewer.Viewer at 0x136ed5d90>
```

instead of a 3D view, follow the instructions here to install pythreejs: https://pythreejs.readthedocs.io/en/stable/

```
pip install pythreejs
jupyter labextension install jupyter-threejs
```

<!-- ### Using jupyter lab
You need first install `nodejs` with version >= 12.0.0, e.g.
```bash
conda install nodejs==14.7.0 -c conda-forge
```
Next install jupyter lab with version < 3.0, e.g.
```bash
conda install -c conda-forge jupyterlab==2.2.0
```
Then install two jupyter lab extensions
```bash
jupyter labextension install jupyter-threejs
jupyter labextension install @jupyter-widgets/jupyterlab-manager
```
Now you will be able to launch the jupyter lab and see meshplot displays.
```bash
jupyter lab
```

Or you can simply download the file `environment.yml` uploaded to create conda env and all these should have already been set up. The first line of the `yml` file sets the environment's name.
```bash
conda env create -f environment.yml
```

To install the package manager conda we refer to its [website](https://docs.conda.io/en/latest/miniconda.html). -->
