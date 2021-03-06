# Custom Dataloader with Pytorch


## Objective

The aim here is to write a neural network that can take 2 inputs:

  1. an image from MNIST dataset, and
  2. a random number between 0 and 9

and gives two outputs:

  1. the "number" that was represented by the MNIST image, and
  2. the "sum" of this number with the random number that was generated and sent as the input to the network

   <p align="center" style="padding: 10px">
    <img alt="Forwarding" src="https://github.com/gokul-pv/Custom-DataLoader-Pytorch/blob/main/Plots/NN.png?raw=true" width =500>
    <br>
    <em style="color: grey">Figure 1.a : Sample neural network</em>
  </p> 


### Creating Custom Dataset
Dataset is created by extending Pytorch Dataset class using standard MNIST dataset and randomly generated numbers .
The MNIST dataset, Modified National Institute of Standards and Technology database, is a famous dataset of handwritten digits that is commonly used for training image processing systems for machine learning. This dataset consists of 70,000 1x28x28  images  of hand written digits from 0-9 with the following split:

  60,000 training images
  10,000 testing images

In addition to this, custom dataset has another input, which is a random number . Random number generated with torch.randint generator is squeezed to match dimension of image labels .Dataset has two outputs also . One is the label of hand written image , second output is generated by adding label and random number in corresponding sample .
Dataloader is defined separately for both train and test data to get data as batches for forward pass. Fixed batch size as 128 in data loader so that each input to network will be of 128x1x28x28 format.

### Defining layers and implementing forward method 

Model architecture has seven convolutional layers and three fully connected layers(refer to Figure 1.b) .The forward() method is the actual network transformation. The forward method is the mapping that maps an input tensor to a prediction output tensor. our input tensor is transformed as we move through the convolutional layers.  The first convolutional layer has a convolutional operation, followed by a relu activation operation whose output is then passed to second convolutional layer .Output of second convolutional layer followed by relu activation function is passed to a max pooling operation with kernel size 3x3 and stride 1x1.We follow other convolution layers also in a similar fashion. We started with a 128 x1 x 28 x 28 input tensor. This gives a single color channel, batch of 28 x 28 image, and by the time our tensor arrives at the last convolution layer, the dimensions have changed. The height and width dimensions have been reduced by the convolution and pooling operations. Last convolution layer output is reshaped into (-1,10) tensor and passed through log_softmax activation function .This is because our loss function nn.NLLLoss expects log probabilities as input instead of probabilities. Obtaining log-probabilities in a neural network is easily achieved by adding a LogSoftmax layer in the last layer of your network. This log_softmax output is y1, which is one of the model outputs. This will be (128x 10) shape tensor. Argmax function is applied to obtain maximum value index from y1 output which is later one hot encoded and concatenated with second output (random number x2) and fed to linear layer. First linear layer has matrix multiplication operation followed by relu activation which further fed to next two linear layer. Output of final linear layer is passed through log_softmax . 

   <p align="center" style="padding: 10px">
    <img alt="Forwarding" src="https://github.com/gokul-pv/Custom-DataLoader-Pytorch/blob/main/Plots/model_architecture.PNG?raw=true" width =1000>
    <br>
    <em style="color: grey">Figure 1.b : Model Architecture</em>
  </p> 


### Training 
For better performance all input and output tensors moved to GPU with to('cuda') method
During the entire training process, we do as many epochs as necessary to reach our desired level of accuracy. With this, we have the following steps:

  1) Get batch from the training set.<br>
  2) Pass batch to network.<br>
  3) Calculate the loss (difference between the predicted values and the true values).<br>
  4) Calculate the gradient of the loss function w.r.t the network's weights.<br>
  5) Update the weights using the gradients to reduce the loss.<br>
  6) Repeat steps 1-5 until one epoch is completed.<br>
  7) Repeat steps 1-6 for as many epochs required to reach the minimum loss.<br>

Finally, after we call the backward() method on our loss tensor, we know the gradients will be calculated and added to the grad attributes of our network's parameters. For this reason, we need to zero out these gradients. We can do this with a method called zero_grad() that comes with the optimizer.
Selected loss function is NLL loss(negative log likelihood loss).It is useful to train a classification problem with C classes. nn.NLLLoss expects log probabilities as input instead of probabilities. Obtaining log-probabilities in a neural network is easily achieved by adding a LogSoftmax layer in the last layer of your network. Figure 1.c shows training logs
   <p align="center" style="padding: 10px">
    <img alt="Forwarding" src="https://github.com/gokul-pv/Custom-DataLoader-Pytorch/blob/main/Plots/training%20logs.png?raw=true" width =1500>
    <br>
    <em style="color: grey">Figure 1.c : Training Log</em>
  </p> 


### Model Evaluation 
Model evaluation metric used is accuracy. Loss is decreasing in each epoch. Training accuracy and test accuracy are comparable.Please refer to figure 1.d of decreasing loss
   <p align="center" style="padding: 10px">
    <img alt="Forwarding" src="https://github.com/gokul-pv/Custom-DataLoader-Pytorch/blob/main/Plots/Training_loss.png?raw=true" width =500>
    <br>
    <em style="color: grey">Figure 1.d : Decreasing training loss</em>
  </p> 

