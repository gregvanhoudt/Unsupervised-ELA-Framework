Unsupervised-ELA-Framework
============================

## Introduction

This repository contains the source code, simulation models, and event logs for the **Unsupervised Event Log Abstraction** project as submitted to Information Systems (version 2023/11/10). The repository contains python, R, and Java code.

This project is licensed under the MIT License: https://choosealicense.com/licenses/mit/.

## Cloning the Repository

In order to start working with this repository, it is advised to clone it to your machine:

    git clone https://github.com/gregvanhoudt/Unsupervised-ELA-Framework.git

Once the repository is cloned, you can navigate to the root directory of the project and install the required dependencies for this project. Dependencies required for sub-applications in this repository are detailed below.

## File Storage

### BPMN - Process Models

For each event log, be it on the low- or high-level, process models are discovered for use in the conformance checking and complexity computation phases. This subfolder stores all these discovered process models in BPMN format.

### BPSim - Simulation Models

The simulation models used in the experiment are generated by running `App_BPSimulation.py`. Executing this application generates new business simulation models and stores the related process trees and BPMN process models. The main parameters are `tree_size` and `n`, with which you can determine the size of each generated business process and the number of processes to generate in total. In case you find the need to alter the distributions further used in the simulation model, you can adapt the `params` dictionary accordingly.

The transformation of this simulation model to an event log, is done by the L-Sim simulator. More information at https://www.lanner.com/en-gb/technology/l-sim-bpmn-simulation-engine.html.

### Logs

Because there is no open access to the L-Sim simulator, the outcome of business process simulation is shared in this repository. All generated low-level event logs are located in the Logs/.gz subfolder in a compressed format. Uncompressing the event logs is not necessary to execute the experimental framework.

### PetriNet - Process Models

For each event log, be it on the low- or high-level, process models are discovered for use in the conformance checking and complexity computation phases. This subfolder stores all these discovered process models in Petri Net format.

### ProcessTree - Process Models

For each event log, be it on the low- or high-level, process models are discovered for use in the conformance checking and complexity computation phases. This subfolder stores all these discovered process models in Process tree format.

### src - Source Files

This directory contains all source files the applications detailed below depend upon. They are divided into, as with the applications, Abstraction, Evaluation, and Simulation.

Additionally, to perform the LPM-based abstraction in a batch approach, a modification had to be made to the ProM (https://promtools.org/) installation. This modification is stored in `ProM_Installation`. The script `VSC-Experiment-LPM.txt` contains the Java code used to instruct the CLI-instance of ProM to mine LPM's and abstract an event log. The script is written to elevate **one** event log.

Finally, the subdirectory `src` also contains an R script named `UnderstandBPMN.R` containing the code to compute the relevant complexity metrics, forming the proxy with comprehensibility.


## Applications

The root folder of the repository contains a number of `app_` python files.

- `App_Abstraction.py`

    The application to perform **session-based** abstraction. For this application to run successfully, due to backwards compatibility, `PM4py` version 2.2.2 and `graphviz` version 0.15.0 are required.

    The application requires parameters for clustering on the one hand, for which `dbscan` is recommended and a time threshold on the other hand. The duration of the abstraction algorithm on every event log is stored in `DurationLogging.csv` for performance analysis in terms of computation time when desired. When the algorithm is complete, the abstracted event log and all identified activity clusters are stored in a subfolder `.AbstractedLogs`. Details on how the algorithm works can be found in the manuscript.

- `App_BPSimulation.py`

    The application to generate business processes and their simulation models. To that end, the first step is generating process trees to describe the behavior one wishes to simulate. If desired, one can clean up the entire file structure to begin with fresh simulations. If not, comment out the line `utils.clean_filestruct()`

    After deciding on the tree size (min, mode, max) and the amount of process trees one wishes to generate, process trees via the `PT generator` in `PM4Py` are created for which the values of the `sequence`, `choice` and `parallel` parameters are stored in the file name. Each process tree is also immediatly converted to a BPMN model and written their respectful folders.

    The BPMN model is required to build the `BPSimPy` model on. The simulation length is dependent on the size of the previously created process tree, while all distributions regarding case arrival times, waiting times, processing times and gateway probabilities employ randomization wherever possible. All output is stored in `src/Simulation/LogGeneration.csv`.

- `App_Evaluation.py`

    After obtaining all low-level and high-level event logs, this application computes relevant evaluation metrics. To be more specific, this application computes the replay-based fitness (with also a line included for alignment-based fitness) and precision values. The simplicity metric is also included.

    This script is designed to be ran on the Flemish Supercomputer, implicating the need for the command line argument containing the event log name. As failsafe, the first event log is supplied to the application.

    In essence, this application loads all event logs on the low- and high-level along with their discovered process models (in Petri Net format). The next step is to *expand* the high-level process models in order to achieve the complete mapping in terms of event classes between the low-level event log and the high-level process model. This step is important as we desire to see how accurately the abstracted process still describes the original data. To that end, all discovered LPM patterns and session-based clusters are re-inserted into the high-level process model. See the section *Methodology* for more information.

    Finally, this application stores all output in their respective folders. The conformance checking results are by default stored in a `Stats` folder, where each event log outcomes are saved in a separate CSV file.

- `App_EventLogTranslation.py`

    This application transforms the L-Sim .log output files towards event logs following the .XES standard: https://xes-standard.org/. It accomplishes by reading the .log file as a tab separated table and restructuring it to the appropriate format. Before saving the event log as an XES file, the events are ordered on ascending timestamp.

    The log files to be processed are automatically identified by scanning the `Logs/.log` folder.

- `App_Petri_size_Counter.py`

    This application computes for a given abstraction level the size of the Petri nets. The size is important for evaluation purposes, as we are interested in how much smaller the higher-level event logs have become. The output of this application is a csv dataset containing the size of all high-level Petri Nets. One can determine the abstraction level for which the computation is made in the `main` part of the applications.

- `App_Petri_to_BPMN.py`

    This application transforms Petri Nets for a given abstraction level into BPMN models. A directory is provided to the `main` function of the application in which all Petri Nets are converted.

Each application contains an example execution of the script. The logical order of execution of these programs would be as follows:

1. Simulate event logs through `app_BPSimulation.py`. Even though our execution only used the `PT and Log Generator` tool in `PM4Py` for Process Tree generation, one can also use it as an alternative to generate the event logs themselves. We only required `App_EventLogTranslation.py` because of the use of L-Sim, and therefore, the requirement for start timestamps too.

2. Abstract the event logs with your given technique. In our execution, the session-based approach is available through `App_Abstraction.py` and the LPM-based approach is available as CLI application through `src/VSC-Experiment-LPM.txt`.

3. Convert all BPMN models to Petri Nets if needed with `App_Petri_to_BPMN.py`.

4. Compute the evaluation metrics with `App_Evaluation.py` and `src/UnderstandBPMN.R`. All statistics are then available at your own discretion for analysis.

5. If desired, the size of all Petri Nets can be computed with `App_Petri_Size_Counter` in case you want to add this as a measure in the evaluation of an abstraction technique.


To conclude, `EventLogs.csv` is included to provide a list of all event logs' names to iterate over where needed.


# Contributing to the project

We welcome contributions to the Unsupervised Event Log Abstract project. If you have any suggestions or bug reports, please feel free to create a new issue on GitHub.