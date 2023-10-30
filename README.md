# LearningPyTorch

## Installing PyTorch and tutorials

### Installation
https://www.cherryservers.com/blog/how-to-install-pytorch-ubuntu

### Tutorials
1. web pages
- tensors : https://pytorch.org/tutorials/beginner/blitz/tensor_tutorial.html
- torch.autograd : https://pytorch.org/tutorials/beginner/blitz/autograd_tutorial.html
- neural network https://pytorch.org/tutorials/beginner/blitz/neural_networks_tutorial.html
- training a classifier : https://pytorch.org/tutorials/beginner/blitz/cifar10_tutorial.html

2. downloaded tutos
see the downloaded tutos on pytorch_env/tutorials

3. codes
see the corresponding codes on pytorch_env/code

4. readme file
see the tutos on readme (not all)

## Tensors

Tensors = array or matrices 

### imports
```python
import torch
import numpy as np
```
### tensor initialization
```python
# directly from data
data = [[1, 2], [3, 4]]
x_data = torch.tensor(data)

# from a NumPy array
np_array = np.array(data)
x_np = torch.from_numpy(np_array)

# from another tensor
x_ones = torch.ones_like(x_data) # retains the properties of x_data
print(f"Ones Tensor: \n {x_ones} \n")

x_rand = torch.rand_like(x_data, dtype=torch.float) # overrides the datatype of x_data
print(f"Random Tensor: \n {x_rand} \n")

# with random or constant values
# shape est un tuple qui représente les dimensions du tenseur
shape = (2, 3,)
rand_tensor = torch.rand(shape)
ones_tensor = torch.ones(shape)
zeros_tensor = torch.zeros(shape)

print(f"Random Tensor: \n {rand_tensor} \n")
print(f"Ones Tensor: \n {ones_tensor} \n")
print(f"Zeros Tensor: \n {zeros_tensor}")
```
### tensor attributes
```python
tensor = torch.rand(3, 4)

print(f"Shape of tensor: {tensor.shape}")
print(f"Datatype of tensor: {tensor.dtype}")
print(f"Device tensor is stored on: {tensor.device}")

# out
# Shape of tensor: torch.Size([3, 4])
# Datatype of tensor: torch.float32
# Device tensor is stored on: cpu
```

### tensor operations
Each of them can be run on the GPU (at typically higher speeds than on a CPU). If you’re using Colab, allocate a GPU by going to Edit > Notebook Settings.

```python
# We move our tensor to the GPU if available
if torch.cuda.is_available():
  tensor = tensor.to('cuda')
  print(f"Device tensor is stored on: {tensor.device}")
```

```python
# standard numpy-like indexing and slicing
tensor = torch.ones(4, 4)
tensor[:,1] = 0
print(tensor)
# out
# tensor([[1., 0., 1., 1.],
#        [1., 0., 1., 1.],
#        [1., 0., 1., 1.],
#        [1., 0., 1., 1.]])

# joining tensors
t1 = torch.cat([tensor, tensor, tensor], dim=1)
print(t1)
'''
out
tensor([[1., 0., 1., 1., 1., 0., 1., 1., 1., 0., 1., 1.],
        [1., 0., 1., 1., 1., 0., 1., 1., 1., 0., 1., 1.],
        [1., 0., 1., 1., 1., 0., 1., 1., 1., 0., 1., 1.],
        [1., 0., 1., 1., 1., 0., 1., 1., 1., 0., 1., 1.]])
'''

# multiplying tensors
# This computes the element-wise product
print(f"tensor.mul(tensor) \n {tensor.mul(tensor)} \n")
# Alternative syntax:
print(f"tensor * tensor \n {tensor * tensor}")
# computes the matrix multiplication between two tensors
print(f"tensor.matmul(tensor.T) \n {tensor.matmul(tensor.T)} \n")
# Alternative syntax:
print(f"tensor @ tensor.T \n {tensor @ tensor.T}")


# in-place operations : Operations that have a _ suffix are in-place
print(tensor, "\n")
tensor.add_(5)
print(tensor)
# In-place operations save some memory, but can be problematic when computing derivatives because of an immediate loss of history. Hence, their use is discouraged.
```

### Bridge with NumPy
```python
# tensor to NumPy array
t = torch.ones(5)
print(f"t: {t}")
n = t.numpy()
print(f"n: {n}")
# A change in the tensor reflects in the NumPy array.
t.add_(1)
print(f"t: {t}")
print(f"n: {n}")

# NumPy array to Tensor
n = np.ones(5)
t = torch.from_numpy(n)
# Changes in the NumPy array reflects in the tensor.
np.add(n, 1, out=n)
print(f"t: {t}")
print(f"n: {n}")
```

## torch.aurograd
### Quezako
Les réseaux neurals (NN) sont un ensemble de fonctions imbriquées qui sont exécutés sur 
certaines données d'entrée. Ces fonctions sont définies par des paramètres 
(composé de poids et de biais), qui sont stockés dans PyTorch tenseurs.

