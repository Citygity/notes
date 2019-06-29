# How to finetune on PyTorch

## pretalk -- about weight

When it comes to saving and loading models, there are three core functions to be familiar with:

1. [torch.save](https://pytorch.org/docs/stable/torch.html?highlight=save#torch.save): Saves a serialized object to disk. This function uses Python’s [pickle](https://docs.python.org/3/library/pickle.html) utility for serialization. Models, tensors, and dictionaries of all kinds of objects can be saved using this function.
2. [torch.load](https://pytorch.org/docs/stable/torch.html?highlight=torch%20load#torch.load): Uses [pickle](https://docs.python.org/3/library/pickle.html)’s unpickling facilities to deserialize pickled object files to memory. This function also facilitates the device to load the data into (see [Saving & Loading Model Across Devices](https://pytorch.org/tutorials/beginner/saving_loading_models.html#saving-loading-model-across-devices)).
3. [torch.nn.Module.load_state_dict](https://pytorch.org/docs/stable/nn.html?highlight=load_state_dict#torch.nn.Module.load_state_dict): Loads a model’s parameter dictionary using a deserialized *state_dict*. For more information on *state_dict*, see [What is a state_dict?](https://pytorch.org/tutorials/beginner/saving_loading_models.html#what-is-a-state-dict).

In PyTorch, the learnable parameters (i.e. weights and biases) of an `torch.nn.Module` model are contained in the model’s *parameters* (accessed with `model.parameters()`). A *state_dict* is simply a Python dictionary object that maps each layer to its parameter tensor. Note that only layers with learnable parameters (convolutional layers, linear layers, etc.) and registered buffers (batchnorm’s running_mean) have entries in the model’s *state_dict*. Optimizer objects (`torch.optim`) also have a *state_dict*, which contains information about the optimizer’s state, as well as the hyperparameters used.

### There are 4 situations for Saving & Loading Model for Inference

#### 1.Save/Load `state_dict` (Recommended)

**Save:**

```python
torch.save(model.state_dict(), PATH)
```

**Load:**

```python
model = TheModelClass(*args, **kwargs)
model.load_state_dict(torch.load(PATH))
model.eval()
```

When saving a model for inference, it is only necessary to **save the trained model’s learned parameters**. Saving the model’s *state_dict* with the `torch.save()` function will give you the most flexibility for restoring the model later, which is why it is the recommended method for saving models.

A common PyTorch convention is to save models using either a `.pt` or `.pth` file extension.

Remember that you must call `model.eval()` to set dropout and batch normalization layers to evaluation mode before running inference. Failing to do this will yield **inconsistent inference results**.

*Notice that the `load_state_dict()` function takes a dictionary object, NOT a path to a saved object. This means that you must deserialize the saved state_dict before you pass it to the `load_state_dict()` function. For example, you CANNOT load using `model.load_state_dict(PATH)`.*

#### 2. Save/Load Entire Model

**Save:**

```python
torch.save(model, PATH)
```

**Load:**

```python
# Model class must be defined somewhere
model = torch.load(PATH)
model.eval()
```

This save/load process uses the most intuitive syntax and involves the least amount of code. Saving a model in this way will save the entire module using Python’s [pickle](https://docs.python.org/3/library/pickle.html) module. **The disadvantage of this approach is that the serialized data is bound to the specific classes and the exact directory structure used when the model is saved.** The reason for this is because pickle does not save the model class itself. Rather, it saves a path to the file containing the class, which is used during load time. Because of this, your code can break in various ways when used in other projects or after refactors.

A common PyTorch convention is to save models using either a `.pt` or `.pth` file extension.

Remember that you must call `model.eval()` to set dropout and batch normalization layers to evaluation mode before running inference. Failing to do this will yield inconsistent inference results.

#### 3. Saving & Loading a General Checkpoint for Inference and/or Resuming Training

 **Save**:

```python
torch.save({
            'epoch': epoch,
            'model_state_dict': model.state_dict(),
            'optimizer_state_dict': optimizer.state_dict(),
            'loss': loss,
            ...
            }, PATH)
```

 **Load**:

```python
model = TheModelClass(*args, **kwargs)
optimizer = TheOptimizerClass(*args, **kwargs)

checkpoint = torch.load(PATH)
model.load_state_dict(checkpoint['model_state_dict'])
optimizer.load_state_dict(checkpoint['optimizer_state_dict'])
epoch = checkpoint['epoch']
loss = checkpoint['loss']

model.eval()
# - or -
model.train()
```

When saving a general checkpoint, to be used for either inference or resuming training, you must save more than just the model’s*state_dict*. It is important to also save the optimizer’s *state_dict*, as this contains buffers and parameters that are updated as the model trains. Other items that you may want to save are the epoch you left off on, the latest recorded training loss, external `torch.nn.Embedding` layers, etc.

To save multiple components, organize them in a dictionary and use `torch.save()` to serialize the dictionary. A common PyTorch convention is to save these checkpoints using the `.tar` file extension.

To load the items, first initialize the model and optimizer, then load the dictionary locally using `torch.load()`. From here, you can easily access the saved items by simply querying the dictionary as you would expect.

#### 4. Saving Multiple Models in One File

 **Save**:

```python
torch.save({
            'modelA_state_dict': modelA.state_dict(),
            'modelB_state_dict': modelB.state_dict(),
            'optimizerA_state_dict': optimizerA.state_dict(),
            'optimizerB_state_dict': optimizerB.state_dict(),
            ...
            }, PATH)
```

**Load**:

```python
modelA = TheModelAClass(*args, **kwargs)
modelB = TheModelBClass(*args, **kwargs)
optimizerA = TheOptimizerAClass(*args, **kwargs)
optimizerB = TheOptimizerBClass(*args, **kwargs)

checkpoint = torch.load(PATH)
modelA.load_state_dict(checkpoint['modelA_state_dict'])
modelB.load_state_dict(checkpoint['modelB_state_dict'])
optimizerA.load_state_dict(checkpoint['optimizerA_state_dict'])
optimizerB.load_state_dict(checkpoint['optimizerB_state_dict'])

modelA.eval()
modelB.eval()
# - or -
modelA.train()
modelB.train()
```

When saving a model comprised of multiple `torch.nn.Modules`, such as a GAN, a sequence-to-sequence model, or an ensemble of models, you follow the same approach as when you are saving a general checkpoint. In other words, save a dictionary of each model’s *state_dict* and corresponding optimizer. As mentioned before, you can save any other items that may aid you in resuming training by simply appending them to the dictionary.

A common PyTorch convention is to save these checkpoints using the `.tar` file extension.

To load the models, first initialize the models and optimizers, then load the dictionary locally using `torch.load()`. From here, you can easily access the saved items by simply querying the dictionary as you would expect.

Remember that you must call `model.eval()` to set dropout and batch normalization layers to evaluation mode before running inference. Failing to do this will yield inconsistent inference results. If you wish to resuming training, call `model.train()` to set these layers to training mode.