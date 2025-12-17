# DREAm: Dual-perspective Reasoning and Attribution-based Refinement for Conversational Query Rewriting

<img width="1515" height="1056" alt="image" src="https://github.com/user-attachments/assets/59dd401b-1205-4c4b-9d08-398f44bb0ef9" />

## Installation

## Data Pre-Processing

1. Download the passage collection

Two public datasets can be download from [QReCC]([url](https://github.com/apple/ml-qrecc)), [TopiOCQA]([url](https://github.com/McGill-NLP/topiocqa)),
We mainly evaluate our method using two types of retrievers: BM25 and ANCE on two Conversational QA benchmarks: TopiOCQA and QReCC.

进入 `./index/preprocess/qrecc` 和 `./index/preprocess/topiocqa` 分别下载 qrecc 和 topiocqa 的相关数据（包括 passage 等），然后用 processing.py 处理。

2. 得到 bm25 和 dense
为了得到 bm25 和 dense 检索库，需要分别执行 `./index/preprocess/bm25` 和 `./index/preprocess/dense` 来构造。


## Training

1. Data Preparing
下载统一处理过格式的 train dataset 和 test dataset：
`https://drive.google.com/drive/folders/1fFWixFDgrxwzyftlX34I7MnkbrpmcXJs?usp=sharing`
执行 `./code/data_processing/convert_lf_format.py` 转换为 LF 支持的训练格式。
修改 LLaMA-Factory 的 dataset_info.json 内容


2. Pruning
每次剪枝需要执行 `./utils/get_keyword.py` 和 `./utils/pruning.py`
using Pruner, trained on original (non-pruned) dataset.

3. SFT
`./code/train_eval/train_sft.sh`

## Inference

1. inference
`./code/train_eval/infer.py`

2. evaluation
分别执行 `./code/train_eval/search_sparse.py` 和 `./code/train_eval/search_dense.py` 进行检索，得到对应的分数。

3. 全流程执行
直接执行 `train_infer_eval.sh`

