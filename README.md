<p align="center">
<h2 align="center">INTELLECT-MATH: Frontier Mathematical Reasoning through Better Initializations for Reinforcement Learning</h1>
</p>

![diagram-training-techniques](https://github.com/user-attachments/assets/81dc657f-5b37-4dc4-b1ba-42de9fb61e0a)


<p align="center">
<a href="https://www.primeintellect.ai/blog/intellect-math">🔗 Blog Post</a>
<span>&nbsp;&nbsp;•&nbsp;&nbsp;</span>
<a href="">🐦 X / Twitter</a>
<span>&nbsp;&nbsp;•&nbsp;&nbsp;</span>
<a href="https://huggingface.co/collections/PrimeIntellect/intellect-math-678a2a25d7c5d74b37b16581">🤗 Weights & Data</a>
</p>
<p>
  
</p>




This repository contains code for reproducing INTELLECT-MATH, a frontier 7B parameter model for mathematical reasoning. 

INTELLECT-Math was trained in two stages: an initial SFT stage and a second online reinforcement learning stage based on [Process Reinforcement through Implicit Rewards](https://github.com/PRIME-RL/PRIME).

By generating our SFT dataset with a strong teacher model like QwQ-32B, we can provide a better policy initialization for online reinforcement learning. This way, we can match the performance of the existing SOTA model Eurus-2-7B-Prime with 10x less time spent on reinforcement learning, and outperform the model with further training.

|      | Intellect-Math (Step 255) | Intellect-Math (Step 47) | Eurus-2-Prime (Step 592) | Intellect-Math-SFT | Eurus-2-SFT | Qwen-2.5-Math |
|----------------|---------------------------:|--------------------------:|--------------------------:|--------------------:|------------:|-------------:|
| **MATH-500**   | 82.0                      | 81.6                     | 79.2                     | 72.8               | 65.1        | 79.8         |
| **OLYMPIADBENCH** | 49.5                   | 46.7                     | 42.1                     | 39.1               | 29.8        | 40.7         |
| **AIME 2024**  | 26.7                      | 26.7                     | 26.7                     | 16.6               | 3.3         | 13.3         |
| **AMC**        | 60.2                      | 57.8                     | 57.8                     | 45.8               | 30.1        | 50.6         |
| **MINERVA MATH** | 39.7                    | 37.8                     | 38.6                     | 33.8               | 32.7        | 34.6         |
| **AVG**        | 51.6                      | 50.1                     | 48.9                     | 41.6               | 32.2        | 43.8         |


<p></p>

## Reproducing INTELLECT-MATH

You can reproduce INTELLECT-MATH step by step using the following code:

### 1) Generate the SFT Dataset

To generate our SFT dataset, we used QwQ-32B to sample two responses for every question from the NuminaMath dataset. To achieve fast throughput and large batch sizes, we use the [sglang](https://github.com/sgl-project/sglang) inference engine. Keeping only the correct responses, we are left with [PrimeIntellect/INTELLECT-MATH-SFT-Data](https://huggingface.co/datasets/PrimeIntellect/INTELLECT-MATH-SFT-Data), a dataset containing 733k questions and responses.

You can use the code in `synthetic-data` to generate an SFT dataset:
```
cd synthetic-data

# install requirements
pip install -r requirements.txt

# generate data, sampling two responses per question
python generate.py --num_responses_per_question 2
```

### 2) Fine-tune a model on the synthetic SFT dataset

We used [open-instruct](https://github.com/allenai/open-instruct) to fine-tune [Qwen/Qwen2.5-Math-7B](https://huggingface.co/Qwen/Qwen2.5-Math-7B) into [PrimeIntellect/INTELLECT-MATH-SFT](https://huggingface.co/PrimeIntellect/INTELLECT-MATH-SFT). The code for can be found in `sft`. Follow the README inside the folder to set up your environment - then you can reproduce our SFT model using the script `intellect-math-scripts/train_intellect_math_7b_sft.sh`.


### 3) Online Reinforcement Learning on top of your SFT model
Finally, you need to further train your SFT model with reinforcement learning using [PRIME-RL](https://github.com/PRIME-RL/PRIME). This last step was used to train [PrimeIntellect/INTELLECT-MATH](https://huggingface.co/PrimeIntellect/INTELLECT-MATH) from [PrimeIntellect/INTELLECT-MATH-SFT](https://huggingface.co/PrimeIntellect/INTELLECT-MATH-SFT). The code for this, along with evals, can be found in `rl-and-evals`. To reproduce our model, you can use the script `rl-and-evals/training/intellect-math-scripts/train_intellect_math_7b.sh`.
