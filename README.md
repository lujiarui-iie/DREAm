# DREAm: Dual-perspective Reasoning and Attribution-based Refinement for Conversational Query Rewriting

<img width="1515" alt="DREAm Framework" src="https://github.com/lujiarui-iie/DREAm/blob/main/assets/framework.jpg" />

## 📖 Introduction
This repository contains the implementation of **DREAm** (*Dual-perspective Reasoning and Attribution-based Refinement*). We propose a method for Conversational Query Rewriting that leverages dual-perspective reasoning and attribution-based refinement. The method is evaluated on **QReCC** and **TopiOCQA** benchmarks using both **sparse (BM25)** and **dense (ANCE)** retrievers.

## 🧠 AIR3 Algorithm

The core of DREAm is the **Attribution-based Iterative Reasoning-Rewrite Refinement (AIR3)** algorithm. It aligns the reasoning rewrite model with the retriever by leveraging passage feedback and rank feedback to iteratively prune noisy and redundant reasoning paths.

<img width="1200" alt="AIR3 Algorithm Pseudocode" src="https://github.com/lujiarui-iie/DREAm/blob/main/assets/pseudocode.png" />


## ⚙️ Installation

Create a virtual environment and install the required dependencies.

```bash
conda create -n dream python==3.8
conda activate dream

# Install main packages
pip install torch==1.8.1 transformers==4.2.0 
pip install numpy==1.22 
pip install faiss-gpu==1.7.2 pyserini==0.16

# Install LLaMA-Factory
cd ./code/LLaMA-Factory
pip install -e .
```

## 📂 Data Preparation

#### Passage Collection

Download the raw datasets (including passages) from their respective repositories:

- [QReCC](https://github.com/apple/ml-qrecc)
- [TopiOCQA](https://github.com/McGill-NLP/topiocqa)


Process the raw datasets:

```bash
# For qrecc dataset
python ./index/preprocess/qrecc/qrecc_processing.py

# For topiocqa dataset
python ./index/preprocess/topiocqa/topiocqa_processing.py
```

#### Index Construction

Run the following scripts to build the BM25 and Dense retrieval indices:

```bash
# Sparse Retrieval
bash ./index/preprocess/bm25/create_index.sh

# Dense Retrieval
bash ./index/preprocess/dense/distributed_index.sh
```

The output should be saved to `./index/{retrieval_type}_{dataset_name}`.

For Dense retrieval, you must download the following checkpoints and place them in `./index/ance_checkpoint/Passage_ANCE_FirstP_Checkpoint` before index construction:

```bash
# pip install huggingface_hub
huggingface-cli download 3ricL/ad-hoc-ance-msmarco --local-dir ./index/ance_checkpoint/Passage_ANCE_FirstP_Checkpoint --local-dir-use-symlinks False
```

## 🚀 Training

#### Data Setup

Download the required datasets and auxiliary data, then organize them as follows:

- Standardized Datasets: Download the preprocessed train/test sets for QReCC and TopiOCQA from [HERE](https://drive.google.com/drive/folders/1fFWixFDgrxwzyftlX34I7MnkbrpmcXJs?usp=sharing) and place them in `./dataset/{dataset_name}`.
- Reasoning Data: Download the DeepSeek-generated reasoning processing and rewrites from [HERE](https://drive.google.com/drive/folders/1UYq2bZRoA_Jl_VuAqBqtVFPcseDcEGuO?usp=sharing) and place them in `./code/think_data`.

Convert the data to the format supported by LLaMA-Factory:

```bash
python ./code/data_processing/convert_lf_format.py
```

Note: You need to manually update `dataset_info.json` in the `LLaMA-Factory` directory to register the new datasets.

#### Pruning

Run the pruning process using the Pruner trained on the original dataset:

```bash
python ./utils/get_keyword.py
python ./utils/pruning.py
```

#### Supervised Fine-Tuning

Start the training process:

```bash
bash ./code/train_eval/train_sft.sh
```

## 🔎 Inference & Evaluation

#### Inference

Generate rewrites using the trained model:

```bash
python ./code/train_eval/infer.py
```

#### Evaluation

Evaluate retrieval performance using Sparse (BM25) or Dense (ANCE) retrieval:

```bash
# Sparse Retrieval Evaluation
python ./code/train_eval/search_sparse.py

# Dense Retrieval Evaluation
python ./code/train_eval/search_dense.py
```

#### All-in-One Script

To run the entire pipeline (Training -> Inference -> Evaluation):

```bash
bash train_infer_eval.sh
```

## 📊 Experiments

#### Main Results

Our experiments demonstrate DREAm's robust generalizability across multiple dimensions. We validate the framework using two distinct backbone models (Llama and Qwen) and two types of retrievers (sparse BM25 and dense ANCE) on both QReCC and TopiOCQA benchmarks.

<img width="1200" alt="Main Result" src="https://github.com/lujiarui-iie/DREAm/blob/main/assets/main_result.png" />

#### Ablation Studies

We conducted comprehensive ablation experiments to verify our core contributions: the dual-perspective reasoning and the pruning strategy. Ablating each perspective reveals that historical context enrichment and intent exploration are mutually reinforcing, demonstrating that both are indispensable for achieving optimal retrieval alignment. Furthermore, regarding different pruning strategies, we experimented with various static pruning iteration counts and dynamic pruning algorithms. These results highlight the necessity of retrieval rank feedback for effective refinement.

<img width="500" alt="Vanilla Ablation" src="https://github.com/lujiarui-iie/DREAm/blob/main/assets/vanilla_ablation.png" />

<img width="500" alt="Pruning Strategy Ablation" src="https://github.com/lujiarui-iie/DREAm/blob/main/assets/pruning_strategy.png" />

We investigate different aggregation strategies for converting attention weights into pruning scores, both across attention heads (for token scores) and across tokens (for sentence scores). These experiments are performed on the TopiOCQA dataset using BM25 for retrieval, with Llama3.2-3b-Instruct serving as the backbone model."

<img width="500" alt="Pruning Score Calculation Ablation" src="https://github.com/lujiarui-iie/DREAm/blob/main/assets/pruning_score_calculation.jpg" />


## 🤝 Acknowledgments

We appreciate the open-source contributions from the following projects:

- [LLaMA-Factory](https://github.com/hiyouga/LLaMA-Factory)
- [(*EMNLP 2024*) CHIQ: Contextual History Enhancement for Improving Query Rewriting in Conversational Search](https://github.com/fengranMark/CHIQ)
- [(*ACL 2023*) ConvGQR: Generative Query Reformulation for Conversational Search](https://github.com/fengranMark/ConvGQR)
- [(*EMNLP 2022*) Saving Dense Retriever from Shortcut Dependency in Conversational Search](https://github.com/naver-ai/cs-shortcut)