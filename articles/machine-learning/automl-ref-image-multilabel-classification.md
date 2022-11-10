---
title: 'CLI (v2) Automated ML Image Multi-Label Classification job YAML schema'
titleSuffix: Azure Machine Learning
description: Reference documentation for the CLI (v2) Automated ML Image Multi-Label Classification job YAML schema.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
ms.custom: cliv2, event-tier1-ignite-2022

ms.author: shoja
author: shouryaj
ms.date: 10/11/2022
ms.reviewer: ssalgado
---

# CLI (v2) Automated ML Image Multi-Label Classification job YAML schema

[!INCLUDE [cli v2](../../includes/machine-learning-cli-v2.md)]

The source JSON schema can be found at https://azuremlsdk2.blob.core.windows.net/preview/0.0.1/autoMLImageClassificationMultilabelJob.schema.json.



[!INCLUDE [schema note](../../includes/machine-learning-preview-old-json-schema-note.md)]

## YAML syntax

| Key | Type | Description | Allowed values | Default value |
| --- | ---- | ----------- | -------------- | ------------- |
| `$schema` | string | The YAML schema. If the user uses the Azure Machine Learning VS Code extension to author the YAML file, including `$schema` at the top of the file enables the user to invoke schema and resource completions. | | |
| `type` | const | **Required.** The type of job. | `automl` | `automl` |
| `task` | const | **Required.** The type of AutoML task. | `image_classification_multilabel` | `image_classification_multilabel` |
| `name` | string | Name of the job. Must be unique across all jobs in the workspace. If omitted, Azure ML will autogenerate a GUID for the name. | | |
| `display_name` | string | Display name of the job in the studio UI. Can be non-unique within the workspace. If omitted, Azure ML will autogenerate a human-readable adjective-noun identifier for the display name. | | |
| `experiment_name` | string | Experiment name to organize the job under. Each job's run record will be organized under the corresponding experiment in the studio's "Experiments" tab. If omitted, Azure ML will default it to the name of the working directory where the job was created. | | |
| `description` | string | Description of the job. | | |
| `tags` | object | Dictionary of tags for the job. | | |
| `compute` | string | Name of the compute target to execute the job on. This compute name can be either a reference to an existing compute in the workspace (using the `azureml:<compute_name>` syntax) or `local` to designate local execution. <br> <br>  *Note:* jobs in pipeline don't support `local` as `compute`. * | | `local` |
| `log_verbosity` | number | Different levels of log verbosity. |`not_set`, `debug`, `info`, `warning`, `error`, `critical` | `info` |
| `primary_metric` | string |  The metric that AutoML will optimize for model selection. |`accuracy`, `auc_weighted`, `average_precision_score_weighted`, `norm_macro_recall`, `precision_score_weighted` | `accuracy` |
| `target_column_name` | string |  **Required.** The name of the column to target for predictions. It must always be specified. This parameter is applicable to `training_data` and `validation_data`. | |  |
| `training_data` | object |  **Required.** The data to be used within the job. It should contain both training feature columns and a target column. the parameter training_data must always be provided. For more infomration, see [Training or Validation Data](#training-or-validation-data) to understand how to write this object.| |  |
| `validation_data` | object |  The validation data to be used within the job. It should contain both training features and label column (optionally a sample weights column). If `validation_data` is specified, then `training_data` and `target_column_name` parameters must be specified. For more infomration, see [Training or Validation Data](#training-or-validation-data) to understand how to write this object.| |  |
| `validation_data_size` | float |  What fraction of the data to hold out for validation when user validation data isn't specified. | A value in range (0.0, 1.0) |  |
| `limits` | object | Dictionary of limit configurations of the job. The key is name for the limit within the context of the job and the value is limit value. If the user wants to specify a different mode for the output, provide an object containing the [Limits](#limits). | | |
| `training_parameters` | object | Dictionary containing training parameters for the job. Provide an object that has keys as listed in [Training Parameters](#training-parameters). | | |
| `sweep` | object | Dictionary containing sweep parameters for the job. Provide an object that has keys as listed in [Sweep Parameters](#sweep-parameters). | | |
| `search_space` | object | Dictionary of the hyperparameter search space. The key is the name of the hyperparameter and the value is the parameter expression. The user can find the possible hyperparameters in [Training Parameters](#training-parameters). | | |
| `search_space<hyperparameter>` | object | See [Parameter expressions](https://learn.microsoft.com/en-us/azure/machine-learning/reference-yaml-job-sweep#parameter-expressions) for the set of possible expressions to use. <br>- For discrete hyperparameters, the user should use [`choice` parameter expression](https://learn.microsoft.com/en-us/azure/machine-learning/reference-yaml-job-sweep#choice). <br>- For continuous hyperparameters, the user should use one of the following parameter-expressions according to the distribution that user wants to explore, [`randint`](https://learn.microsoft.com/en-us/azure/machine-learning/reference-yaml-job-sweep#randint), [`qlognormal`, `qnormal`](https://learn.microsoft.com/en-us/azure/machine-learning/reference-yaml-job-sweep#qlognormal-qnormal), [`lohnormal`, `normal`](https://learn.microsoft.com/en-us/azure/machine-learning/reference-yaml-job-sweep#lognormal-normal), [`loguniform`](https://learn.microsoft.com/en-us/azure/machine-learning/reference-yaml-job-sweep#loguniform), [`uniform`](https://learn.microsoft.com/en-us/azure/machine-learning/reference-yaml-job-sweep#uniform).  | | |
| `outputs` | object | Dictionary of output configurations of the job. The key is a name for the output within the context of the job and the value is the output configuration. | | |
| `outputs.best_model` | object | Dictionary of output configurations for best model. For further details, refer to [Best Model Output Configuration](#best-model-output-configuration). | | |


### Limits

| Key | Type | Description | Allowed values |Default value |
| --- | ---- | ----------- | -------------- | ------------ |
| `max_trials` | integer | The maximum total number of different algorithm and parameter combinations (trials) to try during an AutoML job. If user is using `enable_early_termination`, then the number of trials used can be smaller.  | | 1000 |
| `max_concurrent_trials` | integer | Represents the maximum number of trials (children jobs) that would be executed in parallel. | | 1 |
| `timeout_minutes` | integer | Maximum amount of time in minutes that the whole AutoML job can take before the job terminates. This timeout includes setup, featurization and training runs but doesn't include the ensembling and model explainability runs at the end of the process since those actions need to happen once all the trials (children jobs) are done. If not specified, the default job's total timeout is six days (8,640 minutes). To specify a timeout less than or equal to 1 hour (60 minutes), the user should make sure dataset's size isn't greater than 10,000,000 (rows times column) or an error results. | | 8640 |
| `trial_timeout_minutes` | integer |Maximum time in minutes that each trial (child job) can run for before it terminates. If not specified, a value of one month or 43200 minutes is used. | | 43200 |


### Training Or Validation Data

| Key | Type | Description | Allowed values | Default value |
| --- | ---- | ----------- | -------------- | ------------- |
| `description` | string | The detailed information that describes this input data. | | |
| `path` | string | Path can be a `file` path, `folder` path or `pattern` for paths. `pattern` specifies a search pattern to allow globbing(`*` and `**`) of files and folders containing data. Supported URI types are `azureml`, `https`, `wasbs`, `abfss`, and `adl`. For more information, see [Core yaml syntax](https://learn.microsoft.com/en-us/azure/machine-learning/reference-yaml-core-syntax) to understand how to use the `azureml://` URI format. URI of the location of the artifact file. If this URI doesn't have a scheme (for example, http:, azureml: etc.), then it's considered a local reference and the file it points to is uploaded to the default workspace blob-storage as the entity is created.  | | |
| `mode` | string | | |
| `type` | const |  In order to generate computer vision models, the user needs to bring labeled image data as input for model training in the form of an MLTable. | mltable | mltable|

### Training Parameters
This section describes the hyperparameters available specifically for computer vision tasks in automated ML experiments. There are two types of hyperparameters: 
- **Discrete Hyperparameters**: Discrete hyperparameters are specified as a `Choice` among discrete values. `Choice` can be one or more comma-separated values, a `range` object, or any arbitrary `list` object. Advanced discrete hyperparameters can also be specified using a distribution - `QUniform(min_value, max_value)`, `QLogUniform(min_value, max_value)`, `QNormal(mu, sigma)`, `QLogNormal(mu, sigma)`. For further details, refer to this [section](https://learn.microsoft.com/en-us/azure/machine-learning/how-to-tune-hyperparameters#discrete-hyperparameters).
- **Continuous hyperparameters**: Continuous hyperparameters are specified as a distribution over a continuous range of values. Currently supported distributions namely are - `Uniform(min_value, max_value)`, `LogUniform(min_value, max_value)`, `Normal(mu, sigma)`, `LogNormal(mu, sigma)`. For further details, refer to this [section](https://learn.microsoft.com/en-us/azure/machine-learning/how-to-tune-hyperparameters#continuous-hyperparameters).

While many of the hyperparameters exposed are model-agnostic, there are instances where hyperparameters are model-specific or task-specific. Note that the `training_parameters` object can have properties from [Model Agnostic Hyperparameters](#model-agnostic-hyperparameters) and [Image Classification (multi-class and multi-label) Specific Hyperparameters](#image-classification-multi-class-and-multi-label-specific-hyperparameters).

#### Model Agnostic Hyperparameters
The following table describes the hyperparameters that are model agnostic.

| Key | Type | Description | Allowed values | Default value |
| --- | ---- | ----------- | -------------- | ------------- |
|`ams_gradient` | Enable `ams_gradient` when optimizer is `adam` or `adamw`.| 0, 1| 0 |
|`advanced_settings` | | | |
|`beta1` | float | Value of `beta1` when optimizer is `adam` or `adamw`. | A value in the range [0, 1] | 0.9 |
|`beta2` | float | Value of `beta2` when optimizer is `adam` or `adamw`. | A value in the range [0, 1] | 0.999 |
|`checkpoint_frequency`| integer | Frequency to store model checkpoints. | | Checkpoint at epoch with best primary metric on validation.|
|`checkpoint_run_id`| string | The run ID of the experiment that has a pretrained checkpoint for incremental training.| | no default  |
|`distributed`| bool | | 0, 1 | 0 |
| `early_stopping` | integer | Enable early stopping logic during training. |  `0`, `1` | `1` |
| `early_stopping_delay` | integer | Minimum number of epochs or validation evaluations to wait<br>before primary metric improvement is tracked for early stopping. | | 5 |
| `early_stopping_patience` | integer | Minimum number of epochs or validation evaluations with<br>no primary metric improvement before the run is stopped. | | 5 |
|`evaluation_frequency`| integer | Frequency to evaluate validation dataset to get metric scores. | | 1 |
|`enable_onnx_normalization`| bool | | 0, 1 | 0 |
| `gradient_accumulation_step` | integer | Gradient accumulation means to run a configured number of `gradient_accumulation_step` without updating the model weights while accumulating the gradients of those steps, and then using the accumulated gradients to compute the weight updates. | | 1 |
|`layers_to_freeze`| integer | How many layers to freeze for your model. For instance, passing 2 as value for `seresnext` means freezing layer0 and layer1 referring to the below supported model layer info. <br>- ` 'resnet'`: `[('conv1.', 'bn1.'), 'layer1.', 'layer2.', 'layer3.', 'layer4.']`<br>- `'mobilenetv2'`: `['features.0.', 'features.1.', 'features.2.', 'features.3.', 'features.4.', 'features.5.', 'features.6.', 'features.7.', 'features.8.', 'features.9.', 'features.10.', 'features.11.', 'features.12.', 'features.13.', 'features.14.', 'features.15.', 'features.16.', 'features.17.', 'features.18.']`<br>- `'seresnext'`: `['layer0.', 'layer1.', 'layer2.', 'layer3.', 'layer4.']`<br>- `'vit'`: `['patch_embed', 'blocks.0.', 'blocks.1.', 'blocks.2.', 'blocks.3.', 'blocks.4.', 'blocks.5.', 'blocks.6.','blocks.7.', 'blocks.8.', 'blocks.9.', 'blocks.10.', 'blocks.11.']` | | no default  |
| `learning_rate` | float | Initial learning rate. | A value  in the range [0, 1] | 0.035 <br>(except *vit-variants*:<br>`vits16r224`: 0.025<br>`vitb16r224`: 0.025 <br>`vitl16r224`: 0.002) |
| `learning_rate_scheduler` | string | Type of learning rate scheduler. | `none`, `warmup_cosine`, `step`. | `warmup_cosine` |
| `momentum` | float | Value of momentum when optimizer is `sgd`. | A value in the range [0, 1] | 0.9 |
|`nesterov`| integer | Enable `nesterov` when optimizer is `sgd`. | 0, 1 | 1 |
| `number_of_epochs` | integer | Number of training epochs. | | 15 <br> (except `yolov5`: 30) |
| `number_of_workers` | integer | Number of workers. | |  |
| `optimizer` | string | Type of optimizer. | `sgd`, `adam`, `adamw`  | `sgd` |
| `random_seed` | integer | | | |
| `step_lr_gamma` | float | Value of gamma when learning rate scheduler is `step`.| A value in the range [0, 1] | 0.5 |
| `step_lr_step_size` | integer | Value of step size when learning rate scheduler is `step`. | | 5 |
| `training_batch_size` | integer | Training batch size. <br> <br> *Note: The defaults are largest batch size that can be used on 12-GiB GPU memory*. | | 78 <br>(except *vit-variants*: <br> `vits16r224`: 128 <br>`vitb16r224`: 48 <br>`vitl16r224`:10) |
| `validation_batch_size` | integer | Validation batch size.<br> <br> *Note: The defaults are largest batch size that can be used on 12-GiB GPU memory*. | | 78 <br>(except *vit-variants*: <br> `vits16r224`: 128 <br>`vitb16r224`: 48 <br>`vitl16r224`:10) |
| `warmup_cosine_lr_cycles` | float | Value of cosine cycle when learning rate scheduler is `warmup_cosine`. | A value in the range [0, 1] | 0.45 |
| `warmup_cosine_lr_warmup_epochs` | integer | Value of warmup epochs when learning rate scheduler is `warmup_cosine`. | | 2 |
| `weight_decay` | float | Value of weight decay when optimizer is `sgd`, `adam`, or `adamw`. | A value in the range [0, 1] | 1e-4 |

#### Image Classification (Multi-Class and Multi-Label) Specific Hyperparameters

| Key | Type | Description | Allowed values |Default value |
| --- | ---- | ----------- | -------------- | ------------ |
| `model_name` | string | Model name to be used for image classification task at hand. | `mobilenetv2`, `resnet18`, `resnet34`, `resnet50`, `resnet101`, `resnet152`, `resnest50`, `resnest101`, `seresnext`, `vits16r224`, `vitb16r224`, `vitl16r224` | `seresnext` |
| `training_crop_size` | integer | Image crop size that's input to your neural network for train dataset. <br><br> Notes: <br>- `seresnext` doesn't take an arbitrary size. <br>- ViT-variants should have the same `validation_crop_size` and `training_crop_size`. <br>- Training run may get into CUDA OOM if the size is too large.|  | 224|
| `validation_crop_size` | integer | Image crop size that's input to your neural network for validation dataset. <br><br> Notes: <br>- `seresnext` doesn't take an arbitrary size. <br>- ViT-variants should have the same `validation_crop_size` and `training_crop_size`. <br>- Training run may get into CUDA OOM if the size is too large.| | 224 |
| `validation_resize_size` | integer | Image size to which to resize before cropping for validation dataset. <br><br> Notes: <br>- `seresnext` doesn't take an arbitrary size. <br>- Training run may get into CUDA OOM if the size is too large. | | 256 |
| `weighted_loss` | integer | - `0` for no weighted loss. <br>- `1` for weighted loss with sqrt (class_weights). <br>- `2` for weighted loss with class_weights. | 0, 1, 2 | 0 |

### Sweep Parameters
When using AutoML for Images, we can perform a hyperparameter sweep over a defined parameter space to find the optimal model. If hyperparameter values aren't specified, then default values are used for the specified algorithm. 

| Key | Type | Description | Allowed values |Default value |
| --- | ---- | ----------- | -------------- | ------------ |
| `sampling_algorithm` | string | **Required.** When sweeping hyperparameters, the user needs to specify the sampling method to use for sweeping over the defined parameter space. <br> Note: <br> - Currently only random and grid sampling support conditional hyperparameter spaces. | `random`, `grid`, `bayesian` | `grid` |
| `early_termination` | object | You can automatically end poorly performing runs with an early termination policy. Early termination improves computational efficiency, saving compute resources that would have been otherwise spent on less promising configurations. Automated ML for images supports the following early termination policies using the early_termination parameter. If no termination policy is specified, all configurations are run to completion. Supported early termination policy types are `bandit`, `median_stopping`, `truncation_selection`. By default, bandit policy is used. For the details on individual early termination policies, refer to the [Bandit Early Termination Policy](#bandit-early-termination-policy), [Median Stopping Early Termination Policy](#median-stopping-early-termination-policy), [Truncation Selection Early Termination Policy](#truncation-selection-early-termination-policy). |  | |

#### Bandit Early Termination Policy
Bandit Policy is based on slack factor/slack amount and evaluation interval. Bandit policy ends a job when the primary metric isn't within the specified slack factor/slack amount of the most successful job. For further details on Bandit policy, refer to [documentation](https://learn.microsoft.com/en-us/azure/machine-learning/how-to-tune-hyperparameters#bandit-policy).

| Key | Type | Description | Allowed values |Default value |
| --- | ---- | ----------- | -------------- | ------------ |
| `type` | string | **Required.** Type of early termination policy. | `bandit` | `bandit` |
| `slack_factor` | number | The ratio used to calculate the slack or the allowed distance from the best performing trial. **One of `slack_factor` or `slack_amount` is required.** | | |
| `slack_amount` | number | The absolute slack or distance allowed from the best performing trial. **One of `slack_factor` or `slack_amount` is required.** | | |
| `evaluation_interval` | integer |  The frequency for applying the policy. | | |
| `delay_evaluation` | integer | Delays the first policy evaluation for a specified number of intervals. | | |

#### Median Stopping Early Termination Policy
Median Stopping is an early termination policy based on running averages of primary metrics reported by the jobs. For further details on Median Stopping policy, refer to [documentation](https://learn.microsoft.com/en-us/azure/machine-learning/how-to-tune-hyperparameters#median-stopping-policy).

| Key | Type | Description | Allowed values |Default value |
| --- | ---- | ----------- | -------------- | ------------ |
| `type` | string | **Required.** Type of early termination policy. | `median_stopping` | `median_stopping` |
| `evaluation_interval` | integer | The frequency for applying the policy. | | |
| `delay_evaluation` | integer |  Delays the first policy evaluation for a specified number of intervals. | | |

#### Truncation Selection Early Termination Policy
Truncation Selection policy cancels a percentage of lowest performing jobs at each evaluation interval. jobs are compared using the primary metric. For further details on Median Stopping policy, refer to [documentation](https://learn.microsoft.com/en-us/azure/machine-learning/how-to-tune-hyperparameters#truncation-selection-policy).

| Key | Type | Description | Allowed values |Default value |
| --- | ---- | ----------- | -------------- | ------------ |
| `type` | string | **Required.** Type of early termination policy. | `truncation_selection` | `truncation_selection` |
| `truncation_percentage` | integer | **Required.** The percentage of lowest performing jobs to terminate at each evaluation interval. | [1, 99] |  |
| `evaluation_interval` | integer | The frequency for applying the policy. | | |
| `delay_evaluation` | integer |  Delays the first policy evaluation for a specified number of intervals. | | |
| `exclude_finished_jobs` | integer |  Specifies whether to exclude finished jobs when applying the policy. | | |

### Best Model Output Configuration

| Key | Type | Description | Allowed values |Default value |
| --- | ---- | ----------- | -------------- | ------------ |
| `type` | string | **Required.** Type of best model. AutoML allows only mlflow models. | `mlflow_model` | `mlflow_model` |
| `path` | string | **Required.** URI of the location where the model-artifact file(s) are stored. If this URI doesn't have a scheme (for example, http:, azureml: etc.), then it's considered a local reference and the file it points to is uploaded to the default workspace blob-storage as the entity is created. |  |  |
| `storage_uri` | string | The HTTP URL of the Model. Use this URL with `az storage copy -s THIS_URL -d DESTINATION_PATH --recursive` to download the data.  | | |


## Remarks

The `az ml job` command can be used for managing Azure Machine Learning jobs.

## Examples

Examples are available in the [examples GitHub repository](https://github.com/Azure/azureml-examples/tree/main/cli/jobs). Examples relevant to multi-label image classification job are shown below.


## YAML: AutoML image classification job

:::code language="yaml" source="~/azureml-examples-main/cli/jobs/automl-standalone-jobs/cli-automl-image-classification-multilablel-task-fridge-items/cli-automl-image-classification-multilablel-task-fridge-items.yml":::

## YAML: AutoML image classification pipeline job

:::code language="yaml" source="~/azureml-examples-main/cli/jobs/pipelines/automl/image-multilabel-classification-fridge-items-pipeline/pipeline.yml":::


## Next steps

- [Install and use the CLI (v2)](how-to-configure-cli.md)
