# README: Fine-tuning a Foundation Model with Vertex AI and Kubeflow

## Project Overview

This project involves setting up a data exploration and model tuning pipeline using Google Cloud's Vertex AI, BigQuery, and Kubeflow Pipelines. The dataset includes Stack Overflow questions and answers, with a focus on Python-related queries for effective model tuning.

---

## Authentication and Setup

First, load the project ID and credentials:

```python
from utils import authenticate
credentials, PROJECT_ID = authenticate() 
REGION = "us-central1"
```

Import the Vertex AI SDK, load the model, and initialize the environment:

```python
import vertexai
from vertexai.language_models import TextGenerationModel
vertexai.init(project=PROJECT_ID, location=REGION, credentials=credentials)
```

---

## Model Deployment and Load Balancing

Load the pre-trained `text-bison@001` model and retrieve deployed endpoints:

```python
model = TextGenerationModel.from_pretrained("text-bison@001")
list_tuned_models = model.list_tuned_model_names()
```

Randomly select a model for load balancing:

```python
import random
tuned_model_select = random.choice(list_tuned_models)
deployed_model = TextGenerationModel.get_tuned_model(tuned_model_select)
```

---

## Prediction and Response Handling

To generate predictions:

```python
PROMPT = "How can I load a CSV file using Pandas?"
response = deployed_model.predict(PROMPT)
print(response)
```

For readability:

```python
from pprint import pprint
output = response._prediction_response[0][0]["content"]
pprint(output)
```

---

## Safety Check Implementation

Check if the response is blocked or contains unwanted content:

```python
blocked = response._prediction_response[0][0]['safetyAttributes']['blocked']
print(blocked)
```

Review safety scores to ensure content appropriateness:

```python
safety_attributes = response._prediction_response[0][0]['safetyAttributes']
pprint(safety_attributes)
```

---

## Prompt Management and Templates

Ensure prompts maintain consistency with training data. Use clear instructions:

```python
INSTRUCTION = "Please answer the following Stackoverflow question on Python. Answer it like you are a developer answering Stackoverflow questions."
QUESTION = "How can I store my TensorFlow checkpoint on Google Cloud Storage?"
PROMPT = f"{INSTRUCTION} {QUESTION}"
final_response = deployed_model.predict(PROMPT)
```

---

## Kubeflow Pipelines

Kubeflow pipelines streamline automation by creating modular components.

#### Sample Pipeline

```python
@dsl.component
def say_hello(name: str) -> str:
    return f'Hello, {name}!'

@dsl.component
def how_are_you(hello_text: str) -> str:
    return f"{hello_text}. How are you?"

@dsl.pipeline
def hello_pipeline(recipient: str) -> str:
    hello_task = say_hello(name=recipient)
    how_task = how_are_you(hello_text=hello_task.output)
    return how_task.output

compiler.Compiler().compile(hello_pipeline, 'pipeline.yaml')
```

Submit the pipeline:

```python
from google.cloud.aiplatform import PipelineJob
pipeline_arguments = {"recipient": "World!"}
job = PipelineJob(
    template_path="pipeline.yaml",
    display_name="deep_learning_ai_pipeline",
    parameter_values=pipeline_arguments,
    location="us-central1",
    pipeline_root="./",
)

job.submit()
print(job.state)
```

---

## Contact
For questions or contributions, please reach out to Tapojita Kar at LinkedIn.

