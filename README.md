
# PromptWizard 🧙

[![arXiv](https://img.shields.io/badge/arXiv-2405.18369-b31b1b.svg)](https://arxiv.org/abs/2405.18369)

> **PromptWizard: Task-Aware Prompt Optimization Framework**<br>
> Eshaan Agarwal, Joykirat Singh, Vivek Dani, Raghav Magazine, Tanuja Ganu, Akshay Nambi <br>

>**Abstract**: <br>
> Large language models (LLMs) have transformed AI across diverse domains, with prompting being central to their success in guiding model outputs. However, manual prompt engineering is both labor-intensive and domain-specific, necessitating the need for automated solutions. We introduce PromptWizard, a novel, fully automated framework for discrete prompt optimization, utilizing a self-evolving, self-adapting mechanism. Through a feedback-driven critique and synthesis process, PromptWizard achieves an effective balance between exploration and exploitation, iteratively refining both prompt instructions and in-context examples to generate human-readable, task-specific prompts. This guided approach systematically improves prompt quality, resulting in superior performance across 45 tasks. PromptWizard excels even with limited training data, smaller LLMs, and various LLM architectures. Additionally, our cost analysis reveals a substantial reduction in API calls, token usage, and overall cost, demonstrating PromptWizard's efficiency, scalability, and advantages over existing prompt optimization strategies.

## Overview 🌟

### Main Algorithm
<img src="./images/overview.png" >

<div style="display: flex; justify-content:">
 <img src= "./images/iterative_flowchart-1.png" width="430">
 <img src="./images/sequential_flowchart-1.png" width="430">
</div>

## Installation ⬇️

Follow these steps to set up the development environment and install the package:

1) Clone the repository
    ```
    git clone https://github.com/microsoft/PromptWizard
    cd PromptWizard
    ```
2) Create and activate a virtual environment

    - On Windows
    ```
    python -m venv venv
    venv\Scripts\activate
    ```
    - On macOS/Linux:
    ```
    python -m venv venv
    source venv/bin/activate
    ```
3) Install the package in development mode:
    ```
    pip install -e .
    ```


## Quickstart 🏃

- We support [GSM8K](https://huggingface.co/datasets/openai/gsm8k), [SVAMP](https://huggingface.co/datasets/ChilleD/SVAMP), [AQUARAT](https://huggingface.co/datasets/deepmind/aqua_rat) and [Instruction_Induction(BBII)](https://github.com/xqlin98/INSTINCT/tree/main/Induction/experiments/data/instruction_induction/raw) datasets
- Please note that time taken for prompt optimzation is dependent on the dataset. In our experiments for the above mentioned datasets, it took around 20 - 30 minutes on average.
- To run on your custom dataset please jump [here](#run-on-custom-dataset) 

#### Running on GSM8K (AQUARAT/SVAMP)

- Please note that this code requires access to LLMs via API calling, we use AZURE endpoints for this
- Set the AZURE endpoint configurations in [.env](demos/gsm8k/.env) as shown below
```
AZURE_OPENAI_ENDPOINT="XXXXX" 
# Replace with your Azure OpenAI Endpoint

OPENAI_API_VERSION="XXXX"
# Replace with the version of your API

AZURE_OPENAI_CHAT_DEPLOYMENT_NAME="XXXXX"
# Create a deployment for the model and place the deployment name here. 
```
- Follow the steps in [demo.ipynb](demos/gsm8k/demo.ipynb) to download the data, run the prompt optimization and carry out inference.

#### Running on BBII

- BBII has many datasets in it, based on the dataset set the configs [here](demos/bbh/configs/promptopt_config.yaml)
- In configs ```task_description```,```base_instruction``` and ```answer_format``` need to be changed for different datasets in BBII, the rest of the configs remain the same
- A demo is presented in  [demo.ipynb](demos/bbh/demo.ipynb)

## Run on Custom Datasets 🗃️

### Create Custom Dataset
- Our code expects the dataset to be in ```.jsonl``` file format
- Both the train and test set follow the same format
- Every sample in the ```.jsonl``` should have 3 fields :
  1) ```question``` : It should contain the complete question that is to asked to the LLM
  2) ```answer``` : It should contain the ground truth answer which can be verbose or consize