La formation d'un NN se fait en deux étapes:

- Propagation vers l'avant (=forward propagation): Dans l'alouement, le NN fait sa meilleure 
hypothèse sur la sortie correcte. Il exécute les données d'entrée par l'intermédiaire 
de chacun de ses fonctions pour faire cette supposition.

- Propagation vers l'arrière (= backward propagation: Dans le rétroprop, le NN ajuste ses 
paramètres proportionné à l'erreur dans sa supposition. Il le fait en traversant en arrière 
à partir de la sortie, en recueillant les dérivées de l'erreur avec le respect des paramètres
des fonctions (gradients),  et l'optimisation les paramètres utilisant la descente de gradient.
### Usage in PyTorch
#### initialisation
```python
import torch
# load a pretrained resnet18 model
from torchvision.models import resnet18, ResNet18_Weights
model = resnet18(weights=ResNet18_Weights.DEFAULT)
# create a random data tensor to represent a singel image with 3 channel. height&width 64
data = torch.rand(1, 3, 64, 64)
# label initialized to some random values
# label in pretrained models has shape (1,1000)
labels = torch.rand(1, 1000)
```
#### forward pass 
In forward prop, the NN (=Neutral Network) makes its best guess about the correct output.
It runs the input data through each of its functions to make this guess.
```python
prediction = model(data) # forward pass
```
#### error calculation and backward pass
We use the model’s prediction and the corresponding label to calculate the error.
In backprop the NN adjusts its parameters proportionate to the error in its guess.
Autograd then calculates and stores the gradients for each model parameter in the parameter’s .grad attribute.
```python
loss = (prediction - labels).sum()
loss.backward() # backward pass
```

#### optimizer
 load an optimizer, in this case SGD with a learning rate of 0.01 and momentum of 0.9. 
 We register all the parameters of the model in the optimizer.
```python
optim = torch.optim.SGD(model.parameters(), lr=1e-2, momentum=0.9)
```

#### initialiser la descente de graident
The optimizer adjusts each parameter by its gradient stored in .grad
```python
optim.step() #gradient descent
```

