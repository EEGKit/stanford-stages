# Stanford Stages

An automated sleep stage scoring and narcolepsy identification software program written in Python.

The stanford-stages app uses machine learning to perform automated sleep stage scoring and narcolepsy identification of nocturnal polysomnography (PSG) studies, which must be in European Data Format (.edf).
The software requires previously trained models to accomplish this as described in the 2018 manuscript ["Neural network analysis of sleep stages enables efficient diagnosis of narcolepsy."](https://www.nature.com/articles/s41467-018-07229-3)
## Branches

Git provides support for multiple _branches_ of development.  Notable branches in the stanford-stages repository include:

1. __Master__ 

   The ___master branch___ provides GPU (Nvidia) compatibility and supports TensorFlow 2.0 and GPflow 2.0.  Source code from this branch can be used to automatically score sleep stages using the original hypnodensity models (ac.zip) included below.  It is _not_ compatible with the previously trained narcolepsy classification models.  We are working on developing new classification models.

1. __Manuscript__ 

   The ___manuscript branch___ includes the code presented with the 2018 manuscript.  It uses earlier versions of GPflow and TensorFlow, to retain compatibility with the narcolepsy classification models (gp.zip) presented with the manuscript.  
  
    See the [__manuscript branch__ README page](https://github.com/Stanford-STAGES/stanford-stages/blob/master/documentation/Manuscript_Branch.md) for configuration instructions using this branch and links to the narcolepsy models.  

1. __Beta__ 

   The ___beta branch___ serves as development branch for making and testing changes to the ___master branch___ and is not always stable.      

   **NOTE**: Only Python 3.6 and later is supported. Previous versions of Python have been shown to yield erroneous results. 

# Getting started

The stanford-stages application is most readily configured and used by editing the repository's __stanford_stages.json__ file and passing it to the __run_stanford_stages.py__ script.  Instructions on how to run the application are given first followed by instructions on how to configure the application.  In practice, you will need to configure the application before it can be run.      

_Note_: These instruction are for the master branch.

## Installation and Dependencies

Special thanks to Giorgio for the following instructions:

After being able to construct the correct environment from the initial __requirements.txt__ file of Stanford Stages. I wanted to share the libraries I used, as well the source from where I downloaded the once not available through conda/pip command. Attached to this email you can find the ‘requierment.txt’ from my final environment (Py 3.7.1). Although, to construct the environment I had to install the libraries in a certain order to avoid inconsistency error. The pipeline was the following:

```
conda update -n base -c defaults conda
conda install -c conda-forge tensorflow==2.6.0
conda install -c conda-forge pandas==0.25.1
conda install -c conda-forge scikit-image==0.15.0
conda install -c conda-forge pyedflib
pip install C:\Users\giorg\Downloads\GPflow-2.1.4.tar.gz
conda install -c conda-forge PyWavelets==1.0.3
conda install -c conda-forge numpy==1.19.5
conda install -c conda-forge scipy==1.3.2
```
`gpflow==2.1.4` --> https://github.com/GPflow/GPflow/releases/tag/v2.1.4

The package `tf_nightly_2.0_preview` was not installed and it worked properly, as far as I understood tensforlow installs it as a dependency. In any case it can be found here:  `tf_nightly_2.0_preview` -> https://pypi.org/project/tensorflow/2.0.0b1/

## Running the application

The __run_stanford_stages.py__ script can be called from within the Python runtime environment, or by using one of the shell or batch scripts included below.

### From python      

```python
import run_stanford_stages
run_stanford_stages.run_using_json_file(json_file='/path/to/stanford_stages.json')
```

### From a shell or batch script

Two, operating specific (OS) scripts are included with the repository: 

* __run_stanford_stages.bat__ (Windows) 
* __run_stanford_stages.sh__ (Mac OSX and Linux)

These scripts invoke the Python runtime and call __run_stanford_stages.py__ using the __stanford_stages.json__ configuration file found in the same directory.  

Edit the script associated with your OS to use a different .json configuration file that you have, perhaps created and stored elsewhere.   
 
_Note_: Linux and Unix users will need to modify (chmod) the _run_stanford_stages.sh_ file in order to run it as a standalone executable. 

## Configuring the application

Use the following link for configuring the __run_stanford_stages.py__ wrapper:

[Instructions for configuring the application using JSON](https://github.com/Stanford-STAGES/stanford-stages/blob/master/documentation/JSON_Configuration.md)

Developers interested in bypassing the __run_stanford_stages.py__ wrapper can call __inf_narco_app.py__'s `main` method directly, using a subset of these json arguments. Documentation for these json arguments is included in the same documentation [here](https://github.com/Stanford-STAGES/stanford-stages/blob/master/documentation/JSON_Configuration.md).

# Application Output

This section describes the application's output.  These outputs may be displayed in the console window and/or saved to disk as described in the JSON configuration section readme.

## Narcolepsy diagnosis

The algorithm produces values between −1 and 1, with 1 indicating a high probability of narcolepsy. The cut-off threshold between narcolepsy type 1 and “other“ is set at −0.03.  See [Neural network analysis of sleep stages enables efficient diagnosis of narcolepsy](https://www.nature.com/articles/s41467-018-07229-3) for details.  

## Hypnogram

The hypnogram provides a numeric indicator of wake or sleep stage for every epoch scored.  
By default, epochs are scored in 15 s intervals as follows:

* Wake: `0`
* Stage 1 sleep: `1`
* Stage 2 sleep: `2`
* Stage 3/4 sleep: `3`
* Rapid eye movement (REM) sleep: `5`
* Unscored: `7`

Note: Unscored data includes the time before lights off and after lights on, i.e. provided these values have been set manually (see JSON configuration).  Periods of flatlined data, in excess of 5 minutes can produce divide by 0 conditions in the software and are left unscored as well.

## Hypnodensity

### As a text file
The hypnodensity text output is the same length (number of epochs/rows) as the hypnogram.  Instead of a sleep score, however, five probabilities are given representing the likelihood of the sleep stage corresponding to its column for the current epoch.  Probabilities are ordered starting with wake, and moving to deeper stages of sleep, with REM sleep last.

For example:

<pre>0.24 0.01 0.02 0.03 0.70</pre>

Represents

* Wake: 24% likelihood
* Stage 1 sleep: 1% likelihood
* Stage 2 sleep: 2% likelihood
* Stage 3/4 sleep: 3% likelihood
* REM sleep: 70% likelihood

### As a plot

The x-axis of the hypnodensity plot represents the epochs from the beginning to the end of the study, which are measured in 15 s intervals by default.  The y-axis ranges from
0 to 1 and represents the hypnodensity - the probability of each sleep stage and wake.  Different colors are used to represent wake and each sleep stage.  For each epoch, the probability of wake is drawn first as a vertical line to the proportion of the y-axis range it represents. Stage 1 is then stacked on top of this, followed by Stages 2, 3/4, and REM sleep such that the entire spectrum is covered.  Color matching is as follows:

* Wake: white
* Stage 1 sleep: pink
* Stage 2 sleep: aqua/turquoise
* Stage 3/4 sleep: blue
* REM sleep: green

# Models

The sleep staging classification models are hosted externally at the following location:

* ac.zip - https://www.informaton.org/narco/ml/ac.zip [770 MiB, 807 MB]
* <strike>gp.zip</strike>*
 
Download and extract the [ac.zip](https://www.informaton.org/narco/ml/ac.zip) file to your computer system.  These models, along with a PSG (.edf file) are necessary for the software to run.  The .zip file may be deleted once its contents have been extracted.  

See the JSON configuration instructions for how to specify the location model locations for use with the software.
   
When complete the 'ml/' directory tree should like:<pre>
ac/
<strike>gp/</strike>
noiseM.mat
scaling</pre>

*_Note_ See _manuscript branch_ [readme](https://github.com/Stanford-STAGES/stanford-stages/blob/master/Manuscript_Branch_README.md) for links to the gp.zip file originally posted with the manuscript.  

# Licensing and Attribution

This software is licensed under the Creative Commons CC-BY 4.0 (https://creativecommons.org/licenses/by/4.0/).  

Include the following citation when attributing use of this software:

Jens B. Stephansen, Alexander N. Olesen, Mads Olsen, Aditya Ambati, Eileen B. Leary, Hyatt E. Moore, Oscar Carrillo et al. ["Neural network analysis of sleep stages enables efficient diagnosis of narcolepsy."](https://www.nature.com/articles/s41467-018-07229-3). Nature Communications 9, Article number:5229 (2018).

## Academic citation

Please use the following citation when referencing this software:

Jens B. Stephansen, Alexander N. Olesen, Mads Olsen, Aditya Ambati, Eileen B. Leary, Hyatt E. Moore, Oscar Carrillo et al. ["Neural network analysis of sleep stages enables efficient diagnosis of narcolepsy."](https://www.nature.com/articles/s41467-018-07229-3). Nature Communications 9, Article number:5229 (2018).

```
@article{Stephansen2018,
author = {Stephansen, Jens B and Olesen, Alexander N and Olsen, Mads and Ambati, Aditya and Leary, Eileen B and Moore, Hyatt E and Carrillo, Oscar and Lin, Ling and Han, Fang and Yan, Han and Sun, Yun L and Dauvilliers, Yves and Scholz, Sabine and Barateau, Lucie and Hogl, Birgit and Stefani, Ambra and Hong, Seung Chul and Kim, Tae Won and Pizza, Fabio and Plazzi, Giuseppe and Vandi, Stefano and Antelmi, Elena and Perrin, Dimitri and Kuna, Samuel T and Schweitzer, Paula K and Kushida, Clete and Peppard, Paul E and Sorensen, Helge B. D. and Jennum, Poul and Mignot, Emmanuel},
doi = {10.1038/s41467-018-07229-3},
journal = {Nature Communications},
number = {5229},
title = {{Neural network analysis of sleep stages enables efficient diagnosis of narcolepsy}},
volume = {9},
year = {2018}
}
```
