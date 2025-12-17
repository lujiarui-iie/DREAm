# DREAm: Dual-perspective Reasoning and Attribution-based Refinement for Conversational Query Rewriting

<img width="1515" height="1056" alt="image" src="https://github.com/lujiarui-iie/DREAm/blob/main/assets/framework.jpg" />

## 📖 Introduction
This repository contains the implementation of **DREAm** (*Dual-perspective Reasoning and Attribution-based Refinement*). We propose a method for Conversational Query Rewriting that leverages dual-perspective reasoning and attribution-based refinement. The method is evaluated on **QReCC** and **TopiOCQA** benchmarks using both **sparse (BM25)** and **dense (ANCE)** retrievers.

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

Download the raw datasets (including passage collections) from their respective repositories:

- [QReCC]([url](https://github.com/apple/ml-qrecc))
- [TopiOCQA]([url](https://github.com/McGill-NLP/topiocqa))


进入 `./index/preprocess/qrecc` 和 `./index/preprocess/topiocqa` 分别下载 qrecc 和 topiocqa 的相关数据（包括 passage 等），然后用 processing.py 处理。

```bash
# For qrecc dataset
python ./index/preprocess/qrecc/qrecc_processing.py

# For topiocqa dataset
python ./index/preprocess/topiocqa/topiocqa_processing.py
```

#### Retrieval Indices Construction

To construct the retrieval indices, follow these steps:

为了得到 bm25 和 dense 检索库，需要分别执行 `./index/preprocess/bm25` 和 `./index/preprocess/dense` 来构造。
保存到对应的文件夹 `./index/{retrieval_type}_{dataset_name}`中

除此之外，对于 dense 还需要下载：
下载 `./index/ad-hoc-ance-msmarco`：https://huggingface.co/3ricL/ad-hoc-ance-msmarco/tree/main
下载 `./index/ance_checkpoint`：https://drive.google.com/drive/folders/13jKyMtpfg1nThOAZT3mSCCMXu95QK5GT?usp=sharing


## 🚀 Training

#### Data Preparing
下载统一处理过格式的 （qrecc 和 topiocqa）train dataset 和 test dataset：
`https://drive.google.com/drive/folders/1fFWixFDgrxwzyftlX34I7MnkbrpmcXJs?usp=sharing` 到 `./dataset`

下载 deepseek 生成的思考过程和重写：https://drive.google.com/drive/folders/1UYq2bZRoA_Jl_VuAqBqtVFPcseDcEGuO?usp=sharing 到 `./code/think_data`


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

#### SFT

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

## 🤝 Acknowledgments

We appreciate the open-source contributions from the following projects:

- [LLaMA-Factory]([url](https://github.com/hiyouga/LLaMA-Factory))
- [(*EMNLP 2024*) CHIQ: Contextual History Enhancement for Improving Query Rewriting in Conversational Search]([url](https://github.com/fengranMark/CHIQ))
- [ConvGQR]([url](https://github.com/fengranMark/ConvGQR))
- [cs-shortcut]([url](https://github.com/naver-ai/cs-shortcut))