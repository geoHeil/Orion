<p align="left">
<img width=15% src="https://dai.lids.mit.edu/wp-content/uploads/2018/06/Logo_DAI_highres.png" alt=“DAI-Lab” />
<i>An open source project from Data to AI Lab at MIT.</i>
</p>

<p align="left">
<img width=20% src="https://dai.lids.mit.edu/wp-content/uploads/2018/08/orion.png" alt=“Orion” />
</p>

[![Development Status](https://img.shields.io/badge/Development%20Status-2%20--%20Pre--Alpha-yellow)](https://pypi.org/search/?c=Development+Status+%3A%3A+2+-+Pre-Alpha)
[![PyPi Shield](https://img.shields.io/pypi/v/orion-ml.svg)](https://pypi.python.org/pypi/orion-ml)
[![CircleCI](https://circleci.com/gh/signals-dev/Orion.svg?style=shield)](https://circleci.com/gh/signals-dev/Orion)
[![Travis CI Shield](https://travis-ci.org/signals-dev/Orion.svg?branch=master)](https://travis-ci.org/signals-dev/Orion)
[![Downloads](https://pepy.tech/badge/orion-ml)](https://pepy.tech/project/orion-ml)
[![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/signals-dev/Orion/master?filepath=notebooks)

# Orion

Orion is a machine learning library built for telemetry data generated by satellites.

* License: [MIT](https://github.com/signals-dev/Orion/blob/master/LICENSE)
* Development Status: [Pre-Alpha](https://pypi.org/search/?c=Development+Status+%3A%3A+2+-+Pre-Alpha)
* Homepage: https://github.com/signals-dev/Orion
* Documentation: https://signals-dev.github.io/Orion

# Overview

Orion is a machine learning library built for telemetry data generated by Satellites.

With this data, our interest is to develop techniques to:

* identify rare patterns and flag them for expert review.
* predict outcomes ahead of time.

The library makes use of a number of *automated machine learning* tools developed under
["The human data interaction project"](https://github.com/HDI-Project) within the
[Data to AI Lab at MIT](https://dai.lids.mit.edu/).

With the ready availability of *automated machine learning* tools, the focus is on:

* domain expert interaction with the machine learning system;
* learning from minimal labels;
* explainability of model outputs;
* model audit;
* scalability;

## Leaderboard

In this repository we maintain an up-to-date leaderboard with the current scoring of the
pipelines according to the benchmarking procedure explained in the [benchmark documentation](
BENCHMARK.md).

Benchmark is ran on 11 datasets and we record the number of wins each pipeline has over the
ARIMA pipeline. Results obtained during benchmarking as well as previous releases can be 
found within [benchmark/results](benchmark/results) folder as CSV files. Summarized results can also 
be browsed in the following Google Sheets [document](https://docs.google.com/spreadsheets/d/1ZPUwYH8LhDovVeuJhKYGXYny7472HXVCzhX6D6PObmg/edit?usp=sharing) as well as the details Google Sheets [document](https://docs.google.com/spreadsheets/d/1HaYDjY-BEXEObbi65fwG0om5d8kbRarhpK4mvOZVmqU/edit?usp=sharing).


| Pipeline                  |  Outperforms ARIMA |
|---------------------------|--------------------|
| LSTM Dynamic Thresholding |          5         |
| Azure                     |          0         |

## Table of Contents

* [I. Data Format](#data-format)
   * [I.1 Input](#input)
   * [I.2 Output](#output)
   * [I.3 Dataset we use in this library](#dataset-we-use-in-this-library)
* [II. Orion Pipelines](#orion-pipelines)
   * [II.1 Current Available Pipelines](#current-available-pipelines)
* [III. Install](#install)
   * [III.1 Requirements](#requirements)
   * [III.2 Install with pip](#install-with-pip)
   * [III.3 Docker](#docker)
* [IV. Quickstart](#quickstart)
* [V. Database](#database)

# Data Format

## Input

**Orion Pipelines** work on time Series that are provided as a single table of telemetry
observations with two columns:

* `timestamp`: an INTEGER or FLOAT column with the time of the observation in
  [Unix Time Format](https://en.wikipedia.org/wiki/Unix_time)
* `value`: an INTEGER or FLOAT column with the observed value at the indicated timestamp

This is an example of such table:

|  timestamp |     value |
|------------|-----------|
| 1222819200 | -0.366358 |
| 1222840800 | -0.394107 |
| 1222862400 |  0.403624 |
| 1222884000 | -0.362759 |
| 1222905600 | -0.370746 |

## Output

The output of the **Orion Pipelines** is another table that contains the detected anomalous
intervals and that has at least two columns:

* `start`: timestamp where the anomalous interval starts
* `end`: timestamp where the anomalous interval ends

Optionally, a third column called `score` can be included with a value that represents the
severity of the detected anomaly.

An example of such a table is:

|      start |        end |    score |
|------------|------------|----------|
| 1222970400 | 1222992000 | 0.572643 |
| 1223013600 | 1223035200 | 0.572643 |

## Dataset we use in this library

For development, evaluation of pipelines, we include a dataset which includes several satellite
telemetry signals already formatted as expected by the Orion Pipelines.

This formatted dataset can be browsed and downloaded directly from the
[d3-ai-orion AWS S3 Bucket](https://d3-ai-orion.s3.amazonaws.com/index.html).

This dataset is adapted from the one used for the experiments in the
[Detecting Spacecraft Anomalies Using LSTMs and Nonparametric Dynamic Thresholding paper](https://arxiv.org/abs/1802.04431).
[Original source data is available for download here](https://s3-us-west-2.amazonaws.com/telemanom/data.zip).
We thank NASA for making this data available for public use.

# Orion Pipelines

The main component in the Orion project are the **Orion Pipelines**, which consist of
[MLBlocks Pipelines](https://github.com/MLBazaar/MLBlocks)
specialized in detecting anomalies in time series.

As ``MLPipeline`` instances, **Orion Pipelines**:

* consist of a list of one or more [MLPrimitives](https://github.com/MLBazaar/MLPrimitives)
* can be *fitted* on some data and later on used to *predict* anomalies on more data
* can be *scored* by comparing their predictions with some known anomalies
* have *hyperparameters* that can be *tuned* to improve their anomaly detection performance
* can be stored as a JSON file that includes all the primitives that compose them, as well as
  other required configuration options.

## Current Available Pipelines

In the **Orion Project**, the pipelines are included as **JSON** files, which can be found
in the subdirectories inside the [orion/pipelines](orion/pipelines) folder.

This is the list of pipelines available so far, which will grow over time:

| name | location | description |
|------|----------|-------------|
| ARIMA | [orion/pipelines/arima](orion/pipelines/verified/arima) | ARIMA based pipeline |
| LSTM Dynamic Threshold | [orion/pipelines/lstm_dynamic_threshold](orion/pipelines/verified/lstm_dynamic_threshold) | LSTM based pipeline inspired by the [Detecting Spacecraft Anomalies Using LSTMs and Nonparametric Dynamic Thresholding paper](https://arxiv.org/abs/1802.04431) |
| Dummy | [orion/pipelines/dummy](orion/pipelines/sandox/dummy) | Dummy pipeline to showcase the input and output format and the usage of sample primitives |
| TadGAN | [orion/pipelines/tadgan](orion/pipelines/sandbox/tadgan) | GAN based pipeline with reconstruction based errors |
| Azure | [orion/pipelines/azure](orion/pipelines/sandbox/azure) | Azure API for [Anomaly Detector](https://azure.microsoft.com/en-us/services/cognitive-services/anomaly-detector/)

# Install

## Requirements

### Python

**Orion** has been developed and runs on [Python 3.6](https://www.python.org/downloads/release/python-360/).

Also, although it is not strictly required, the usage of a [virtualenv](https://virtualenv.pypa.io/en/latest/)
is highly recommended in order to avoid interfering with other software installed in the system
where you are trying to run **Orion**.

### MongoDB

In order to be fully operational, **Orion** requires having access to a
[MongoDB](https://www.mongodb.com/) database running version **3.6** or higher.

## Install with pip

The easiest and recommended way to install **Orion** is using [pip](https://pip.pypa.io/en/stable/):

```bash
pip install orion-ml
```

This will pull and install the latest stable release from [PyPi](https://pypi.org/).

If you want to install from source or contribute to the project please read the
[Contributing Guide](https://sdv-dev.github.io/Copulas/contributing.html#get-started).

## Docker

Even thought it's not mandatory to use it, **Orion** comes with the possibility to be
distributed and run as a docker image, making its usage in offline systems easier.

For more details please read the [Docker Usage Documentation](DOCKER.md).

# Quickstart

In the following steps we will show a short guide about how to run one of the **Orion Pipelines**
on one of the signals from the **Demo Dataset**.

## 1. Load the data

In the first step we will load the **S-1** signal from the **Demo Dataset**.

We will do so in two parts, train and test, as we will use the first part to fit the
pipeline and the second one to evaluate its performance.

To do so, we need to import the `orion.data.load_signal` function and call it twice passing
the `'S-1-train'` and `'S-1-test'` names.

```python3
from orion.data import load_signal

train_data = load_signal('S-1-train')
test_data = load_signal('S-1-test')
```

The output will be a table in the format described above:

```
    timestamp     value
0  1222819200 -0.366359
1  1222840800 -0.394108
2  1222862400  0.403625
3  1222884000 -0.362759
4  1222905600 -0.370746
```

## 2. Detect anomalies using Orion

Once we have the data, let us try to use an Orion pipeline to analyze it and search for anomalies.

In order to do so, we will have to create an instance of the `orion.Orion` class.

```python3
from orion import Orion

orion = Orion()
```

Optionally, we might want to select a pipeline other than the default one or alter the
hyperparameters by the underlying MLBlocks pipeline.

For example, let's select the `lstm_dynamic_threshold` pipeline and reduce the number of
training epochs and increase the verbosity if the LSTM primitive that it uses.

```python3
hyperparameters = {
    'keras.Sequential.LSTMTimeSeriesRegressor#1': {
        'epochs': 5,
        'verbose': True
    }
}
orion = Orion(
    pipeline='lstm_dynamic_threshold',
    hyperparameters=hyperparameters
)
```

Once we the pipeline is ready, we can proceed to fit it to our data:

```python3
orion.fit(train_data)
```

Once it is fitted, we are ready to use it to detect anomalies in our data:

```python3
anomalies = orion.detect(test_data)
```

> :warning: Depending on your system and the exact versions that you might have installed some *WARNINGS* may be printed. These can be safely ignored as they do not interfere with the proper behavior of the pipeline.

The output of the previous command will be a ``pandas.DataFrame`` containing a table in the
Output format described above:

```
        start         end     score
0  1394323200  1399701600  0.673494
```

## 3. Evaluate the performance of your pipeline

In this next step we will load some already known anomalous intervals and evaluate how
good our anomaly detection was by comparing those with our detected intervals.

For this, we will first load the known anomalies for the signal that we are using:

```python3
from orion.data import load_anomalies

ground_truth = load_anomalies('S-1')
```

The output will be a table in the same format as the `anomalies` one.

```
        start         end
0  1392768000  1402423200
```

Afterwards, we can call the `Orion.evaluate` method, passing both the test data
and the ground truth:

```python3
scores = orion.evaluate(test_data, ground_truth)
```

The output will be a ``pandas.Series`` containing a collection of scores indicating
how the predictions were:

```
accuracy     0.988131
f1           0.892193
recall       0.805369
precision    1.000000
dtype: float64
```

# Database

**Orion** comes ready to use a MongoDB Database to easily register and explore:

* Multiple Datasets based on signals from one or more satellites.
* Multiple Pipelines, including historical Pipeline versions.
* Pipeline executions on the registered Datasets, including any environment details required to
  later on reproduce the results.
* Pipeline execution results and detected events.
* Comments about the detected events.

This, among other things, allows:

* Providing visibility about the system usage.
* Keeping track of the evolution of the registered pipelines and their performance over multiple datasets.
* Visualizing and browsing the detected events by the pipelines using a web application.
* Collecting comments from multiple domain experts about the detected events to be able to later
  on curate the pipelines based on their knowledge.
* Reproducing previous executions in identical environments to replicate the obtained results.
* Detecting and keeping a history of system failures for later investigation.

The complete **Database schema and usage instructions** can be found in the
[database documentation](DATABASE.md)
