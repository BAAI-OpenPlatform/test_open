license: [Apache License 2.0](https://model.baai.ac.cn/use-agreement)


# AquilaChat-7B

## 简介/Overview
Aquila语言大模型在技术上继承了GPT-3、LLaMA等的架构设计优点，替换了一批更高效的底层算子实现、重新设计实现了中英双语的tokenizer，升级了BMTrain并行训练方法，在Aquila的训练过程中实现了比Magtron+DeepSpeed zero-2将近８倍的训练效率。Aquila语言大模型是在中英文高质量语料基础上从０开始训练的，通过数据质量的控制、多种训练的优化方法，实现在更小的数据集、更短的训练时间，获得比其它开源模型更优的性能。也是首个支持中英双语知识、支持商用许可协议、符合国内数据合规需要的大规模开源语言模型。

The Aquila language model inherits the architectural design advantages of GPT-3 and LLaMA, replacing a batch of more efficient underlying operator implementations and redesigning the tokenizer for Chinese-English bilingual support. It upgrades the BMTrain parallel training method, achieving nearly 8 times the training efficiency of Magtron+DeepSpeed ZeRO-2 in the training process of Aquila. The Aquila language model is trained from scratch on high-quality Chinese and English corpora. Through data quality control and various training optimization methods, it achieves better performance than other open-source models with smaller datasets and shorter training times. It is also the first large-scale open-source language model that supports Chinese-English-Knowledge, commercial licensing, and complies with domestic data regulations.
  
AquilaChat-7B是在Aquila-7B模型的基础上，进行SFT微调后的支持中英双语的对话式语言模型。模型在一台8卡Nvidia A100上训练8小时，总共对15万条数据训练了3个epoch。AquilaChat-7B模型由智源研究院研发，其在主流评测数据集上的评测结果如下

AquilaChat-7B is a conversational language model that supports Chinese-English dialogue. It is based on the Aquila-7B model and fine-tuned using SFT. The model was trained on an 8-card Nvidia A100 for 8 hours, and a total of 150,000 lines of data were trained for 3 epochs. AquilaChat-7B model was developed by Beijing Academy of Artificial Intelligence. The evaluation results on mainstream benchmark datasets are as follows:

| 名称/Name | MMLU_Chinese_EM | CLUE-EM |MMLU-EM| BoolQ-EM| TruthfulQA-EM |IMDB-EM| RAFT-EM|
|  -----  | ----  | -----  | ----  | -----  | ----  | -----  | -----  |
| [AcuilaChat-7B](https://model.baai.ac.cn/model-detail/xxxxx) | 0.292 | 0.385|0.269 | 0.731|0.347 |0.939| 0.443|
| [BiLLa-7B-LLM](https://model.baai.ac.cn/model-detail/xxxxx) | 0.279 | 0.374|0.257 | 0.76|0.205 |0.864| 0.514|
| [Ziya-LLaMA-13B-v1](https://model.baai.ac.cn/model-detail/xxxxx) | 0.273 | 0.404|0.406 | 0.786|0.284 |0.762| 0.191|

您可以在[FlagEval基础模型评测平台](https://flageval.baai.ac.cn/#/home) 查看更多评测指标

You can view [FlagEval Model Evaluation Platform](https://flageval.baai.ac.cn/#/home) for more details



## 模型细节/Model details

我们使用了一系列更高效的底层算子来辅助模型训练，其中包括参考[flash-attention](https://github.com/HazyResearch/flash-attention)的方法并替换了一些中间计算，同时还使用了RMSNorm。在此基础上，我们应用了[BMtrain](https://github.com/OpenBMB/BMTrain)技术进行轻量化的并行训练，该技术采用了数据并行、ZeRO（零冗余优化器）、优化器卸载、检查点和操作融合、通信-计算重叠等方法来优化模型训练过程。

Aquila模型所采用的tokenizer是由我们从头开始训练的，支持中英双语。与其他tokenizer的参数对比见下图：

We used a series of more efficient low-level operators to assist with model training, including methods referenced from [flash-attention](https://github.com/HazyResearch/flash-attention) and replacing some intermediate calculations, as well as using RMSNorm. Building upon this foundation, we applied the [BMtrain](https://github.com/OpenBMB/BMTrain) for lightweight parallel training, which utilizes methods such as data parallelism, ZeRO (zero redundancy optimizer), optimizer offloading, checkpoint and operation fusion, and communication-computation overlap to optimize the model training process.

The tokenizer used in the Aquila model was trained from scratch by us and supports both English and Chinese. The parameters of this tokenizer are compared to those of other tokenizers in the figure below:

| 模型/Model | 词表大小 | 说明 |英文平均tokens量| 中文平均tokens量|代码平均tokens量  |
|  -----  | ----  | -----  | ----  | -----  | ----  | 
| gpt2 | 50527 | bpe|1717.2914 | 1764.7128|2323.8167 |
| llama | 32000 | sp(bpe)|1805.6541| 1257.9891|1970.3644 |
| gpt2_new_100k | 100000 | bpe|1575.7418 | 477.4393|1679.7736 |


## 训练数据集/Training data 

我们采用了一系列高质量中英文数据集来训练和微调我们的对话语言模型，并且在不断更新迭代

We used a series of high-quality Chinese and English datasets to train and fine-tune our conversational language model, and continuously updated it through iterations.

![Screenshot](img/data.jpg)


## 快速使用/Quick start

### 推理/Inference

```python
import torch
from flagai.auto_model.auto_loader import AutoLoader
from flagai.model.predictor.predictor import Predictor
from flagai.data.tokenizer import Tokenizer

model_info = "Aquila-7b-sft-10m-1"
model_dir = "./aquila-7b-45500-sft-10m-1"
prompt = "今天吃什么"

tokenizer = Tokenizer.from_pretrained("llama-30b-en", 
                                      cache_dir="./gpt2_new_100k/")

loader = AutoLoader("lm", model_name="llama-7b-en", 
                    only_download_config=False, 
                    use_cache=True, 
                    fp16=True,
                    model_dir=model_dir)

model = loader.get_model()
model.eval()
vocab = tokenizer.get_vocab()
id2word = {v:k for k, v in vocab.items()}
predictor = Predictor(model, tokenizer)
with torch.no_grad():
    prompt = "#用户#" + prompt + " " + "#ai助手#"
    model_in = "[CLS]" + prompt
    out, tokens, probs = predictor.predict_generate_randomsample(prompt, 
                                                    out_max_length=200, 
                                                    )
```

### 微调/Fine-tuning

```python

```



 ## 源码/Source Code

源代码可以进入GitHub[FlagAI仓库](https://github.com/THUDM/GLM) 查看. 同时我们也支持[Huggingface平台](hflink)

The original code can be found in Github [FlagAI](https://github.com/THUDM/GLM) repository, and we also support [Huggingface](hflink)



## 证书/License

Aquila-7B开源模型使用 [智源Aquila系列模型许可协议](linkhere), 原始代码基于[Apache Licence 2.0](link)


Aquila-7B open-source model is licensed under [ BAAI Aquila Model Licence Agreement](linkhere). The source code is under [Apache Licence 2.0](link)