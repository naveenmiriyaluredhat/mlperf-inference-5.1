# MLPerf Inference Benchmarks for Natural Language Processing

This is the reference implementation for MLPerf Inference benchmarks for Natural Language Processing.

The chosen model is BERT-Large performing SQuAD v1.1 question answering task.

Please see the [new docs site](https://docs.mlcommons.org/inference/benchmarks/language/bert) for an automated way to run this benchmark across different available implementations and do an end-to-end submission with or without docker.

## Prerequisites

- [nvidia-docker](https://github.com/NVIDIA/nvidia-docker)
- Any NVIDIA GPU supported by TensorFlow or PyTorch

## Supported Models

| model | framework | accuracy | dataset | model link | model source | precision | notes |
| ----- | --------- | -------- | ------- | ---------- | ------------ | --------- | ----- |
| BERT-Large | TensorFlow | f1_score=90.874% | SQuAD v1.1 validation set | [from zenodo](https://zenodo.org/record/3733868) [from zenodo](https://zenodo.org/record/3939747) | [BERT-Large](https://github.com/google-research/bert), trained with [NVIDIA DeepLearningExamples](https://github.com/NVIDIA/DeepLearningExamples/tree/master/TensorFlow/LanguageModeling/BERT) | fp32 | |
| BERT-Large | PyTorch | f1_score=90.874% | SQuAD v1.1 validation set | [from zenodo](https://zenodo.org/record/3733896) | [BERT-Large](https://github.com/google-research/bert), trained with [NVIDIA DeepLearningExamples](https://github.com/NVIDIA/DeepLearningExamples/tree/master/TensorFlow/LanguageModeling/BERT), converted with [bert_tf_to_pytorch.py](bert_tf_to_pytorch.py) | fp32 | |
| BERT-Large | ONNX | f1_score=90.874% | SQuAD v1.1 validation set | [from zenodo](https://zenodo.org/record/3733910) | [BERT-Large](https://github.com/google-research/bert), trained with [NVIDIA DeepLearningExamples](https://github.com/NVIDIA/DeepLearningExamples/tree/master/TensorFlow/LanguageModeling/BERT), converted with [bert_tf_to_pytorch.py](bert_tf_to_pytorch.py) | fp32 | |
| BERT-Large | ONNX | f1_score=90.067% | SQuAD v1.1 validation set | [from zenodo](https://zenodo.org/record/3750364) | Fine-tuned based on the PyTorch model and converted with [bert_tf_to_pytorch.py](bert_tf_to_pytorch.py) | int8, symetrically per-tensor quantized without bias | See [MLPerf INT8 BERT Finetuning.pdf](MLPerf INT8 BERT Finetuning.pdf) for details about the fine-tuning process |
| BERT-Large | PyTorch | f1_score=90.633% | SQuAD v1.1 validation set | [from zenodo](https://zenodo.org/record/4792496) | Fine-tuned based on [Huggingface bert-large-uncased pretrained model](https://huggingface.co/bert-large-uncased) | int8, symetrically per-tensor quantized without bias | See README.md at Zenodo link for details about the fine-tuning process |

## Disclaimer
This benchmark app is a reference implementation that is not meant to be the fastest implementation possible.

## Automated command to run the benchmark via MLCFlow

Please see the [new docs site](https://docs.mlcommons.org/inference/benchmarks/language/bert/) for an automated way to run this benchmark across different available implementations and do an end-to-end submission with or without docker.

You can also do `pip install mlc-scripts` and then use `mlcr` commands for downloading the model and datasets using the commands given in the later sections.

### Download model through MLCFlow Automation

**Pytorch Framework**

```
mlcr get,ml-model,bert-large,_pytorch --outdirname=<path_to_download> -j
```

**Onnx Framework**

```
mlcr get,ml-model,bert-large,_onnx --outdirname=<path_to_download> -j
```

**TensorFlow Framework**

```
mlcr get,ml-model,bert-large,_tensorflow --outdirname=<path_to_download> -j
```

### Download dataset through MLCFlow Automation

**Validation**
```
mlcr get,dataset,squad,validation  --outdirname=<path_to_download> -j
```

**Calibration**
```
mlcr get,dataset,squad,_calib1 --outdirname=<path_to_download> -j
```

## Commands

Please run the following commands:

- `make setup`: initialize submodule, download datasets, and download models.
- `make build_docker`: build docker image.
- `make launch_docker`: launch docker container with an interaction session.
- `python3 run.py --backend=[tf|pytorch|onnxruntime|tf_estimator] --scenario=[Offline|SingleStream|MultiStream|Server] [--accuracy] [--quantized]`: run the harness inside the docker container. Performance or Accuracy results will be printed in console.

* ENV variable `MLC_MAX_NUM_THREADS` can be used to control the number of parallel threads issuing queries.

## Details

- SUT implementations are in [tf_SUT.py](tf_SUT.py), [tf_estimator_SUT.py](tf_estimator_SUT.py) and [pytorch_SUT.py](pytorch_SUT.py). QSL implementation is in [squad_QSL.py](squad_QSL.py).
- The script [accuracy-squad.py](accuracy-squad.py) parses LoadGen accuracy log, post-processes it, and computes the accuracy.
- Tokenization and detokenization (post-processing) are not included in the timed path.
- The inputs to the SUT are `input_ids`, `input_make`, and `segment_ids`. The output from SUT is `start_logits` and `end_logits` concatenated together.
- `max_seq_length` is 384.
- The script [tf_freeze_bert.py] freezes the TensorFlow model into pb file.
- The script [bert_tf_to_pytorch.py] converts the TensorFlow model into the PyTorch `BertForQuestionAnswering` module in [HuggingFace Transformers](https://github.com/huggingface/transformers) and also exports the model to [ONNX](https://github.com/onnx/onnx) format.

### Evaluate the accuracy through MLCFlow Automation
```bash
mlcr process,mlperf,accuracy,_squad --result_dir=<Path to directory where files are generated after the benchmark run>
```

Please click [here](https://github.com/mlcommons/inference/blob/master/language/bert/accuracy-squad.py) to view the Python script for evaluating accuracy for the squad dataset.

## Automated command for submission generation via MLCFlow

Please see the [new docs site](https://docs.mlcommons.org/inference/submission/) for an automated way to generate submission through MLCFlow. 

## Loadgen over the Network 

```
pip install mlc-scripts
```

The below MLC command will launch the SUT server

```
mlcr generate-run-cmds,inference --model=bert-99 --backend=pytorch  \
--mode=performance --device=cuda --quiet --test_query_count=1000 --network=sut
```

Once the SUT server is launched, the below command can be run on the loadgen node to do issue queries to the SUT nodes. In this command `-sut_servers` has just the localhost address - it can be changed to a comma-separated list of any hostname/IP in the network. 


```
mlcr generate-run-cmds,inference --model=bert-99 --backend=pytorch  --rerun \
--mode=performance --device=cuda --quiet --test_query_count=1000  \
--sut_servers,=http://localhost:8000 --network=lon
```

If you are not using MLC, just add `--network=lon` along with your normal run command on the SUT side.
On the loadgen node, add `--network=lon` option and `--sut_server <IP1> <IP2>` to the normal command to connect to SUT nodes at IP addresses IP1, IP2 etc. 

Loadgen over the network works for `onnxruntime` and `pytorch` backends.

## License

Apache License 2.0
