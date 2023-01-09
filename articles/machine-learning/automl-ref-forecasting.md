---
title: 'CLI (v2) command job YAML schema'
titleSuffix: Azure Machine Learning
description: Reference documentation for the CLI (v2) command job YAML schema.
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

# CLI (v2) command job YAML schema

[!INCLUDE [cli v2](../../includes/machine-learning-cli-v2.md)]

The source JSON schema can be found at https://azuremlschemas.azureedge.net/latest/autoMLForecastingJob.schema.json



[!INCLUDE [schema note](../../includes/machine-learning-preview-old-json-schema-note.md)]

## YAML syntax
<br>

| Key | Type | Description | Allowed values | Default value |
| --- | ---- | ----------- | -------------- | ------------- |
| `$schema` | string | Represents the location/url to load the YAML schema.<br>If the user uses the Azure Machine Learning VS Code extension to author the YAML file, including `$schema` at the top of the file enables the user to invoke schema and resource completions. | | |
| `compute` | string | **Required.** <br>Represents the name of the AML compute infrastructure to execute the job on. <br>This compute can be either a reference to an existing compute machine in the workspace <br>*Note:* jobs in pipeline don't support 'local' as `compute`. The 'local' here means that compute instance created in user's AzureML Studio workspace. | 1. pattern `[^azureml:<compute_name>]` to use existing compute,<br>2.`'local'` to use local execution | `'local'` |
| `limits` | object | Represents a dictionary object consisting of limit configurations of the automl tabular job.<br>The key is name for the limit within the context of the job and the value is limit value. See [limits](#limits) to find out the properties of this object.|  |  |
| `name` | string |  Represents the name of the submitted AutoML job.<br>It must be unique across all jobs in the workspace. If not specified, Azure ML will autogenerate a GUID for the name. | | |
| `description` | string | Represents the description of the AutoML job. | | |
| `display_name` | string | The name of the job that user want to display in the studio UI. It can be non-unique within the workspace. If it is omitted, Azure ML will autogenerate a human-readable adjective-noun identifier for the display name. | | |
| `experiment_name` | string | Represents the name of the experiment.<br>Experiments are records of your ML training jobs on Azure. Experiments contains the results of your runs, along with logs, charts, and graphs. Each job's run record will be organized under the corresponding experiment in the studio's "Experiments" tab. | | Name of the working directory in which it was created|
| `environment_variables` | object | Represents a dictionary object of environment variables to set on the process where the command is being executed. |  |  |
| `outputs` | object | Represents a dictionary of output configurations of the job. The key is a name for the output within the context of the job and the value is the output configuration. See [job output](#output) to find out properties of this object.|  |  |  
| `log_files` | object | Represents a dictionary object containing logs of AutoML job execution | | |
| `log_verbosity` | string | Represnts the level of log verbosity for writing to the log file.<br>The acceptable values are defined in the Python [logging library](https://docs.python.org/3/library/logging.html).| `'not_set'`, `'debug'`, `'info'`, `'warning'`, `'error'`, `'critical'` | `'info'` |
| `type` | const | **Required.** <br>Represents the type of job. | `automl` | `automl` |
| `task` | const | **Required.** <br>Represents the type of AutoML task to execute.| `forecasting` | `forecasting` |
| `target_column_name` | string |  **Required.** <br>Represents the name of the column to be forecasted. The AutoMl job raises an error if not specified.|  |  |
| `featurization` | object | Represents a dictionary object defining the configuration of custom featurization. Incase it is not created, the AutoML config will apply auto featurization. See [featurization](#featurization) to see the properties of this object. |  |  |
| `forecasting` | object | Represents a dictionary object defining the settings of forecasting job. See [forecasting](#forecasting) to find out the properties of this object.|  |  |
| `n_cross_validations` | string or integer | Represents the number of cross validations to perform during model/pipeline selection if `validation_data` is not specified.<br>In case both `validation_data` and this parameter is not provided or set to `None`, then automl job set it to `auto` by default. In case `distributed_featurization` is enabled and `validation_data` is not specified, then it is set to 2 by default.  | `'auto'`, [int] | `None` |
| `primary_metric` | string | Represents the metric that AutoML will optimize for Time Series Forecasting model selection.<br>If `allowed_training_algorithms` has 'tcn_forecaster' to use for training, then automl only supports  in 'normalized_root_mean_squared_error' and 'normalized_mean_absolute_error' to be used as primary_metric.| `"spearman_correlation"`, `"normalized_root_mean_squared_error"`, `"r2_score"` `"normalized_mean_absolute_error"`| `"normalized_root_mean_squared_error"` |
| `training` | object | Represents a dictionary object defining the configuration that will be used in model training. |  |  |
| `training_data` | object | **Required**<br>Represents a dictionary object containing the MLTable configuration defining training data to be used in as input for model training. This data is a subset of data and should be comprised of both independent features/columns and target feature/column. The user can use a registered MLTable in the workspace using the format ':' (e.g Input(mltable='my_mltable:1')) OR can use a local file or folder as a MLTable(e.g Input(mltable=MLTable(local_path="./data")). This object must be provided. If target feature is not present in source file, then automl will throw an error. Please check [training or validation or test data](#training-data-or-validation-data-test-data) to find out the properties of this object. | | |
| `validation_data` | object | Represents a dictionary object containing the MLTable configuration defining validation data to be used within automl experiment for cross validation.It should be comprised of both independent features/columns and target feature/column if this object is provided. Please note that samples in training data and validation data can not overlap in a fold. <br>See [training or validation or test data](#training-data-or-validation-data-test-data) to find out the properties of this object. In case this object is not defined, then automl will use `n_cross_validations` to split validation data from training data defined in `training_data` object.| | |
| `test_data` | object | Represents a dictionary object containing the MLTable configuration defining test data to be used in test run for predictions in using best model and will evaluate model using defined metrics. It should be comprised of only independent features used in training data (without target feature) if this object is provided. <br>Please check [training or validation or test data](#training-data-or-validation-data-test-data) to find out the properties of this object. If it is not provided, then automl uses other built in methods to suggest best model to use for inferencing. | | |

<br>
<br>

### limits

| Key | Type | Description | Allowed values | Default value |
| --- | ---- | ----------- | -------------- | ------------- |
| `enable_early_termination` | boolean | Represents whether to enable of experiment termination if the loss score does not improve after 'x' number of iterations.<br>In a AutoML job, no early stopping is applied on first 20 iterations. The early stopping window starts only after first 20 iterations. | `true`, `false` | `true` |
| `max_concurrent_trials` | integer | Represents the maximum number of trials (children jobs) that would be executed in parallel. It is highly recommended to set the number of concurrent runs to the number of nodes in the cluster (aml compute defined in `compute`). | | `1` |
| `max_trials` | integer | Represents the maximum number of trials an automl job can try to run a training algorithm with different combination of hyperparameters. It's default value is set to 1000. If `enable_early_termination` is defined, then the number of trials used to run training algorithms can be smaller.| | `1000` |
| `max_cores_per_trial` | integer | Represents the maximum number of cores per that can be used by each trial. It's default value is set to -1 which means all cores will be used in the process.| | `-1` |
| `timeout_minutes ` | integer | Represnts the maximum amount of time in minutes that the submitted AutoML job can take to run . After this, the job will get terminated. This timeout includes setup, featurization, training runs, ensembling and model explainability (if provided) of all trials. Please note that it won't include the ensembling and model explainability runs at the end of the process if the job fails to get completed within provided `timeout_minutes` since these features are available once all the trials (children jobs) are done. It's default value is set to 360 minutes (6 hours). To specify a timeout less than or equal to 1 hour (60 minutes), the user should make sure dataset's size isn't greater than 10,000,000 (rows times column) or an error results. | | `360` |
| `trial_timeout_minutes ` | integer | Represents the maximum amount of time in minutes that each trial (child job) in the submitted automl job can take run. After this, the child job will get terminated. | | |
| `exit_score` | float | Represents the score to achieve by an experiment. The experiment terminates after this score is reached. If not specified (no criteria), the experiment runs until no further progress is made on the defined `primary metric`. | | |

<br>
<br>

### forecasting

| Key | Type | Description | Allowed values | Default value |
| --- | ---- | ----------- | -------------- | ------------- |
| `time_column_name` | string | **Required** <br>Represents the name of the column in the dataset that corresponds to the time axis of each time series. The input dataset for training, validation or test must contain this column if the task is `forecasting`. If not provided or set to `None`, automl forcasting job will throw an error and terminate the experiment. | | |
| `forecast_horizon` | string or integer | Represents the maximum forecast horizon in units of time-series frequency. These units are based on the inferred time interval of your training data, e.g., monthly, weekly that the forecaster should predict. If it is set to None or `auto`, then it's default value is set to 1 which means 't+1' from the last timestamp t in the input data. | `auto`, [int] | 1 |
| `frequency` | string | Represents the frequency at which the forecast generation is desired, for example daily, weekly, yearly, etc. <br>If it is not specified or set to None, then it's default value is inferred from the dataset time index. The user can set it's value greater than dataset's inferred frequency, but not less than it. For example, if dataset's frequency is daily, it can take values like daily, weekly, monthly, but not hourly as hourly is less than daily(24 hours).<br>Please refer to [pandas documentation](https://pandas.pydata.org/pandas-docs/stable/user_guide/timeseries.html#dateoffset-objects) for more information.|  | `None` |
| `time_series_id_column_names` | string or list(strings) | Represents the names of columns in the data to be used to group data into multiple time series. If time_series_id_column_names is not defined or set to None, the data set is assumed to be one time series. |  | `None` |
| `feature_lags` | string | Represents if user wants to generate lags automatically for the provided numeric features. The default is set to `auto`.| `'auto'`, `None` | `None` |
| `country_or_region_for_holidays` | string | Represents a country or region to be used to generate holiday features. These should be represented in ISO 3166 two-letter country/region codes, for example 'US' or 'GB'. The list of the ISO codes can be found here : https://en.wikipedia.org/wiki/List_of_ISO_3166_country_codes|  | `None` |
| `cv_step_size` | string or integer | Represents the number of periods between the origin_time of one CV fold and the next fold. For example, if it is set to 3 for daily data, the origin time for each fold will be three days apart. If it set to None or not specified, then it's set to `auto` by default. If it is of integer type, minimum value it can take is 1 else it will raise error. | `auto`, [int] | `auto` |
| `seasonality` | string or integer | Represents time series seasonality as an integer multiple of the series frequency. If seasonality is not specified, it's value is set to `'auto'` which means it will be inferred automatically by AutoML. If this parameter is not set to `None`, the AutoML will assume time series as non-seasonal which is equivalent to setting it as integer value 1. | `'auto'`, [int] | `auto` | 
| `short_series_handling_config` | string | Represents how AutoML should handle short time series if specified. It takes following values: <br><ul><li>`'auto'` : short series will be padded if there are no long series,otherwise short series will be dropped.</li><li>`'pad'`: all the short series will be padded with zeros.</li><li>`'drop'`: all the short series will be dropped.</li><li> `None`:the short series will not be modified.</li><ul>| `'auto'`, `'pad'`, `'drop'`, `None` | `None` |
| `target_aggregate_function` | string | Represents the aggregate function to be used to aggregate the target column in time series and generate the forecasts at specified frequency (defined in `freq`). If this parameter is set, but the `freq` parameter is not set, then an error is raised. It it omitted or set to None, then no aggregation will be applied.| `'sum'`, `'max'`, `'min'`, `'mean'` | `auto` |
| `target_lags` | string or integer or list(integer) | Represents the number of past/historical periods to use to lag from the target values based on the dataset frequency. By default, this parameter is turned off. The `'auto'` setting allows system to use automatic heuristic based lag. <br>This lag property should be used when the relationship between the independent variables and dependent variable do not match up or correlate by default. For example, when trying to forecast demand for a product, the demand in any month may depend on the price of specific commodities 3 months prior. In this example, you may want to lag the target (demand) negatively by 3 months so that the model is training on the correct relationship. For more information, see [Auto-train a time-series forecast model](https://docs.microsoft.com/azure/machine-learning/how-to-auto-train-forecast).| `'auto'`, [int] | `None` |
| `target_rolling_window_size` | string or integer | Represents the number of past observations to use for creating a rolling window average of the target column. When forecasting, this parameter represents n historical periods to use to generate forecasted values, <= training set size. If omitted, n is the full training set size. Specify this parameter when you only want to consider a certain amount of history when training the model. | `'auto'`, integer, `None` | `None` |
| `use_stl` | string | Represents component to generate by applying STL decomposition on time series.If not provided or set to None, no time series component will be generated.<br>use_stl can take two values: <br>`'season'` : to generate season component . <br>`'season_trend'` : to generate both season and trend components. | `'season'`, `'seasontrend'` | `None` |

<br>
<br>

### training or validation or test data

| Key | Type | Description | Allowed values | Default value |
| --- | ---- | ----------- | -------------- | ------------- |
| `datastore` | string | Represents the name of the datastore where data is uploaded by user. | | |
| `path` | string | Represnts the path from where data should be loaded. It can be a `file` path, `folder` path or `pattern` for paths. <br>`pattern` specifies a search pattern to allow globbing(`*` and `**`) of files and folders containing data. Supported URI types are `azureml`, `https`, `wasbs`, `abfss`, and `adl`. For more information, see [Core yaml syntax](https://learn.microsoft.com/en-us/azure/machine-learning/reference-yaml-core-syntax) to understand how to use the `azureml://` URI format. URI of the location of the artifact file. If this URI doesn't have a scheme (for example, http:, azureml: etc.), then it's considered a local reference and the file it points to is uploaded to the default workspace blob-storage as the entity is created.  | | |
| `type` | const | In order to generate computer vision models, the user needs to bring labeled image data as input for model training in the form of an MLTable. | `mltable` | `mltable`|


<br>
<br>

### training

| Key | Type | Description | Allowed values | Default value |
| --- | ---- | ----------- | -------------- | ------------- |
| `allowed_training_algorithms` | list(string) | Represents a list of Time Series Forecasting algorithms to try out as base model for model training in an experiment. If it is omitted or set to None, then all supported algorithms are used during experiment, except algorithms specified in `blocked_training_algorithms`.| `'auto_arima'`, `'prophet'`, `'naive'`,`'seasonal_naive'`, `'average'`, `'seasonal_average'`, `'exponential_smoothing'`, `'arimax'`, `'tcn_forecaster'`, `'elastic_net'`, `'gradient_boosting'`, `'decision_tree'`, `'knn'`, `'lasso_lars'`, `'sgd'`, `'random_forest'`, `'extreme_random_trees'`, `'light_gbm'`, `'xg_boost_regressor'` | `None` |
| `blocked_training_algorithms` | list(string) | Represents a list of Time Series Forecasting algorithms to not run as base model while model training in an experiment. If it is omitted or set to None, then all supported algorithms are used during model training. | `'auto_arima'`, `'prophet'`, `'naive'`, `'seasonal_naive'`, `'average'`, `'seasonal_average'`, `'exponential_smoothing'`, `'arimax'`,`'tcn_forecaster'`, `'elastic_net'`, `'gradient_boosting'`, `'decision_tree'`, `'knn'`, `'lasso_lars'`, `'sgd'`, `'random_forest'`, `'extreme_random_trees'`, `'light_gbm'`, `'xg_boost_regressor'` | `None` |
| `enable_dnn_training` | boolean | Represents a flag to turn on or off the inclusion of DNN based models to try out during model selection.| `True`, `False` | `False` |
| `enable_model_explainability` | boolean |  Represents a flag to turn on model explainability like feature importance, of best model evaluated by AutoML system. | `True`, `False` | `False` |
| `enable_vote_ensemble` | boolean | Represents a flag to enable or disable the ensembling of some base models using Voting algorithm. For more information about ensembles, see [Ensemble configuration](https://docs.microsoft.com/azure/machine-learning/how-to-configure-auto-train#ensemble). | `true`, `false` | `true` |
| `enable_stack_ensemble` | boolean | Represents a flag to enable or disable ensembling of some base models using Stacking algorithm. In forecasting tasks, this flag is turned off by default, to avoid risks of overfitting due to small training set used in fitting the meta learner. For more information about ensembles, see [Ensemble configuration](https://docs.microsoft.com/azure/machine-learning/how-to-configure-auto-train#ensemble). | `true`, `false` | `false` |

<br>
<br>

### featurization

| Key | Type | Description | Allowed values | Default value |
| --- | ---- | ----------- | -------------- | ------------- |
| `mode` | string | Represents the featurization mode to be used by AutoML job.<br>Setting it to : <ul><li>`'auto'` indicates whether featurization step should be done automatically</li><li>`'off'` indicates no featurization<li>`'custom'` indicates whether customized featurization should be used.</li></ul> Note: If the input data is sparse, featurization cannot be turned on. | `'auto'`, `'off'`, `'custom'` | `None` |
| `blocked_transformers` | list(string) | Represents a list of transformer names to be blocked during featurization step by AutoML, if featurization `mode` is set to 'custom'. | `'text_target_encoder'`, `'one_hot_encoder'`, `'cat_target_encoder'`, `'tf_idf'`, `'wo_e_target_encoder'`, `'label_encoder'`, `'word_embedding'`, `'naive_bayes'`, `'count_vectorizer'`, `'hash_one_hot_encoder'` | `None` |
| `column_name_and_types` | object | Represents a dictionary object consisting of column names as dict key and feature types used to update column purpose as associated value, if featurization `mode` is set to 'custom'.|  |  |
| `transformer_params` | object | A nested dictionary object consisting of transformer name as key and corresponding customization parameters on dataset columns for featurization, if featurization `mode` is set to 'custom'.<br>The forecasting only supports `imputer` transformer for customization.<br>Please check out [column_transformers](#column_transformers) to find out how to create customization parameters. |  | `None` |

<br>
<br>

### column_transformers

| Key | Type | Description | Allowed values | Default value |
| --- | ---- | ----------- | -------------- | ------------- |
| `fields` | list(string) | Represent a list of column names on which provided `transformer_params` should be applied.|  |  |
| `parameters` | object | Represents a dictionary object consisting of 'strategy' as key and value as imputation strategy.<br> More details on how it can be provided, is provided in examples [here](#quick-links-for-further-reference). |  |  |

<br>
<br>

### Job outputs

| Key | Type | Description | Allowed values | Default value |
| --- | ---- | ----------- | -------------- | ------------- |
| `type` | string | Represents the type of job output. For the default `uri_folder` type, the output will correspond to a folder. | `uri_folder` , `mlflow_model`, `custom_model`| `uri_folder` |
| `mode` | string | Represents the mode of how output file(s) will get delivered to the destination storage. For read-write mount mode (`rw_mount`) the output directory will be a mounted directory. For upload mode the file(s) written will get uploaded at the end of the job. | `rw_mount`, `upload` | `rw_mount` |

<br>
<br>

### How to run AutoML forecasting job via CLI
```
az ml job create --file [YOUR_CLI_YAML_FILE] --workspace-name [YOUR_AZURE_WORKSPACE] --resource-group [YOUR_AZURE_RESOURCE_GROUP] --subscription [YOUR_AZURE_SUBSCRIPTION]
```

### Quick links for further reference:
1. [Install and use the CLI (v2)](how-to-configure-cli.md)
2. [How to run AutoML job via CLI]()
2. [How to auto train forecasts](https://learn.microsoft.com/en-us/azure/machine-learning/how-to-auto-train-forecast)
3. CLI Forecasting examples:<br><ul><li>[Orange Juice Sale Forecasting](https://github.com/Azure/azureml-examples/tree/main/cli/jobs/automl-standalone-jobs/cli-automl-forecasting-orange-juice-sales)</li><li>[Energy Demand Forecasting](https://github.com/Azure/azureml-examples/tree/main/cli/jobs/automl-standalone-jobs/cli-automl-forecasting-task-energy-demand)</li><li>[Bike Share Demand Forecasting](https://github.com/Azure/azureml-examples/tree/main/cli/jobs/automl-standalone-jobs/cli-automl-forecasting-bike-share)</li><li>[Github Daily Active Users Forecast](https://github.com/Azure/azureml-examples/tree/main/cli/jobs/automl-standalone-jobs/cli-automl-forecasting-task-github-dau)
