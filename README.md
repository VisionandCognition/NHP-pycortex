pycortex
========
[![Build Status](https://travis-ci.org/gallantlab/pycortex.svg?branch=master)](https://travis-ci.org/gallantlab/pycortex)

[![quickflat demo](https://raw.github.com/jamesgao/pycortex/master/docs/wn_med.png)](https://gallantlab.github.io/pycortex)

Pycortex is a software library that allows you to visualize fMRI or other volumetric neuroimaging data on cortical surfaces. In this fork, some code has been adapted by Chris Klink (c.klink@nin.knaw.nl) to make pycortex compatible with the NHP neuroimaging pipelines (especially [NHP-Freesurfer](https://github.com/VisionandCognition/NHP-Freesurfer)) of the Netherlands Institute for Neuroscience.

Quickstart
----------
Install [conda](https://conda.io/projects/conda/en/latest/user-guide/install/index.html) to manage virtual python environments. The do the following:

```bash
# create a conda environment with Python 3
# here we call it 'neuro' and assume the default Python for conda is Python 3
conda create --name neuro  
# activate the virtual environment
conda activate neuro    
# install some prerequisite packages
pip install numpy Cython scipy h5py nibabel matplotlib Pillow numexpr tornado lxml networkx jupyter jupyterlab
# install this adapted pycortex release
git clone https://github.com/VisionandCognition/NHP-pycortex.git
cd NHP-pycortex
python setup.py develop
```

Pycortex uses a configuration file to specify where things are save. Edit to make it point to where you want your files.
You can check the location of the filestore after installing by running:

```python
import cortex
cortex.database.default_filestore
```

And you can check the location of the config file by running:

```python
import cortex
cortex.options.usercfg
```
If you want to move the filestore, you need to update the config file:

```
[basic]
filestore=/abs/path/to/filestore
```

Documentation
-------------
Pycortex documentation is available at [https://gallantlab.github.io/pycortex](https://gallantlab.github.io/pycortex). You can find many examples of pycortex features in the [pycortex example gallery](https://gallantlab.github.io/pycortex/auto_examples/index.html). These examples are also present in this repository. In `examples` you can find python code you can execute in [IPython](http://www.ipython.org/). In `example-notebooks` you will find [Jupyter notebooks](https://jupyter.org/) with the same examples.



Citation
--------
If you use pycortex in published work, please cite the [pycortex paper](http://dx.doi.org/10.3389/fninf.2015.00023):

_Gao JS, Huth AG, Lescroart MD and Gallant JL (2015) Pycortex: an interactive surface visualizer for fMRI. Front. Neuroinform. 9:23. doi: 10.3389/fninf.2015.00023_

Getting help
-----------
Please post on [NeuroStars](https://neurostars.org/) with the tag `pycortex` to 
ask questions about how to use Pycortex. For NHP-pycortex specific questions, post an issue on this repository.
