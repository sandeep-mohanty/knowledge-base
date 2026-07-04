# Learn Hugging Face 🤗 - Complete Hands-On Guide

Practical hands-on machine learning tutorials using the Hugging Face ecosystem.

## Table of Contents

1. [Introduction to Hugging Face](#introduction-to-hugging-face)
2. [Prerequisites & Setup](#prerequisites--setup)
3. [Project 0: Text Classification - "Food Not Food"](#project-0-text-classification)
4. [Project 1: Object Detection - "Trashify"](#project-1-object-detection)
5. [Project 2: LLM Full Fine-tuning - "FoodExtract"](#project-2-llm-full-fine-tuning)
6. [Project 3: VLM Fine-tuning - "FoodExtract Vision"](#project-3-vlm-fine-tuning)
7. [Project 4: Multimodal RAG](#project-4-multimodal-rag)
8. [Extension: Batched Inference](#extension-batched-inference)
9. [Best Practices & Tips](#best-practices--tips)
10. [Resources & Next Steps](#resources--next-steps)

---

## Introduction to Hugging Face

Hugging Face is a platform that offers access to many different kinds of open-source machine learning models and datasets. They're also the creators of the popular `transformers` library, a Python-based library for working with pre-trained models as well as custom models.

### The Hugging Face Ecosystem

The Hugging Face ecosystem provides a comprehensive suite of tools for machine learning:

- **Transformers**: State-of-the-art models for NLP, computer vision, audio, and multimodal tasks
- **Datasets**: Library for easily accessing and processing datasets
- **Tokenizers**: Fast tokenization for NLP models
- **Hub**: Platform for hosting and sharing models, datasets, and demos
- **Spaces**: Host machine learning demos and applications
- **PEFT**: Parameter-Efficient Fine-Tuning techniques
- **Accelerate**: Easy distributed training
- **Evaluate**: Model evaluation metrics and tools
- **Gradio**: Create beautiful ML demos

Many of the biggest companies in the world use Hugging Face including Apple, Google, Meta, Microsoft, OpenAI, and ByteDance.

### Why Hugging Face?

- Easy access to state-of-the-art models (Stable Diffusion, Whisper, etc.)
- Share your own models, datasets, and resources
- Consider Hugging Face the homepage of your AI/ML profile
- Active community and extensive documentation

---

## Prerequisites & Setup

### Required Knowledge

- 3-6 months Python experience
- 1 beginner machine learning or deep learning course
- PyTorch experience is a bonus

### Installation

```bash
# Install core libraries
pip install transformers datasets evaluate torch torchvision

# Install additional tools
pip install accelerate gradio huggingface_hub

# For computer vision projects
pip install timm

# For PEFT (Parameter-Efficient Fine-Tuning)
pip install peft

# For data processing
pip install pandas matplotlib scikit-learn
```

### Create a Hugging Face Account

1. Go to [huggingface.co/join](https://huggingface.co/join)
2. Sign up for a free account
3. Get your API token from Settings → Access Tokens
4. Login via the CLI:
```bash
huggingface-cli login
```

### Verify Installation

```python
import transformers
import datasets
import torch

print(f"Transformers version: {transformers.__version__}")
print(f"Datasets version: {datasets.__version__}")
print(f"PyTorch version: {torch.__version__}")
print(f"CUDA available: {torch.cuda.is_available()}")
```

---

## Project 0: Text Classification

**Project: "Food Not Food"** - Build a text classification model to classify image captions into "food" or "not_food"

### Overview

This is the ideal starting point if you've never used the Hugging Face ecosystem. We'll follow the complete workflow: Data → Model → Demo.

**Resources:**
- Dataset: [learn_hf_food_not_food_image_captions](https://huggingface.co/datasets/mrdbourke/learn_hf_food_not_food_image_captions)
- Model: [DistilBERT classifier](https://huggingface.co/mrdbourke/learn_hf_food_not_food_text_classifier-distilbert-base-uncased)
- Demo: [Live Demo](https://huggingface.co/spaces/mrdbourke/learn_hf_food_not_food_text_classifier_demo)

### Step 1: Load and Explore the Dataset

```python
from datasets import load_dataset
import pandas as pd
import random
from collections import Counter

# Load the dataset from Hugging Face Hub
dataset = load_dataset(path="mrdbourke/learn_hf_food_not_food_image_captions")

# Check the structure
print(dataset)
# Output: DatasetDict with 'train' split

# Check column names
print(dataset.column_names)
# Output: {'train': ['text', 'label']}

# Access the training split
train_dataset = dataset["train"]

# View a sample
print(train_dataset[0])
# Output: {'text': 'Creamy cauliflower curry...', 'label': 'food'}
```

### Step 2: Explore and Visualize the Data

```python
# View random samples
random_samples = random.sample(range(len(train_dataset)), 5)
for idx in random_samples:
    item = train_dataset[idx]
    print(f"Text: {item['text']} | Label: {item['label']}")

# Get unique labels
unique_labels = train_dataset.unique("label")
print(f"Unique labels: {unique_labels}")

# Count label distribution
label_counts = Counter(train_dataset["label"])
print(f"Label distribution: {label_counts}")

# Convert to DataFrame for easier exploration
df = pd.DataFrame(train_dataset)
print(df.sample(7))
print(df['label'].value_counts())
```

### Step 3: Create Label Mappings

```python
# Create mapping from id2label and label2id
id2label = {0: "not_food", 1: "food"}
label2id = {"not_food": 0, "food": 1}

print(f"ID to Label mapping: {id2label}")
print(f"Label to ID mapping: {label2id}")

# Or create programmatically (better for many classes)
id2label = {idx: label for idx, label in enumerate(sorted(train_dataset.unique("label")))}
label2id = {label: idx for idx, label in id2label.items()}
```

### Step 4: Preprocess the Data

```python
# Function to map labels to numbers
def map_labels_to_number(example):
    example["label"] = label2id[example["label"]]
    return example

# Apply to dataset
dataset = dataset["train"].map(map_labels_to_number)

# Create train/test splits
dataset = dataset.train_test_split(test_size=0.2, seed=42)

# Verify the splits
print(dataset)
```

### Step 5: Tokenization

```python
from transformers import AutoTokenizer

# Load tokenizer
tokenizer = AutoTokenizer.from_pretrained("distilbert-base-uncased")

# Test tokenizer
tokens = tokenizer("I love pizza")
print(tokens)
# Output: {'input_ids': [101, 1045, 2293, 10733, 102], 'attention_mask': [1, 1, 1, 1, 1]}

# Tokenization function
def tokenize_text(examples):
    return tokenizer(
        examples["text"],
        truncation=True,
        padding=True,
        max_length=512
    )

# Apply tokenization to dataset
tokenized_dataset = dataset.map(
    function=tokenize_text,
    batched=True,
    remove_columns=["text"]
)

print(tokenized_dataset)
```

### Step 6: Load and Configure the Model

```python
from transformers import AutoModelForSequenceClassification

# Load model
model = AutoModelForSequenceClassification.from_pretrained(
    "distilbert-base-uncased",
    num_labels=2,
    id2label=id2label,
    label2id=label2id
)

# Count parameters
def count_params(model):
    return {
        'trainable_parameters': sum(p.numel() for p in model.parameters() if p.requires_grad),
        'total_parameters': sum(p.numel() for p in model.parameters())
    }

print(count_params(model))
```

### Step 7: Define Metrics

```python
import evaluate
import numpy as np

# Load accuracy metric
accuracy = evaluate.load("accuracy")

def compute_metrics(eval_pred):
    predictions, labels = eval_pred
    predictions = np.argmax(predictions, axis=1)
    return accuracy.compute(predictions=predictions, references=labels)
```

### Step 8: Set Up Training

```python
from transformers import TrainingArguments, Trainer
from pathlib import Path

# Create model save directory
model_save_dir = Path("models/learn_hf_food_not_food_text_classifier-distilbert-base-uncased")
model_save_dir.mkdir(parents=True, exist_ok=True)

# Define training arguments
training_args = TrainingArguments(
    output_dir=str(model_save_dir),
    per_device_train_batch_size=8,
    per_device_eval_batch_size=8,
    num_train_epochs=3,
    evaluation_strategy="epoch",
    save_strategy="epoch",
    load_best_model_at_end=True,
    metric_for_best_model="accuracy",
    logging_dir="./logs",
    logging_steps=10,
    report_to="none",  # Disable wandb/tensorboard for simplicity
)

# Create trainer
trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=tokenized_dataset["train"],
    eval_dataset=tokenized_dataset["test"],
    tokenizer=tokenizer,
    compute_metrics=compute_metrics,
)
```

### Step 9: Train the Model

```python
# Train the model
results = trainer.train()

# Inspect training metrics
for key, value in results.metrics.items():
    print(f"{key}: {value}")

# Get training history
trainer_history = trainer.state.log_history

# Separate training and evaluation metrics
training_metrics = [item for item in trainer_history if 'loss' in item and 'eval_loss' not in item]
eval_metrics = [item for item in trainer_history if 'eval_loss' in item]

# Visualize training loss
import matplotlib.pyplot as plt

training_loss = [item['loss'] for item in training_metrics]
plt.plot(training_loss)
plt.title('Training Loss')
plt.xlabel('Step')
plt.ylabel('Loss')
plt.show()
```

### Step 10: Evaluate the Model

```python
# Evaluate on test set
predictions = trainer.predict(tokenized_dataset["test"])
print(f"Test metrics: {predictions.metrics}")

# Get predictions and labels
preds = np.argmax(predictions.predictions, axis=1)
labels = predictions.label_ids

# Calculate accuracy
from sklearn.metrics import accuracy_score
test_accuracy = accuracy_score(labels, preds)
print(f"Test accuracy: {test_accuracy * 100:.2f}%")

# Create DataFrame for analysis
test_df = pd.DataFrame({
    'text': dataset["test"]["text"],
    'true_label': [id2label[label] for label in labels],
    'predicted_label': [id2label[pred] for pred in preds],
    'pred_prob': np.max(predictions.predictions, axis=1)
})

# Find uncertain predictions
uncertain = test_df.sort_values("pred_prob", ascending=True).head(10)
print("Most uncertain predictions:")
print(uncertain)
```

### Step 11: Save and Upload to Hub

```python
# Save model locally
trainer.save_model(output_dir=model_save_dir)
print(f"Model saved to {model_save_dir}")

# Push to Hugging Face Hub
model.push_to_hub("learn_hf_food_not_food_text_classifier")
tokenizer.push_to_hub("learn_hf_food_not_food_text_classifier")

# Create model card (README.md for your model)
```

### Step 12: Create Inference Pipeline

```python
import torch

# Set device
def set_device():
    device = "cuda" if torch.cuda.is_available() else "cpu"
    print(f"[INFO] Using device: {device}")
    return device

DEVICE = set_device()

# Create pipeline
from transformers import pipeline

classifier = pipeline(
    task="text-classification",
    model=model,
    tokenizer=tokenizer,
    device=0 if torch.cuda.is_available() else -1
)

# Test the classifier
sample_text = "A delicious photo of a plate of scrambled eggs, bacon and toast"
result = classifier(sample_text)
print(result)

# Another test
sample_text_2 = "Set of oven mitts hanging on a hook"
result_2 = classifier(sample_text_2)
print(result_2)
```

### Step 13: Create Gradio Demo

```python
import gradio as gr

def classify_text(text):
    result = classifier(text)[0]
    return {
        "food": result['score'] if result['label'] == 'food' else 1 - result['score'],
        "not_food": result['score'] if result['label'] == 'not_food' else 1 - result['score']
    }

demo = gr.Interface(
    fn=classify_text,
    inputs=gr.Textbox(placeholder="Enter text to classify..."),
    outputs=gr.Label(num_top_classes=2),
    title="Food Not Food Classifier",
    description="Classify text as 'food' or 'not_food' using a fine-tuned DistilBERT model"
)

demo.launch()
```

---

## Project 1: Object Detection

**Project: "Trashify 🚮"** - Build an object detection model to detect "trash", "hand", "bin"

### Overview

Learn computer vision with Hugging Face by building and deploying an object detection model.

**Resources:**
- Dataset: [trashify_manual_labelled_images](https://huggingface.co/datasets/mrdbourke/trashify_manual_labelled_images)
- Model: [RT-DETRv2 fine-tuned](https://huggingface.co/mrdbourke/rt_detrv2_finetuned_trashify_box_detector_v1)
- Demo: [Live Demo](https://huggingface.co/spaces/mrdbourke/trashify_demo_v4)

### Key Steps

```python
from datasets import load_dataset
from transformers import AutoImageProcessor, AutoModelForObjectDetection
from PIL import Image
import requests

# 1. Load dataset
dataset = load_dataset("mrdbourke/trashify_manual_labelled_images")

# 2. Load image processor and model
image_processor = AutoImageProcessor.from_pretrained("PekingU/rtdetr_r50vd")
model = AutoModelForObjectDetection.from_pretrained("PekingU/rtdetr_r50vd")

# 3. Preprocess images
def prepare_examples(examples):
    images = [Image.open(path).convert("RGB") for path in examples['image']]
    targets = [
        {
            'image_id': idx,
            'annotations': [
                {
                    'area': ann['area'],
                    'bbox': ann['bbox'],
                    'category_id': ann['category_id']
                }
                for ann in annotations
            ]
        }
        for idx, annotations in enumerate(examples['objects'])
    ]
    
    result = image_processor(images=images, annotations=targets, return_tensors="pt")
    result["labels"] = targets
    return result

# 4. Fine-tune the model
from transformers import TrainingArguments, Trainer

training_args = TrainingArguments(
    output_dir="./trashify-detector",
    per_device_train_batch_size=4,
    num_train_epochs=10,
    evaluation_strategy="epoch",
)

trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=dataset["train"],
    eval_dataset=dataset["test"],
    tokenizer=image_processor,
)

trainer.train()

# 5. Create inference pipeline
from transformers import pipeline

object_detector = pipeline(
    "object-detection",
    model=model,
    image_processor=image_processor
)

# 6. Test on new image
result = object_detector("path/to/image.jpg")
print(result)
```

---

## Project 2: LLM Full Fine-tuning

**Project: "FoodExtract"** - Fine-tune Google's Gemma 3 270M for structured data extraction

### Overview

Learn to fully fine-tune a Small Language Model (SLM) for structured data extraction tasks.

**Resources:**
- Dataset: [FoodExtract-1k](https://huggingface.co/datasets/mrdbourke/FoodExtract-1k)
- Model: [Fine-tuned Gemma 3 270M](https://huggingface.co/mrdbourke/FoodExtract-gemma-3-270m-fine-tune-v1)
- Demo: [Live Demo](https://huggingface.co/spaces/mrdbourke/FoodExtract-v1)

### Key Steps

```python
from datasets import load_dataset
from transformers import AutoTokenizer, AutoModelForCausalLM
from peft import LoraConfig, get_peft_model, TaskType
from transformers import TrainingArguments, Trainer
import torch

# 1. Load dataset
dataset = load_dataset("mrdbourke/FoodExtract-1k")

# 2. Load tokenizer and model
model_id = "google/gemma-3-270m-it"
tokenizer = AutoTokenizer.from_pretrained(model_id)
model = AutoModelForCausalLM.from_pretrained(
    model_id,
    device_map="auto",
    torch_dtype=torch.bfloat16
)

# 3. Configure LoRA for efficient fine-tuning
lora_config = LoraConfig(
    r=8,
    lora_alpha=32,
    target_modules=["q_proj", "k_proj", "v_proj", "o_proj"],
    lora_dropout=0.05,
    bias="none",
    task_type="CAUSAL_LM"
)

model = get_peft_model(model, lora_config)
model.print_trainable_parameters()

# 4. Tokenize dataset
def tokenize_function(examples):
    return tokenizer(
        examples["text"],
        truncation=True,
        max_length=512,
        padding="max_length"
    )

tokenized_dataset = dataset.map(tokenize_function, batched=True)

# 5. Set up training
training_args = TrainingArguments(
    output_dir="./foodextract-gemma",
    per_device_train_batch_size=4,
    gradient_accumulation_steps=4,
    num_train_epochs=3,
    learning_rate=2e-4,
    fp16=True,
    logging_steps=10,
    save_strategy="epoch",
    report_to="none",
)

trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=tokenized_dataset["train"],
    tokenizer=tokenizer,
)

# 6. Train
trainer.train()

# 7. Save and push to hub
model.push_to_hub("FoodExtract-gemma-3-270m-fine-tune")
```

---

## Project 3: VLM Fine-tuning

**Project: "FoodExtract Vision"** - Fine-tune SmolVLM2-500M for structured data extraction from images

### Overview

Combine vision and language understanding by fine-tuning a Vision Language Model.

**Resources:**
- Dataset: [FoodExtract-1k-Vision](https://huggingface.co/datasets/mrdbourke/FoodExtract-1k-Vision)
- Model: [Fine-tuned SmolVLM2-500M](https://huggingface.co/mrdbourke/FoodExtract-Vision-SmolVLM2-500M-fine-tune-v1)
- Demo: [Live Demo](https://huggingface.co/spaces/mrdbourke/FoodExtract-Vision-v1)

### Key Concepts

Vision Language Models (VLMs) can understand both images and text, making them perfect for tasks like:
- Image captioning
- Visual question answering
- Document understanding
- Structured data extraction from images

### Key Steps

```python
from transformers import AutoProcessor, AutoModelForVision2Seq
from datasets import load_dataset
from peft import LoraConfig, get_peft_model
from transformers import TrainingArguments, Trainer

# 1. Load dataset with images
dataset = load_dataset("mrdbourke/FoodExtract-1k-Vision")

# 2. Load processor and model
processor = AutoProcessor.from_pretrained("HuggingFaceTB/SmolVLM2-500M-Video-Instruct")
model = AutoModelForVision2Seq.from_pretrained(
    "HuggingFaceTB/SmolVLM2-500M-Video-Instruct",
    device_map="auto",
    torch_dtype=torch.bfloat16
)

# 3. Prepare examples
def prepare_examples(examples):
    images = [Image.open(path).convert("RGB") for path in examples['image']]
    texts = [f"Extract food information: {text}" for text in examples['text']]
    
    inputs = processor(
        text=texts,
        images=images,
        return_tensors="pt",
        padding=True,
        truncation=True
    )
    
    inputs["labels"] = inputs["input_ids"].clone()
    return inputs

# 4. Configure LoRA
lora_config = LoraConfig(
    r=8,
    lora_alpha=32,
    target_modules=["q_proj", "v_proj"],
    lora_dropout=0.05,
    task_type="CAUSAL_LM"
)

model = get_peft_model(model, lora_config)

# 5. Train
training_args = TrainingArguments(
    output_dir="./smolvlm-foodextract",
    per_device_train_batch_size=2,
    gradient_accumulation_steps=8,
    num_train_epochs=3,
    learning_rate=2e-4,
    fp16=True,
    report_to="none",
)

trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=dataset["train"],
    tokenizer=processor,
    data_collator=processor,
)

trainer.train()
```

---

## Project 4: Multimodal RAG

**Project: Multimodal Recipe RAG** - Build a RAG system with text and image embeddings

### Overview

Level up your RAG pipelines by embedding text and images into a shared embedding space for unified retrieval.

**Resources:**
- Dataset: [recipe-synthetic-images-10k](https://huggingface.co/datasets/mrdbourke/recipe-synthetic-images-10k)
- Models: 
  - [NVIDIA Nemotron Embed VL](https://huggingface.co/nvidia/llama-nemotron-embed-vl-1b-v2)
  - [NVIDIA Nemotron Rerank VL](https://huggingface.co/nvidia/llama-nemotron-rerank-vl-1b-v2)
- Demo: [Live Demo](https://huggingface.co/spaces/mrdbourke/multimodal-rag-with-nemotron)

### Key Concepts

**RAG (Retrieval-Augmented Generation)** combines:
1. **Retrieval**: Find relevant information from a knowledge base
2. **Augmentation**: Add retrieved information to the context
3. **Generation**: Generate responses based on augmented context

**Multimodal RAG** extends this to work with both text and images.

### Key Steps

```python
from transformers import AutoProcessor, AutoModel
from datasets import load_dataset
import torch
from PIL import Image

# 1. Load embedding model
embed_model_id = "nvidia/llama-nemotron-embed-vl-1b-v2"
processor = AutoProcessor.from_pretrained(embed_model_id)
embed_model = AutoModel.from_pretrained(embed_model_id, device_map="auto")

# 2. Load dataset
dataset = load_dataset("mrdbourke/recipe-synthetic-images-10k")

# 3. Create embeddings
def get_embeddings(texts, images=None):
    inputs = processor(
        text=texts,
        images=images,
        return_tensors="pt",
        padding=True,
        truncation=True
    ).to(embed_model.device)
    
    with torch.no_grad():
        outputs = embed_model(**inputs)
        embeddings = outputs.last_hidden_state.mean(dim=1)
    
    return embeddings.cpu().numpy()

# 4. Build vector store
from sklearn.neighbors import NearestNeighbors
import numpy as np

# Generate embeddings for all documents
all_embeddings = []
for item in dataset["train"]:
    embedding = get_embeddings([item["text"]], [item["image"]])
    all_embeddings.append(embedding[0])

all_embeddings = np.array(all_embeddings)

# Create nearest neighbor index
nn = NearestNeighbors(n_neighbors=5, metric='cosine')
nn.fit(all_embeddings)

# 5. Query the RAG system
def query_rag(query_text, query_image=None, k=5):
    # Get query embedding
    query_embedding = get_embeddings([query_text], [query_image] if query_image else None)
    
    # Find nearest neighbors
    distances, indices = nn.kneighbors(query_embedding, n_neighbors=k)
    
    # Retrieve results
    results = []
    for idx, dist in zip(indices[0], distances[0]):
        results.append({
            'document': dataset["train"][idx],
            'distance': dist
        })
    
    return results

# 6. Use with LLM for generation
from transformers import AutoModelForCausalLM, AutoTokenizer

llm_id = "meta-llama/Llama-2-7b-chat-hf"
llm_tokenizer = AutoTokenizer.from_pretrained(llm_id)
llm = AutoModelForCausalLM.from_pretrained(llm_id, device_map="auto")

def generate_response(query, retrieved_docs):
    # Construct prompt with retrieved context
    context = "\n".join([doc['document']['text'] for doc in retrieved_docs])
    prompt = f"Context:\n{context}\n\nQuestion: {query}\n\nAnswer:"
    
    inputs = llm_tokenizer(prompt, return_tensors="pt").to(llm.device)
    outputs = llm.generate(**inputs, max_length=500)
    response = llm_tokenizer.decode(outputs[0], skip_special_tokens=True)
    
    return response
```

---

## Extension: Batched Inference

### Overview

Learn how to speed up LLM inference by batching samples together.

**Resources:**
- Notebook: [Batched Inference Tutorial](https://www.learnhuggingface.com/notebooks/hugging_face_llm_batched_inference_with_transformers)

### Key Concepts

Batching multiple samples together can significantly speed up inference by:
- Better GPU utilization
- Reduced overhead
- Higher throughput

### Example Implementation

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
from datasets import load_dataset
import torch
from torch.utils.data import DataLoader, Dataset

# 1. Load model and tokenizer
model_id = "microsoft/phi-2"
tokenizer = AutoTokenizer.from_pretrained(model_id)
model = AutoModelForCausalLM.from_pretrained(
    model_id,
    device_map="auto",
    torch_dtype=torch.float16
)

# 2. Create dataset
class TextDataset(Dataset):
    def __init__(self, texts, tokenizer, max_length=512):
        self.texts = texts
        self.tokenizer = tokenizer
        self.max_length = max_length
    
    def __len__(self):
        return len(self.texts)
    
    def __getitem__(self, idx):
        encoding = self.tokenizer(
            self.texts[idx],
            truncation=True,
            max_length=self.max_length,
            return_tensors="pt"
        )
        return {k: v.squeeze(0) for k, v in encoding.items()}

# 3. Create dataloader with batching
texts = ["What is machine learning?", "Explain neural networks", "What is deep learning?"]
dataset = TextDataset(texts, tokenizer)
dataloader = DataLoader(dataset, batch_size=2, shuffle=False)

# 4. Batch inference
model.eval()
all_outputs = []

with torch.no_grad():
    for batch in dataloader:
        inputs = {k: v.to(model.device) for k, v in batch.items()}
        outputs = model.generate(**inputs, max_length=100)
        decoded = tokenizer.batch_decode(outputs, skip_special_tokens=True)
        all_outputs.extend(decoded)

for i, output in enumerate(all_outputs):
    print(f"Input {i+1}: {texts[i]}")
    print(f"Output {i+1}: {output}\n")
```

---

## Best Practices & Tips

### 1. Data Practices

- **Always visualize your data** before training
- **Check for class imbalance** and handle it appropriately
- **Create proper train/validation/test splits**
- **Use seed values** for reproducibility
- **Start small** - test with a small subset first

### 2. Model Training

```python
# Use mixed precision training for faster training
training_args = TrainingArguments(
    fp16=True,  # or bf16=True for newer GPUs
    gradient_accumulation_steps=4,  # Simulate larger batch sizes
    gradient_checkpointing=True,  # Save memory
)

# Monitor training
training_args = TrainingArguments(
    logging_steps=10,
    evaluation_strategy="epoch",
    save_strategy="epoch",
    load_best_model_at_end=True,
)
```

### 3. Memory Optimization

```python
# Use PEFT/LoRA for fine-tuning large models
from peft import LoraConfig, get_peft_model

lora_config = LoraConfig(
    r=8,  # Rank
    lora_alpha=32,
    target_modules=["q_proj", "v_proj"],
    lora_dropout=0.05,
    bias="none",
    task_type="CAUSAL_LM"
)

model = get_peft_model(model, lora_config)

# Use gradient checkpointing
model.gradient_checkpointing_enable()

# Use 8-bit or 4-bit quantization
from transformers import BitsAndBytesConfig

bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_use_double_quant=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype=torch.bfloat16
)
```

### 4. Evaluation

```python
# Always evaluate on a held-out test set
# Use multiple metrics
import evaluate

accuracy = evaluate.load("accuracy")
f1 = evaluate.load("f1")
precision = evaluate.load("precision")
recall = evaluate.load("recall")

# Create comprehensive evaluation function
def compute_metrics(eval_pred):
    predictions, labels = eval_pred
    predictions = np.argmax(predictions, axis=1)
    
    return {
        'accuracy': accuracy.compute(predictions=predictions, references=labels),
        'f1': f1.compute(predictions=predictions, references=labels, average='weighted'),
        'precision': precision.compute(predictions=predictions, references=labels, average='weighted'),
        'recall': recall.compute(predictions=predictions, references=labels, average='weighted'),
    }
```

### 5. Deployment

```python
# Create efficient inference pipelines
from transformers import pipeline

# Text classification
classifier = pipeline("text-classification", model=model, tokenizer=tokenizer)

# Object detection
detector = pipeline("object-detection", model=model)

# Text generation
generator = pipeline("text-generation", model=model, tokenizer=tokenizer)

# Use Gradio for quick demos
import gradio as gr

demo = gr.Interface(
    fn=classifier,
    inputs="text",
    outputs="label",
    title="My Model",
    description="A simple demo"
)

# Deploy to Hugging Face Spaces
demo.launch()
```

### 6. Version Control and Sharing

```python
# Save model card with your model
model_card = """
# Model Name

## Model Description
This model is fine-tuned for...

## Training Data
- Dataset: [link]
- Size: X examples
- Task: classification

## Usage
```python
from transformers import pipeline

classifier = pipeline("text-classification", model="your-username/model-name")
result = classifier("Your text here")
```

## Evaluation Results
- Accuracy: X%
- F1 Score: X

## Limitations and Bias
...
"""

# Push to hub with model card
model.push_to_hub("model-name", model_card=model_card)
```

---

## Resources & Next Steps

### Official Documentation

- **Hugging Face Docs**: [huggingface.co/docs](https://huggingface.co/docs)
- **Transformers**: [huggingface.co/docs/transformers](https://huggingface.co/docs/transformers)
- **Datasets**: [huggingface.co/docs/datasets](https://huggingface.co/docs/datasets)
- **PEFT**: [huggingface.co/docs/peft](https://huggingface.co/docs/peft)
- **Accelerate**: [huggingface.co/docs/accelerate](https://huggingface.co/docs/accelerate)

### Learning Resources

- **Learn Hugging Face**: [learnhuggingface.com](https://www.learnhuggingface.com/)
- **Video Course**: [Zero to Mastery Hugging Face Bootcamp](https://dbourke.link/ZTMHuggingFace)
- **GitHub Repository**: [github.com/mrdbourke/learn-huggingface](https://github.com/mrdbourke/learn-huggingface)

### Community

- **Hugging Face Forums**: [discuss.huggingface.co](https://discuss.huggingface.co)
- **Discord**: Join the Hugging Face Discord
- **GitHub Issues**: Report issues at [github.com/mrdbourke/learn-huggingface/issues](https://github.com/mrdbourke/learn-huggingface/issues)

### Next Projects to Try

1. **Text Classification** - Start here if you're new
2. **Object Detection** - Computer vision fundamentals
3. **LLM Fine-tuning** - Work with large language models
4. **VLM Fine-tuning** - Multimodal AI
5. **Multimodal RAG** - Advanced retrieval systems
6. **Batched Inference** - Optimize production systems

### Our Mottos

Remember these principles as you work:

1. **"If in doubt, run the code."** – Machine learning is experimental. Try things!
2. **"Visualize, visualize, visualize!"** – Always inspect your data and results
3. **"Experiment, experiment, experiment!"** – Keep trying different approaches
4. **"Data, model, demo!"** – Create data, build models, share demos

---

## Updates

- **10 June 2026** - All videos for the LLM fine-tuning course are live on ZTM
- **16 Apr 2026** - Added batched inference notebook
- **1 Apr 2026** - Fully finished LLM fine-tuning notebook
- **26 Feb 2026** - Updated links and added dark mode
- **08 Jan 2026** - Added LLM fine-tuning notebook for Gemma 3 270M
- **07 Nov 2025** - Videos for object detection project available
- **18 June 2025** - Completed object detection project
- **1 Oct 2024** - Video course version of text classification went live

---

## FAQ

**Is this an official Hugging Face website?**

No, it's a personal project by Daniel Bourke to help others learn the Hugging Face ecosystem.

**How is this website made?**

This is a Quarto website. To learn more, visit [quarto.org/docs/websites](https://quarto.org/docs/websites).

---

## Contributing

Found a bug or want to suggest a new tutorial? Leave an issue at [github.com/mrdbourke/learn-huggingface/issues](https://github.com/mrdbourke/learn-huggingface/issues).

---

**Happy Learning! 🤗**

*Start with the text classification project and work your way up. Remember: the best way to learn is by doing!*