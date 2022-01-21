# Video Classification

This repository introduces Video Classification through a detailed exploration of a dataset and a pretrained model. Video Classification assigns a set of scores to a video clip, where scores typically correspond to action classes.

![Upsampling](images/video_classification_task.png)

The primary contributions of this repository are

1. A Jupyter Notebook
    - Demonstrating how to download, organize, explore and visuzlize the [Kinetics Human Action Video Dataset](https://deepmind.com/research/open-source/kinetics).
    - Demonstrating how to download from the [Tensorflow hub](https://www.tensorflow.org/hub) a pretrained [I3D video classification model](https://deepmind.com/research/open-source/i3d-model), and test it on small samples of the Kinetics dataset. 
    - Highlighting challenges and considerations of working with video data. 
2. A script that allows the user to perform evaluation of I3D on larger samples, or full splits, of the Kinetics dataset.

>     For an introduction to the broad field of video understanding, to which video classification belongs, check out the blog [An Introduction to Video Understanding: Capabilities and Applications](https://blog.fastforwardlabs.com/2021/12/14/an-introduction-to-video-understanding-capabilities-and-applications.html).

Instructions are now given to use this repository, both for general use (on a laptop, say), and for Cloudera CML and CDSW.
We'll first describe the repository's content, then go through how to run everything.

## Structure

The folder structure of the repo is as follows:

```
.
├── cml             # Contains scripts that facilitate the project launch on CML.
├── data            # Location for storing video data. 
├── scripts         # Contains the evaluation script.
└── vidbench        # A small library of useful functions.
```
There is also an `images` directory which holds figures used in the Jupyter Notebook. 

Let's examine each of the important folders in turn.

### `cml`
These scripts facilitate the automated project setup on CML and are triggered by the declarative 
pipeline as defined in the `.project-metadata.yaml` file found in the project's root directory.

```
cml
├── install_dependencies.py
└── download_data.py
```

### `vidbench`
This is a small Python library to facilitate data downloading and processing, as well as model inference and evaluation. 

Its structure is as follows:
```
vidbench
├── data
│   ├── fetch.py     
│   ├── load.py
│   └── process.py
├── arguments.py
├── models.py
├── predict.py
├── visualize.py
└── utils.py
```

### `scripts`
The script contained here is used in conjunction with the `vidbench` library above and performs large-scale evaluation of a video classification model. 

```
scripts
├── config.txt
└── evaluate.py
```

### `data`
This AMP makes use of the [Kinetics Human Action Video Dataset](https://arxiv.org/abs/1705.06950), of which there are several versions. 
Documentation on these data sources can be found [here](https://github.com/cvdfoundation/kinetics-dataset/tree/ed85ec6b29aa569f0e4b21edbc1cd90818446ea4). 

If the project is launched on CML, a small selection of data will be pre-downloaded. If not, there are cues in the notebook to execute a data download. 

## Deploying on CML
There are three ways to launch this project on CML:

1. **From Prototype Catalog** - Navigate to the Prototype Catalog on a CML workspace, select the "Video Classification" tile, click "Launch as Project", click "Configure Project"
2. **As ML Prototype** - In a CML workspace, click "New Project", add a Project Name, select "AMPs" as the Initial Setup option, copy in the repo URL, click "Create Project", click "Configure Project"
3. **Manual Setup** - In a CML workspace, click "New Project", add a Project Name, select "Git" as the Initial Setup option, copy in the repo URL, click "Create Project". 
Launch a JupyterLab Session with at least 16GiB of memory and 2vCPUs. Then follow the Installation instructions in the notebook.

### Installation
The code and applications within were developed against Python 3.6.9, and are likely also to function with more recent versions of Python.

To install dependencies, first create and activate a new virtual environment through your preferred means, then pip install from the requirements file. We recommend:

```python
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

In CML or CDSW, no virtual env is necessary. Instead, inside a JupyterLab Session (with at least 2 vCPU / 16 GiB Memory), simply run through the Setup cells in the Jupyter Notebook. 

### Running the evaluation script

The `scripts/evaluate.py` script requires the `scripts/config.txt` file which handles all the input arguments. First specify your choices in the config file, and then run the following in an open Session: `!python3 scripts/evaluate.py @scripts/config.txt`

Alternatively, this script can also be run automatically with the **Jobs** abstraction by first clicking on **Jobs**, then **New Job**, and then selecting `scripts/evaluate.py` under **Script**. Enter `@scripts/config.txt` under **Arguments** and ensure that the job is given at least 2vCPU/16GiB of resources. Then click **Create Job**. You can now run the job as often as you like or schedule the job at your convenience. 

-----------
### Notes on resources: 

Video data are rich and complex and, as such, require a considerable amount of storage and memory considerations. 

#### Storage
The Kinetics video data used in this repo is only a fraction of what is available for model evaluation. Downloading the full, raw dataset would require 30-60GiB (depending on whether you download the `validation` or `test` set). Our `KineticsLoader` class converts these raw videos to NumPy arrays and caches them for re-use. These cached, pre-processed video clips require even more storage -- up to a TB for the full validation set. We recommend starting with only a few hundred videos and assessing your storage capabilities before running a model evaluation on the full Kinetics `test` or `validation` sets. 

#### Memory
In addition to the storage requirements, processing these data through a pre-trained CNN model also requires considerable RAM. We recommend a minimum of 16GiB which is enough to process a batch of 8 videos. Large batch sizes will require additional resources. 

#### GPUs
This notebook currently supports CPU-only. In the future, we look forward to upgrading this demo to utilize GPUs. 

-----------