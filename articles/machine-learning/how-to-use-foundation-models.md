> [!IMPORTANT]
> Items marked (preview) in this article are currently in public preview.
> The preview version is provided without a service level agreement, and it's not recommended for production workloads. Certain features might not be supported or might have constrained capabilities.
> For more information, see [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

# How to use Foundation Models in AzureML (preview)


### How to access foundation models in AzureML
'Model catalog' (preview) provides a catalog view of all models that you have access to via system registries. You can view the complete list of suported foundation models in the [Model catalog](https://ml.azure.com/model/catalog), under the 'azureml' registry.
![image](./model_catalog.png)

You can filter the list of models in the Model catalog by Task, or by license. Clicking on a specific model name will take you to the model card for that model, which lists detailed information about that specific model. For e.g. -
![image](./model_card.png)


	* The 'task' tag on each model calls out the inferencing task that this pre-trained model can be used for. 
	* The 'finetuning-tasks' property lists the tasks that this model can be fine tuned for. 
	* The 'license' property calls out the licensing info NOTE: Models from Hugging Face are subject to third party license terms available on the Hugging Face model details page. It is your responsibility to comply with the model's license terms.
	* For more details about any given Hugging Face <modelID>, please view the model card at https://huggingface.co/<modelID>



### How to finetune foundation models using your own training data
In order to improve model performance in your workload, you might want to fine tune a foundation model using your own training data. You can easily finetune these foundation models using either code based notebook samples or by using the Finetune UI wizard linked from the model card.

#### Finetuning using notebook samples
Currently, AzureML supports finetuning models for the following language tasks -

* Text classification (Both single label and multi-label)
* Token classification
* Question answering
* Summarization
* Translation

To enable users to quickly get started with fine tuning, we have published samples (both Python notebooks as well as CLI examples) for each task in this git repo at [https://github.com/Azure/azureml-foundation-models/tree/main/finetune/sample_pipelines](https://github.com/Azure/azureml-foundation-models/tree/main/finetune/sample_pipelines) 
		
#### Finetuning using the UI wizard
You can invoke the Finetune UI wizard by clicking on the 'Finetune' button on the model card for any foundtaion model. 

<b>Finetuning Task Settings</b>
![image](./finetune_quick_wizard.png)

	* Finetuning task <br> Every pre-trained model from the model catalog can be finetuned for a specific set of tasks (e.g. Text classification, Token classification, Question answering, etc). Select the task you would like to use from the drop down.
	* Training Data <br> Pass in the training data you would like to use to finetune your model. You can choose to either upload a local file (in JSONL, CSV or TSV format) or select an existing regsistered dataset from your workspace. Once you've selected the dataset, you will need to map the data columns based on the schema needed for the task. For e.g. map the column names that correspond to the 'sentence' and 'label' keys 
	* Validation data <br> Pass in the data you would like to use to validate your model. Selecting 'Automatic' will reserve an automatic split of training data for validation
	* Test data <br> Pass in the test data you would like to use to evaluate your model. Selecting 'Automatic' will reserve an automatic split of training data for test. Selecting 'None' will result in no evaluation being run on the fine-tuned model
	* Compute <br> Provide the AzureML Compute cluster you would like to use for finetuning the model. Fine tuning needs to run on GPU compute. We recommend using compute SKUs with A100 / V100 GPUs for this. Please ensure that you have sufficient compute quota for the compute SKUs you wish to use.


Finetuning Task Parameters
![image](./finetune_task_parameters.png)
You can configure parameters such as learning rate, epochs, batch size, etc via Advanced Settings. Each of these settings have default values, but can be customized if needed.

Clicking on 'Finish' in the Finetune Wizard will submit your finetuning job. Once the job completes, you can view evaluation metrics for the finetuned model. You can then go ahead and register the finetuned model output by the finetuning job and deploy this model to an endpoint for inferencing.

### Evaluating foundation models using your own test data
You can evaluate a foundation model against your test dataset, by using either code based notebook samples or by using the Evaluate UI wizard linked from the model card.

### Deploying foundation models to endpoints for inferencing

### Importing foundation models 