### Run on Custom Dataset

NOTE : Refer to [demos](demos) folder for examples of folders for four datasets. The ```.ipynb``` in each of the folders shows how to run PromptWizard on that particular dataset. A similar procedure can be followed for a new dataset. Below is the explanation of each of the components of the ```.ipynb``` and the dataset specifc folder structure in detail

#### Steps to be followed for custom datasets 

1) Every new dataset needs to have the following 
    - ```configs``` folder to store files for defining optimization hyperparameters and setup configs 
    - ```data``` folder to store ```train.jsonl``` and ```test.jsonl``` as curated [here](#create-custom-dataset) (this is done in the notebooks)
    - ```.env``` file for environment varibles to be used for API calling
    - ```.py/.ipynb``` script to run the code

2) Hyperparameters like number of mutations, refine steps, in-context examples etc. can be changed in [promptopt_config.yaml](demos/gsm8k/configs/promptopt_config.yaml)
    - Set the following : 
        - ```task_description``` : Desciption of the task at hand which will be fed into the prompt
        - ```base_instruction``` : Base intruction in line with the dataset
        - ```answer_format``` : Instruction for specifying the answer format
    - It is crucial to set the ```answer_format``` properly to ensure correct extraction by ```def extract_final_answer()```
    - Refer ```promptopt_config.yaml``` files in folders present [here](demos)  for the descriptions used for AQUARAT, SVAMP and GSM8k. For BBII refer [description.py](demos/bbh/description.py) which has the meta instructions for each of the datasets
3) Create a dataset specific class which inherits ```class DatasetSpecificProcessing``` similar to ```GSM8k(DatasetSpecificProcessing)``` in [demo.ipynb](demos/gsm8k/demo.ipynb) and define the following functions in it
      1) In ```def extract_answer_from_output()``` : This is a dataset specific function, given the ```answer``` from the dataset it should extract and return  a consize form of the answer. Note that based on the dataset it can also simply return the ```answer``` as it is like in case of SVAMP and AQUARAT datasets
      2) ```def extract_final_answer()``` : This is a LLM output specific function, given the verbose answer from the LLM it should extract and return the consize final answer
      3) Define ```def access_answer()``` : This function takes an input the LLM output, then does the following:
         - Extracts the consize answer using ```def extract_final_answer()``` from the LLM output as defined above
         - Evaluates the extracted answer with the ground truth and retuns
            - Extracted answer from LLM output
            - Boolean value indicating if answer is correct or not
         - The evaluation done here is dataset specific, for datasets like GSM8K, SVAMP and AQUARAT which are there final answer as an number we can do a direct match between the numbers generated and the ground truth, while for datasets where the answer is a sentence or paragraph it would be better to do evaluation with llm-as-a-judge, to compare the generated and ground truth paragraph/sentence. An example is available in ```def access_answer()``` in [this](demos/bbh/demo.ipynb) notebook
4) ```use_synthetic_examples``` can be used to set the type of in-context examples in the final prompt, i.e. it can be synthetic examples or examples from train data



## Configurations ⚙️ 

Here we define the various hyperparameters used in prompt optimization process found in [promptopt_config.yaml](demos/gsm8k/configs/promptopt_config.yaml)

- ```mutate_refine_iterations```: Number of iterations for conducting mutation of task description
 followed by refinement of instructions
- ```mutation_rounds```: Number of rounds of mutation to be performed when generating different styles
- ```refine_task_eg_iterations```: Number of iterations for refining task description and in context examples 
- ```style_variation```: Number of thinking style variations to be used in prompt mutation
- ```questions_batch_size```: Number of questions to be asked to LLM in a single batch, during training step
- ```min_correct_count```: Minimum number of batches of questions to correctly answered, for a prompt to be considered as performing good
- ```max_eval_batches```: Maximum number of mini-batches on which we should evaluate the prompt
- ```top_n```: Number of top best prompts to be considered from scoring stage for the next stage
- ```seen_set_size```: Number of samples from trainset to be used for training
- ```few_shot_count```: Number of in-context examples required in final prompt

