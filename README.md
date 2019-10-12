# What happens when ML pipeline meets Hydra?

[Hydra](https://github.com/facebookresearch/hydra) is a handy and powerful tool that can dramatically reduce our boilerplate codes and combine dynamically various configurations. I started out with the idea that `Hydra` could be used in **ML Pipeline** as well, and this is a Python app in the form of a template that I quickly implemented. Feedback is always welcome.

## Command Architecture
This app has a two-level command architecture. ([c](https://github.com/withsmilo/When-ML-pipeline-meets-Hydra/tree/master/src/when_ml_pipeline_meets_hydra/config/c)) The command line arguments for executing each command are as follows:

```
├── preprocessing
│   ├── foo       -> c=preprocessing c/preprocessing_sub=foo
│   └── bar       -> c=preprocessing c/preprocessing_sub=bar
├── modeling
│   ├── foo       -> c=modeling c/modeling_sub=foo
│   └── bar       -> c=modeling c/modeling_sub=bar
├── deployment
│   ├── foo       -> c=deployment c/deployment_sub=foo
│   └── bar       -> c=deployment c/deployment_sub=foo
└── help          -> c=help
```

## Prepared Configuration
Here are the configurations prepared for this app. ([preprocessing](https://github.com/withsmilo/When-ML-pipeline-meets-Hydra/tree/master/src/when_ml_pipeline_meets_hydra/config/preprocessing), [modeling](https://github.com/withsmilo/When-ML-pipeline-meets-Hydra/tree/master/src/when_ml_pipeline_meets_hydra/config/modeling/model), [deployment](https://github.com/withsmilo/When-ML-pipeline-meets-Hydra/tree/master/src/when_ml_pipeline_meets_hydra/config/deployment))  The command line arguments for using each configuration are as follows:

```
├── preprocessing
│   ├── dataset
│   │   ├── dataset_1.yaml  -> preprocessing/dataset=dataset_1
│   │   └── dataset_2.yaml  -> preprocessing/dataset=dataset_2
│   └── param
│       ├── param_1.yaml    -> preprocessing/param=param_1
│       └── param_2.yaml    -> preprocessing/param=param_2
├── modeling
│   ├── model
│   │   ├── model_1.yaml    -> modeling/model=model_1
│   │   └── model_1.yaml    -> modeling/model=model_2
│   └── param
│       ├── param_1.yaml    -> modeling/param=param_1
│       └── param_2.yaml    -> modeling/param=param_2
└── deployment
    └── cluster
        ├── cluster_1.yaml  -> deployment/cluster=cluster_1
        └── cluster_1.yaml  -> deployment/cluster=cluster_2
```

## How to Install

```
# Create a new Anaconda environment if needed.
$ conda create --name when_ml_pipeline_meets_hydra python=3.6 -y
$ conda activate when_ml_pipeline_meets_hydra

# Clone this repo.
$ git clone https://github.com/withsmilo/When-ML-pipeline-meets-Hydra.git
$ cd When-ML-pipeline-meets-Hydra

# Install this app.
$ python setup.py develop
$ when_ml_pipeline_meets_hydra --help
```

## ML Pipeline Test
First of all, I will construct a new ML pipeline dynamically using all `*_1.yaml` configurations and executing the same `foo` subcommand per each step. The command you need is simple and structured.

```
$ when_ml_pipeline_meets_hydra \
  preprocessing/dataset=dataset_1 \
  preprocessing/param=param_1 \
  modeling/model=model_1 \
  modeling/param=param_1 \
  deployment/cluster=cluster_1 \
  c/preprocessing_sub=foo \
  c/modeling_sub=foo \
  c/deployment_sub=foo \
  c=preprocessing,modeling,deployment \
  --multirun
```

```
[2019-10-13 02:05:17,597] - Launching 3 jobs locally
[2019-10-13 02:05:17,597] - Sweep output dir : .multirun/2019-10-13
[2019-10-13 02:05:17,597] - 	#0 : preprocessing/dataset=dataset_1.yaml preprocessing/param=param_1.yaml modeling/model=model_1.yaml modeling/param=param_1.yaml deployment/cluster=cluster_1.yaml c/preprocessing_sub=foo c/modeling_sub=foo c/deployment_sub=foo c=preprocessing
========== Run preprocessing's 'foo' subcommand ==========
dataset_info:
{
  "name": "dataset_1",
  "path": "/path/of/dataset/1"
}
param_info:
{
  "name": "param_1",
  "output_path": "/path/of/output/path/1",
  "key_1_1": "value_1_1",
  "key_1_2": "value_1_2"
}
Do something here!
[2019-10-13 02:05:17,741] - 	#1 : preprocessing/dataset=dataset_1.yaml preprocessing/param=param_1.yaml modeling/model=model_1.yaml modeling/param=param_1.yaml deployment/cluster=cluster_1.yaml c/preprocessing_sub=foo c/modeling_sub=foo c/deployment_sub=foo c=modeling
========== Run modeling's 'foo' subcommand ==========
model_info:
{
  "name": "model_1",
  "input_path": "/path/of/input/path/1",
  "output_path": "/path/of/output/path/1"
}
param_info:
{
  "name": "param_1",
  "hyperparam_key_1_1": "hyperparam_value_1_1",
  "hyperparam_key_1_2": "hyperparam_value_1_2"
}
Do something here!
[2019-10-13 02:05:17,878] - 	#2 : preprocessing/dataset=dataset_1.yaml preprocessing/param=param_1.yaml modeling/model=model_1.yaml modeling/param=param_1.yaml deployment/cluster=cluster_1.yaml c/preprocessing_sub=foo c/modeling_sub=foo c/deployment_sub=foo c=deployment
========== Run deployment's 'foo' subcommand ==========
cluster_info:
{
  "name": "cluster_1",
  "url": "https://cluster/1/url",
  "id": "user_1",
  "pw": "pw_1"
}
Do something here!
```

After then, if you'd like to deploy a model that has only changed the hyperparameter settings to another cluster, you can simply change `modeling/param` to `param_2.yaml` and `deployment/cluster` to `cluster_2.yaml` before executing your command. That's it!

```
$ when_ml_pipeline_meets_hydra \
  preprocessing/dataset=dataset_1.yaml \
  preprocessing/param=param_1.yaml \
  modeling/model=model_1.yaml \
  modeling/param=param_2.yaml \
  deployment/cluster=cluster_2.yaml \
  c/preprocessing_sub=foo \
  c/modeling_sub=foo \
  c/deployment_sub=foo \
  c=preprocessing,modeling,deployment \
  --multirun
```

```
[2019-10-13 02:11:17,319] - Launching 3 jobs locally
[2019-10-13 02:11:17,319] - Sweep output dir : .multirun/2019-10-13
[2019-10-13 02:11:17,319] - 	#0 : preprocessing/dataset=dataset_1.yaml preprocessing/param=param_1.yaml modeling/model=model_1.yaml modeling/param=param_2.yaml deployment/cluster=cluster_2.yaml c/preprocessing_sub=foo c/modeling_sub=foo c/deployment_sub=foo c=preprocessing
========== Run preprocessing's 'foo' subcommand ==========
dataset_info:
{
  "name": "dataset_1",
  "path": "/path/of/dataset/1"
}
param_info:
{
  "name": "param_1",
  "output_path": "/path/of/output/path/1",
  "key_1_1": "value_1_1",
  "key_1_2": "value_1_2"
}
Do something here!
[2019-10-13 02:11:17,459] - 	#1 : preprocessing/dataset=dataset_1.yaml preprocessing/param=param_1.yaml modeling/model=model_1.yaml modeling/param=param_2.yaml deployment/cluster=cluster_2.yaml c/preprocessing_sub=foo c/modeling_sub=foo c/deployment_sub=foo c=modeling
========== Run modeling's 'foo' subcommand ==========
model_info:
{
  "name": "model_1",
  "input_path": "/path/of/input/path/1",
  "output_path": "/path/of/output/path/1"
}
param_info:
{
  "name": "param_2",
  "hyperparam_key_2_1": "hyperparam_value_2_1",
  "hyperparam_key_2_2": "hyperparam_value_2_2"
}
Do something here!
[2019-10-13 02:11:17,588] - 	#2 : preprocessing/dataset=dataset_1.yaml preprocessing/param=param_1.yaml modeling/model=model_1.yaml modeling/param=param_2.yaml deployment/cluster=cluster_2.yaml c/preprocessing_sub=foo c/modeling_sub=foo c/deployment_sub=foo c=deployment
========== Run deployment's 'foo' subcommand ==========
cluster_info:
{
  "name": "cluster_2",
  "url": "https://cluster/2/url",
  "id": "user_2",
  "pw": "pw_2"
}
Do something here!
```
As well as this scenario, you can think of various cases.

## Note
This project has been set up using PyScaffold 3.2.2. For details and usage information on PyScaffold see https://pyscaffold.org/.

## License
This app is licensed under [MIT License](https://github.com/withsmilo/When-ML-pipeline-meets-Hydra/blob/master/LICENSE.txt).
