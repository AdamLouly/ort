This repository contains below libraries. They can be installed independent of each other.
* [ort_moe](#mixture-of-experts): Mixture of Experts implementation in PyTorch
* [torch_ort](#accelerate-pytorch-models-with-onnx-runtime): ONNX Runtime package that accelerates PyTorch models

# Mixture Of Experts

Mixture of Experts layer implementation is available in the [ort_moe](ort_moe) folder. 
- [ort_moe/docs/moe.md](ort_moe/docs/moe.md) provides brief overview of the implementation.
- A simple MoE tutorial is provided [here](ort_moe/docs/tutorials/moe_tutorial.py).
- Note: ONNX Runtime (following pre-requisites) is not required to run the MoE layer. It is intergrated in stand-alone Pytorch.

## Build MoE
```
cd ort_moe
pip install build # Install PyPA build
python -m build
```

# Accelerate PyTorch models with ONNX Runtime

ONNX Runtime for PyTorch accelerates PyTorch model training using ONNX Runtime.

It is available via the torch-ort python package.

This repository contains the source code for the package, as well as instructions for running the package.

## Pre-requisites

You need a machine with at least one NVIDIA or AMD GPU to run ONNX Runtime for PyTorch.

You can install and run torch-ort in your local environment, or with [Docker](torch_ort/docker/README.md).

## Install in a local Python environment

### Default dependencies

By default, torch-ort depends on PyTorch 1.9.0, ONNX Runtime 1.9.0 and CUDA 10.2.

1. Install CUDA 10.2

2. Install CuDNN 7.6

3. Install torch-ort

    - `pip install torch-ort`

4. Run post-installation script for ORTModule

    - `python -m torch_ort.configure`

Get install instructions for other combinations in the `Get Started Easily` section at <https://www.onnxruntime.ai/> under the `Optimize Training` tab.

## Verify your installation

1. Clone this repo

    - `git clone git@github.com:pytorch/ort.git`

2. Install extra dependencies

    - `pip install wget pandas sklearn transformers`

3. Run the training script

    - `python ./ort/tests/bert_for_sequence_classification.py`

## Add ONNX Runtime for PyTorch to your PyTorch training script

```python
from torch_ort import ORTModule
model = ORTModule(model)

# PyTorch training script follows
```

## Usage of FusedAdam

```python
import torch
from torch_ort.optim import FusedAdam

class NeuralNet(torch.nn.Module):
    ...

# Only supports GPU Currently.
device = "cuda"
model = NeuralNet(...).to(device)

ort_fused_adam_optimizer = FusedAdam(
    model.parameters(), lr=1e-3, betas=(0.9, 0.999), weight_decay=0.01, eps=1e-8
)

loss = model(...).sum()
loss.backward()

ort_fused_adam_optimizer.step()
ort_fused_adam_optimizer.zero_grad()

```
For detailed documentation see [FusedAdam](https://github.com/microsoft/onnxruntime/blob/master/orttraining/orttraining/python/training/optim/fused_adam.py#L25)

For a full working example see [FusedAdam Test Example](https://github.com/pytorch/ort/blob/main/torch_ort/tests/torch-ort_test_api.py) 

## Usage of LoadBalancingDistributedSampler

```python
import torch
from torch.utils.data import DataLoader 
from torch_ort.utils.data import LoadBalancingDistributedSampler

class MyDataset(torch.utils.data.Dataset):
   ...
   
def collate_fn(data): 
    ...
    return samples, label_list 

samples = [...] 
labels = [...] 

dataset = MyDataset(samples, labels) 

data_sampler = sampler.LoadBalancingDistributedSampler( 

    dataset, complexity_fn=complexity_fn, world_size=2, rank=0, shuffle=False 

) 

train_dataloader = DataLoader(dataset, batch_size=2, sampler=data_sampler, collate_fn=collate_fn) 

for batched_data, batched_label in train_dataloader: 
    optimizer.zero_grad() 
    loss = loss_fn(model(batched_data) , batched_labels) 
    loss.backward() 
    optimizer.step() 
    
```
For detailed documentation see [LoadBalancingDistributedSampler](https://github.com/microsoft/onnxruntime/blob/master/orttraining/orttraining/python/training/utils/data/sampler.py#L37)

For a full working example see [LoadBalancingDistributedSampler Test Example](https://github.com/microsoft/onnxruntime/blob/master/orttraining/orttraining/python/training/utils/data/sampler.py#L37)

## Samples

To see torch-ort in action, see https://github.com/microsoft/onnxruntime-training-examples, which shows you how to train the most popular HuggingFace models.

## License

This project has an MIT license, as found in the [LICENSE](LICENSE) file.