## Best Practices 💡

Following are some of best pracitices we followed during are experiments 
- Regarding the parameters in [promptopt_config.yaml](demos/gsm8k/configs/promptopt_config.yaml)
    - We found the best performing values for ```mutate_refine_iterations```,```mutation_rounds```,```refine_task_eg_iterations``` to be 3 or 5
    - Other parameters have been set to their ideal values. ```seen_set_size``` can be increased to 50 and ```few_shot_count``` can be set based on the use case
- The prompts generated at the end of the training process are usually very detailed, however user supervision can help tune it further for the task at hand
- Trying both configurations of having synthetic in-context examples or in-context examples from the train set can be tried to find the best prompt based on use case. 

## Results 📈

<div style="text-align: center;">
<img src="./images/curve.png" width="342" height="242">
</div>

- The fiqure shows the performance profile curve for the instruction induction
tasks. The performance profile curve [(Dolan & Moré, 2002)](https://arxiv.org/abs/cs/0102001) visualizes how frequently
different approaches’ performance is within a given distance of the best performance. In this curve,
the x-axis (τ) represents the performance ratio relative to the best-performing method, and the y-axis
(p(τ )) reflects the fraction of tasks where a method’s performance is within this ratio. So for a given
method, the curve tells what percentage of the tasks are within τ distance to the best performance. \
PromptWizard consistently outperforms other methods across various
thresholds, maintaining the highest p(τ) values, indicating that it consistently performs near the best
possible accuracy across all tasks.

## How to contribute: ✋
This project welcomes contributions and suggestions. Most contributions require you to agree to a Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us the rights to use your contribution. For details, visit https://cla.microsoft.com.
When you submit a pull request, a CLA-bot will automatically determine whether you need to provide a CLA and decorate the PR appropriately (e.g., label, comment). Simply follow the instructions provided by the bot. You will only need to do this once across all repositories using our CLA.
This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact opencode@microsoft.com with any additional questions or comments.

## Citation 📝

If you make use of our work, please cite our paper:

```
@misc{agarwal2024promptwizardtaskawarepromptoptimization,
      title={PromptWizard: Task-Aware Prompt Optimization Framework}, 
      author={Eshaan Agarwal and Joykirat Singh and Vivek Dani and Raghav Magazine and Tanuja Ganu and Akshay Nambi},
      year={2024},
      eprint={2405.18369},
      archivePrefix={arXiv},
      primaryClass={cs.CL},
      url={https://arxiv.org/abs/2405.18369}, 
}
```

## Responsible AI Considerations 

### PromptWizard: Responsible AI FAQ 

- What is PromptWizard? 

    PromptWizard is a novel framework for prompt optimization that supports to tune a good prompt for a given task and dataset, so that LLMs’ output/accuracy can be optimized. PromptWizard is solely designed for research settings, and its testing has only been carried out in such environments. It should not be used in downstream applications without additional analysis and mitigation to address potential harm or bias in the proposed application. Please refer to the paper - [PromptWizard: Task-Aware Agent-driven Prompt Optimization Framework (arxiv.org)](https://arxiv.org/abs/2405.18369)-for more details. 

- What can PromptWizard do? 

    PromptWizard framework is an AI-based framework that internally uses LLM to find the optimal prompt for a given task. It takes as input task description, dataset format & few training examples, hyperparameter configurations and outputs an optimized prompt for the given LLM and task intent. 
    Unlike existing approaches, PromptWizard optimizes both prompt instructions and in-context examples, maximizing the LLM performance. It iteratively refines prompts by mutating instructions using and incorporating negative examples. It further enhances both instructions and examples with the aid of a critic provided by LLM on a candidate prompt.  
    New synthetic instructions and examples are generated with detailed reasoning steps using LLM. 

- What is/are PromptWizard’s intended use(s)? 

    Please note that PromptWizard is an open-source framework under active development and intended for use for research purposes. It should not be used in any downstream applications without additional detailed evaluation of robustness, safety issues and assessment of any potential harm or bias in the proposed application. For all GenAI applications, prompt design and tuning are a tedious, skilful and laborious tasks. PromptWizard’s intended use is to design and optimize the prompt along with the few shot examples for a given task/domain and dataset. This well crafted prompt would enable the LLM to provide more accurate and high quality answer. We have also integrated Azure AI Content Safety service, to avoid/slow-down malicious uses. 

- How was PromptWizard evaluated? What metrics are used to measure performance? 

    PromptWizard framework is generic enough to work on any domain/dataset/task. However, we have evaluated the performance of PromptWizard across 35 tasks on 8 datasets. More details can be found [PromptWizard: Task-Aware Agent-driven Prompt Optimization Framework (arxiv.org)](https://arxiv.org/abs/2405.18369) 

    The opensource datasets used for evaluation include
    - Medical challenges ([MedQA](https://github.com/jind11/MedQA), [PubMedQA](https://pubmedqa.github.io/)) 
    - Commonsense reasoning ([CSQA](https://amritasaha1812.github.io/CSQA/), [SQA](https://www.microsoft.com/en-in/download/details.aspx?id=54253))
    - Math reasoning problems ([GSM8k](https://huggingface.co/datasets/openai/gsm8k))
    - Hate speech classification ([Ethos](https://link.springer.com/article/10.1007/s40747-021-00608-2)),  
    - Complex domain-specific tasks ([MMLU](https://huggingface.co/datasets/cais/mmlu) 6 medical tasks, [Big-Bench-Hard-23](https://huggingface.co/datasets/maveriq/bigbenchhard)) 

    Additionally, the team has also conducted “red team” analysis to evaluate if PromptWizard optimizes harmful intent. With appropriate Azure content moderation deployed in the pipeline on the input to PromptWizard and output from PromptWizard, it didn’t optimize prompts for harmful intent. Please refer to the details for Azure content moderation [here](https://learn.microsoft.com/en-us/azure/ai-services/content-moderator/overview). 

- What are the limitations of PromptWizard? How can users minimize the impact of PromptWizard’s limitations when using the system? 

    - The framework is evaluated primarily on English languages tasks as described in earlier section. The framework is not yet evaluated for multilingual settings. 
    - The framework generates synthetic examples for few-shot learning based on task description. User is required to validate the correctness and diversity of generated synthetic examples. 
    - PromptWizard utilizes existing LLMs and does not train a new model. Hence, it inherits the capabilities and limitations of its base model, as well as common limitations among other large language models or limitations caused by its training process. Hence, we suggest using the appropriate base LLM suitable for your use-cases to work with PromptWizard. 

- What operational factors and settings allow for effective and responsible use of PromptWizard? 

  - Input considerations: Better performance with PromptWizard can be achieved by specifying the input components like task and intent as clearly and concisely as possible. 
  - Human involvement: PromptWizard optimizes the prompt with prompt instruction and a few shot examples for the given intent and task.  We suggest human oversight to review the optimized prompts before those are executed with LLMs. 
  - LLMs: Users can choose the LLM that is optimized for responsible use. The default LLM is GPT-4 which inherits the existing RAI mechanisms and filters from the LLM provider. Caching is enabled by default to increase reliability and control cost. We encourage developers to review [OpenAI’s Usage policies](https://openai.com/policies/usage-policies/) and [Azure OpenAI’s Code of Conduct](https://learn.microsoft.com/en-us/legal/cognitive-services/openai/code-of-conduct) when using GPT-4. 
  - Content Safety: We have integrated [Azure AI Content Safety](https://learn.microsoft.com/en-us/azure/ai-services/content-safety/overview) service for content moderation. We suggest to deploy PromptWizard with such content safety system in the pipeline.  
