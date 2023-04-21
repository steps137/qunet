# QuNet

[![Python 3.7+](https://img.shields.io/badge/python-3.7+-blue.svg)](https://www.python.org/downloads/release/python-370/)
[![PyPI version](https://badge.fury.io/py/torchinfo.svg)](https://badge.fury.io/py/torchinfo)


Working with deep learning models.
* Large set of custom modules for neural networks (MLP, CNN, Transformer, etc.)
* Trainer class for training the model.

## Install

```
pip install qunet
```

## Usage

```python
from qunet import MLP, Data, Trainer, CosScheduler
```
To work with DL, **data** and **model** are required. 

## Model

In order for `Trainer` to work with a model, it must be a class (successor of nn.Module) with methods:
* The `forward` function takes input x and returns y.
These can be tensors or tuples (lists) of tensors.
* The `metrics` function takes  (x, y_true, y_pred) and returns the model's scalar loss and quality metric: 
tensor (B,1) for one metric (accuracy, for example) or (B,n) for n quality metrics.

For example, a 1D linear regression  y=f(x) with mse-loss and metric as |y_pred-y_true|, model looks like:
```python
class Model(nn.Module):
    def __init__(self):        
        super().__init__() 
        self.fc = nn.Linear( 1, 1 )

    def forward(self, x):                            # (B,1)
        return self.fc(x)                            # (B,1)

    def metrics(self, x, y_pred, y_true)             # (B,1) (B,1)  (B,1)        
        loss   = (y_pred - y_true).pow(2).mean()     # ()     scalar!
        errors = torch.abs(y_pred.detach()-y_true)   # (B,1)  one metric
        return loss, errors                          # ()  (B,1)
```

## Data

The `Data` - training or validation data class. It can be overridden or the standard `DataLoader` can be used.
For the `Trainer` to work in the `__next__` iterator, the `Data` instance must return an `X,Y` tuple, where
* X - tensor or tuple (list) of tensors for model input,
* Y - tensor or tuple (list) of tensors for model target values.

For example, let's create training data in which two tensors X1,X2 are the input of the model and one tensor Y is the output (target):
```python    
    data_trn = Data(dataset=( (X1,X2), Y ),  shuffle=True)  
```        
The data minibatch tuple (X,Y) is used in the Trainer as follows:
```python
    for b (X,Y_true) in enumerate(data):  # при обучении
        X, Y_true = to_device(X), to_device(Y_true)            
        Y_pred = model(X)
        loss, score = model.metrics(X, Y_pred, Y_true)
```
So dataset is a list or tuple of two elements (input and target).
Each element can be a tensor or a list (tuple) of tensors.
All tensors in the dataset are assumed to have the same length (by first index).

You can also use the standard DataLoader with Trainer:
```python
    from torchvision            import datasets
    from torchvision.transforms import ToTensor 
    from torch.utils.data       import DataLoader

    mnist = datasets.MNIST(root='data', train=True,  transform=ToTensor(), download=True)
    data_trn  = DataLoader(dataset=mnist, batch_size=1024, shuffle=True)
```

## Trainer

The Trainer is given the model, training and validation data.
Using the `set_optimizer` function, the optimizer is set.
After that, the `run` function is called:
```
trainer = Trainer(model, data_trn, data_val)
trainer.set_optimizer( torch.optim.SGD(model.parameters(), lr=1e-2) )
trainer.run(epochs=100, pre_val=True, period_plot = 10)
```
You can add different training schedulers, customize the output of training graphs, manage the storage of the best models and checkpoints, and much more.

## Visualization of the training process

<img src="img/loss.png">

## Using Schedules

## Working with large models

## Model state visualization

## Examples

* Regression_1D - visualization of changes in model parameters
* Interpolation_F(x)
* MNIST
* Vanishing gradient

## Versions

* 0.0.4 - fixed version for competition IceCube (kaggle)