## Neural network
- Neural networks can be constructed using the torch.nn package.
- Now that you had a glimpse of autograd, nn depends on autograd to define models and 
differentiate them. An nn.Module contains layers, and a method forward(input) that returns 
the output.
- For example, look at this network that classifies digit images:
![neural network expl](https://pytorch.org/tutorials/_images/mnist.png)
- It is a simple feed-forward network. It takes the input, feeds it through several layers one after the other, and 
then finally gives the output.
- A typical training procedure for a neural network is as follows:
  - Define the neural network that has some learnable parameters (or weights)
  - Iterate over a dataset of inputs
  - Process input through the network
  - Compute the loss (how far is the output from being correct)
  - Propagate gradients back into the network’s parameters
  - Update the weights of the network, typically using a simple update rule: weight = weight - learning_rate * gradient

### Define the network
```python
import torch
import torch.nn as nn
import torch.nn.functional as F


class Net(nn.Module):

    def __init__(self):
        super(Net, self).__init__()
        # 1 input image channel, 6 output channels, 5x5 square convolution
        # kernel
        self.conv1 = nn.Conv2d(1, 6, 5)
        self.conv2 = nn.Conv2d(6, 16, 5)
        # an affine operation: y = Wx + b
        self.fc1 = nn.Linear(16 * 5 * 5, 120)  # 5*5 from image dimension
        self.fc2 = nn.Linear(120, 84)
        self.fc3 = nn.Linear(84, 10)

    def forward(self, x):
        # Max pooling over a (2, 2) window
        x = F.max_pool2d(F.relu(self.conv1(x)), (2, 2))
        # If the size is a square, you can specify with a single number
        x = F.max_pool2d(F.relu(self.conv2(x)), 2)
        x = torch.flatten(x, 1) # flatten all dimensions except the batch dimension
        x = F.relu(self.fc1(x))
        x = F.relu(self.fc2(x))
        x = self.fc3(x)
        return x


net = Net()
print(net)

'''
Out:
Net(
  (conv1): Conv2d(1, 6, kernel_size=(5, 5), stride=(1, 1))
  (conv2): Conv2d(6, 16, kernel_size=(5, 5), stride=(1, 1))
  (fc1): Linear(in_features=400, out_features=120, bias=True)
  (fc2): Linear(in_features=120, out_features=84, bias=True)
  (fc3): Linear(in_features=84, out_features=10, bias=True)
)
'''
```
- You just have to define the forward function, and the backward function (where gradients are computed) is 
automatically  defined for you using autograd. You can use any of the Tensor operations in the forward function.
- The learnable parameters of a model are returned by net.parameters()

```python
params = list(net.parameters())
print(len(params))
print(params[0].size())  # conv1's .weight

'''
Out:
10
torch.Size([6, 1, 5, 5])
'''
```

Let’s try a random 32x32 input. Note: expected input size of this net (LeNet) is 32x32. 
To use this net on the MNIST dataset, please resize the images from the dataset to 32x32.

```python
input = torch.randn(1, 1, 32, 32)
out = net(input)
print(out)

'''
Out:
tensor([[ 0.1453, -0.0590, -0.0065,  0.0905,  0.0146, -0.0805, -0.1211, -0.0394,
         -0.0181, -0.0136]], grad_fn=<AddmmBackward0>)
'''
```

Zero the gradient buffers of all parameters and backprops with random gradients:
```python
net.zero_grad()
out.backward(torch.randn(1, 10))
```

#### note
- torch.nn only supports mini-batches. The entire package only supports inputs that are a mini-batch of samples, and not
a single sample
- For expl, nn.Conv2d will take a 4D Tensor of nSamples x nChannels x Height x Width
- If you have a single sample, just use input.unsqueeze(0) to add a fake batch dimension.

#### recap
- torch.Tensor - A multi-dimensionnal array with support for autograd operations like backward(). Also holds the gradient
w.r.t the tensor
- nn.Module - Neutral Network module. Convenient way of encapsuling parameters, with helpers for moving them to GPU,
exporting, loading, etc.
- nn.Parameter - A kind of Tensor, that is automatically registered as a parameter when assigned as an attribute to a 
module
- autograd.Function - Implements forward and backward definitions of an autograd operation. Every Tensor operation
creates at least a single Function node that created a Tensor and encodes its history.

#### At this point, we covered
- Defining a neural network
- Processing inputs and calling backward

#### Still Left
- Computing the loss
- Updateing the weights of the network

### Loss Function
- A loss function takes the (output, target) pair of inputs, and computes a value that estimates how far away the output 
is from the target.
- There are several different loss functions under the nn package . A simple loss is: nn.MSELoss which computes the 
mean-squared error between the output and the target.

```python
output = net(input)
target = torch.randn(10)  # a dummy target, for example
target = target.view(1, -1)  # make it the same shape as output
criterion = nn.MSELoss()

loss = criterion(output, target)
print(loss)

'''
Out:
tensor(1.3619, grad_fn=<MseLossBackward0>)
'''
```

So, when we call loss.backward(), the whole graph is differentiated w.r.t. the neural net parameters, and all Tensors 
in the graph that have requires_grad=True will have their .grad Tensor accumulated with the gradient.

For illustration, let us follow a few steps backward:

```python
print(loss.grad_fn)  # MSELoss
print(loss.grad_fn.next_functions[0][0])  # Linear
print(loss.grad_fn.next_functions[0][0].next_functions[0][0])  # ReLU
'''
Out:
<MseLossBackward0 object at 0x7fea21319e40>
<AddmmBackward0 object at 0x7fea2131ba90>
<AccumulateGrad object at 0x7fea2131b3a0>
'''
```

### Backprop
To backpropagate the error all we have to do is to loss.backward(). You need to clear the existing gradients though, 
else gradients will be accumulated to existing gradients.

Now we shall call loss.backward(), and have a look at conv1’s bias gradients before and after the backward.

```python
net.zero_grad()     # zeroes the gradient buffers of all parameters

print('conv1.bias.grad before backward')
print(net.conv1.bias.grad)

loss.backward()

print('conv1.bias.grad after backward')
print(net.conv1.bias.grad)

'''
Out:
conv1.bias.grad before backward
None
conv1.bias.grad after backward
tensor([ 0.0081, -0.0080, -0.0039,  0.0150,  0.0003, -0.0105])
'''
```



The neural network package contains various modules and loss functions that form the building blocks of deep neural 
networks. A full list with documentation is here : https://pytorch.org/tutorials/beginner/blitz/neural_networks_tutorial.html#update-the-weights

### Update the weights
The simplest update rule used in practice is the Stochastic Gradient Descent (SGD):
weight = weight - learning_rate * gradient

```python
learning_rate = 0.01
for f in net.parameters():
    f.data.sub_(f.grad.data * learning_rate)
```

However, as you use neural networks, you want to use various different update rules such as SGD, Nesterov-SGD, Adam, 
RMSProp, etc. To enable this, we built a small package: torch.optim that implements all these methods. 
Using it is very simple:

```python
import torch.optim as optim

# create your optimizer
optimizer = optim.SGD(net.parameters(), lr=0.01)

# in your training loop:
optimizer.zero_grad()   # zero the gradient buffers
output = net(input)
loss = criterion(output, target)
loss.backward()
optimizer.step()    # Does the update
```

- note
Observe how gradient buffers had to be manually set to zero using optimizer.zero_grad(). This is because gradients are 
accumulated as explained in the Backprop section.

## Training a classifier 
Training an image classifier
We will do the following steps in order:
1. Load and normalize the CIFAR10 training and test datasets using torchvision
2. Define a Convolutional Neural Network
3. Define a loss function
4. Train the network on the training data
5. Test the network on the test data

see the tutorial directly
