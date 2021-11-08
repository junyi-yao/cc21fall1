# Beginner's Walk-through of Deep Learning in R

Dashansh Prajapati and Vijay S Kalmath




```r
library(dplyr)
library(ggplot2)
library(keras)
library(tensorflow)
library(raster)
```

## Introduction

Deep Learning a term whenever heard of in a conversation, most of the time accompanies the word “Python”. This shows how high correlation exists between Deep Learning and Python. Indeed the libraries in Python like Keras, Tensorflow, Pytorch have made it possible for users to create robust deep learning applications. The popularity of these libraries have made Python the de facto language when it comes to Deep learning.

R on the other hand, is a language preferred by statisticians, which has great capabilities when dealing with statistics or visualizations. Deep Learning with R, is something unexpected and it was in the recent years that tensorflow was introduced for R, thereby extending the capabilities of R into the Deep Learning domain.

This tutorial is aimed for beginners to get acquainted with tensorflow and how to build models with it in R. Computer Vision and Natural Language Processing are the two major domains of Deep Learning. We will explore each of them by building a text classification model and an image classification.

Let's see how to bulid a text classification model first

## Text Classification - Sentiment Analysis with R

Sentiment Analysis is the task of finding the associated sentiment of a text. In a nutshell it's a text classification task whose output is a sentiment. 

Here we are using the IMDB dataset, which you can download through this [link](https://www.kaggle.com/lakshmi25npathi/imdb-dataset-of-50k-movie-reviews){target="_blank"}. It contains of 50k movie reviews, with equal number of positive and negative reviews.

### Step 1: Load the required Libraries


```r
library(dplyr)
library(ggplot2)
library(keras)
library(tensorflow)
```

Follow the steps in this [link](https://tensorflow.rstudio.com/installation/){target="_blank"} to install tensorflow in your system.

### Step 2: Read and explore data


```r
df <- read.csv("resources/deep_learning_with_r/imdb_dataset_one_fifth.csv")
str(df)
```

Check the distribution of the sentiment column


```r
df %>% count(sentiment)
```

There as many positive reviews as there are negative ones, i.e., they are equally distributed.

### Step 3: Split the data into training (80%) and testing (20%) datasets.


```r
ids <- sample.int(nrow(df), size = nrow(df)*0.8)
train <- df[ids,]
test <- df[-ids,]
```

Let's find out how many words are there in each reviews and get summary statistics for it. This will be later useful in setting parameters for the text vectorization function


```r
df$review %>% 
  strsplit(" ") %>% 
  sapply(length) %>% 
  summary()
```

Given a max of 2470 words in a sentence and the 3rd quantile being 280 words, we should limit the number of words to 300. The sentences have less than 300 words will be padded with zeros and those with greater than 300 will be truncated to contain 300 words.

We will arbitrarily fix the vocabulary to be 10000 words.

### Step 4: Data Preparation

The model works on tensors, hence we need to encode the data into tensors first. How do we do that? By using text vectorization layer we will assign integers to 10,000 most common words. These integers are used in place of the words to represent the input sequence.

Furthermore we need to encode each word separately. We will use embedding layer to do this. The embedding layer maps the integer to a fixed size array which is the encoded version of the integer.

Let's define the text vectorization layer.


```r
num_words <- 10000
max_length <- 300

text_vectorization <- layer_text_vectorization(
  max_tokens = num_words, 
  output_sequence_length = max_length, 
)
```
 
We need to adapt the text vectorization layer to the input data, to built a fix sized vocabulary and assign integer value to the words in them.


```r
text_vectorization %>% 
  adapt(df$review)
```

To check the vocabulary, uncomment the following code and run it.


```r
# get_vocabulary(text_vectorization)
```

Following is the example of how the layer transforms the input.


```r
text_vectorization(matrix(df$review[1], ncol = 1))
```

### Step 5: Build, train a model

Using keras_model function create the following model.


```r
input <- layer_input(shape = c(1), dtype = "string")

output <- input %>% 
  text_vectorization() %>% 
  layer_embedding(input_dim = num_words + 1, output_dim = 16) %>%
  layer_global_average_pooling_1d() %>%
  layer_dense(units = 16, activation = "relu") %>%
  layer_dropout(0.5) %>% 
  layer_dense(units = 1, activation = "sigmoid")

model <- keras_model(input, output)
```

Define the loss functiona and optimizer, and compile the model.


```r
model %>% compile(
  optimizer = 'adam',
  loss = 'binary_crossentropy',
  metrics = list('accuracy')
)
```

Train the model.


```r
history <- model %>% fit(
  train$review,
  as.numeric(train$sentiment == "positive"),
  epochs = 10,
  batch_size = 512,
  validation_split = 0.2,
  verbose=2
)
```

### Step 6: Evaluate the model:

Let's see how the model performs on unseen data?


```r
results <- model %>% evaluate(test$review, as.numeric(test$sentiment == "pos"), verbose = 0)
results
```

Plot the training vs validation accuracy and loss.


```r
plot(history)
```

## Image  Classification using R for CIFAR10 

Contextual image classification is a pattern recognition problem used in the field of computer vision used to classify the object or context of an image using machine AI and more importantly deep learining. 

Deep learning is a subset of machine learning, which is essentially a neural network with three or more layers. These neural networks attempt to simulate the behavior of the human brain—albeit far from matching its ability—allowing it to “learn” from large amounts of data. - [link](https://www.ibm.com/cloud/learn/deep-learning){target="_blank"}

In this walkthrough of Deep learning in R , We will be working with the CIFAR10 Dataset. The CIFAR10 Dataset consists of 60000 rbg images in 10 classes. 

The 10 Classes that are present in the CIFAR10 dataset are 
1.airplane
2.automobile
3.bird
4.cat
5.deer
6.dog
7.frog
8.horse
9.ship
10.truck

The CIFAR10 images are very tiny in nature , they are 32 x 32 pixels wide and have 3 Channels for each of the colors 

More Information about the CIFAR10 can be found at [link](https://www.cs.toronto.edu/~kriz/cifar.html){target="_blank"}


### Step 1: Load the required Libraries


```r
library(dplyr)
library(ggplot2)
library(keras)
library(tensorflow)
library(raster)
```

### Step 1a : To Make sure that your tensorflow is installed correctly run : 



```r
tf$constant("We are building CNNs in R !!")
```

### Step 2: Explore the Data

The CIFAR10 data is split into 2 components, the training data and the test data. The training data has 50,000 Images and the test data has 10,000 Images. 

The CIFAR10 dataset is packaged with Tensorflow and Keras and can be downloaded directly using tensorflow APIs.

?dataset_cifar10 has further information about the dataset.


```r
cifar10_dataset <- dataset_cifar10()
```


### Step 3: Split the data into training (80%) and testing (20%) datasets.


```r
c(train_images, train_labels) %<-% cifar10_dataset$train
c(test_images, test_labels) %<-% cifar10_dataset$test

# Image Data Preprocessing 

train_images = train_images/255
test_images = test_images/255
```

In standard Deep Learning , there are two main aspects of the model creation , one is the supervised learning part wherein the model is fed both the input and the output over which the model is optimized and the latter is the prediction and evaluation of images that have never been seen by the model before. 



### Step 3a : To see the dimensions of the train_images and the test_images dataset , use the dims function.


```r
dim(train_images)

dim(train_labels)
```

### Step 4 : As the dataset Labels are just integers , It is crucial we define the label array.

```r
class_names = c('airplane','automobile','bird','cat','deer','dog','frog','horse','ship','truck')										
```


### Step 5 : Let us plot these tiny images !! 

In order to understand how the images look like and what we are feeding into the model , let us build this function to plot the RBG image from the array of training images. 


```r
# Taking a random image from the training data , the leading comma's are important as we need to the entire 3D matrix.

random_image = train_images[10,,,]

dim(random_image)
```

We can see above that plot_image is just a 3D array of size (32X32x3)


```r
#Defining Function PlotImage

plotImage <- function(image){
  
  plot(raster::as.raster(image))
  
}

plotImage(random_image)
```

Since these are small images (32 pixels only) , the images are very blurry when plotting.

### Step 6 : Build a model

The Keras Sequential Model Object is instantiated to hold all the layers that we add to the CNN. Convolution Layers are special layers that can directly be fed images and there is no need to image data manipulation aside from some image processing for better results.

The construction of a convolutional neural network is a multi-layered feed-forward neural network, made by assembling many unseen layers on top of each other in a particular order.


```r
model <- keras_model_sequential()
```

Let us know the build the Model with the Convolutional Layers and Other Deep Learning layers for image classification.


```r
model %>%
  layer_conv_2d(filters=32,kernel_size = 3, padding="same",input_shape = c(32, 32,3), activation = 'relu') %>% 
  layer_batch_normalization() %>% 
  layer_conv_2d(filters=32,kernel_size = 3, padding="same",activation = 'relu') %>% 
  layer_batch_normalization() %>% 
  layer_max_pooling_2d(pool_size = c(2,2)) %>% 
  layer_dropout(0.2) %>% 
  layer_conv_2d(filters=16,kernel_size = 3, padding="same",activation = 'relu') %>% 
  layer_batch_normalization() %>%
  layer_conv_2d(filters=16,kernel_size = 3, padding="same",activation = 'relu') %>% 
  layer_batch_normalization() %>% 
  layer_max_pooling_2d(pool_size = c(2,2)) %>% 
  layer_dropout(0.3) %>% 
  layer_flatten() %>%
  layer_dense(units = 128, activation = 'relu') %>%
  
  layer_dense(units = 10, activation = 'softmax')
```

Information of the layers and how to add them to the model can be found here .  [link](https://tensorflow.rstudio.com/reference/keras/){target="_blank"}



We can see the summary of the Layers that build up the model by using the command : 


```r
summary(model)
```

### Step 7 : Compile the Model

In order to start training the model , we need to define the loss functions , the accuracy measurement techniques and the optimizers.

Optimizers Information can be found here [link](https://tensorflow.rstudio.com/reference/keras/optimizer_adam/){target="_blank"}

Loss functions Information can be found here [link](https://tensorflow.rstudio.com/reference/keras/loss_mean_squared_error/){target="_blank"}


```r
model %>% compile(
  optimizer = 'adam', 
  loss = 'sparse_categorical_crossentropy',
  metrics = c('accuracy')
)
```


### Step 8 : Time to train the model . 

The Fit function trains the model with the given training and testing images , the Number of epochs define how long the model is trained and for how many images.


```r
model_run  <- model %>% fit(train_images, train_labels, epochs = 10, verbose = 2)
```


### Step 8a : Let us plot the model Loss and Accuracy Functions. 


```r
plot(model_run)
```


### Step 9: Evaluate the model:

Let's see how the model performs on unseen data?


```r
score <- model %>% evaluate(test_images, test_labels, verbose = 0)

score
```

### Step 10:  Let us predict some images 


```r
img <- test_images[3, , , , drop = FALSE]
plotImage(img[1,,,])
class_pred <- model %>% predict(img)
class_id <- which.max(class_pred)
class_names[class_id]



img <- test_images[15, , , , drop = FALSE]
plotImage(img[1,,,])
class_pred <- model %>% predict(img)
class_id <- which.max(class_pred)
class_names[class_id]




img <- test_images[100, , , , drop = FALSE]
plotImage(img[1,,,])
class_pred <- model %>% predict(img)
class_id <- which.max(class_pred)
class_names[class_id]




img <- test_images[500, , , , drop = FALSE]
plotImage(img[1,,,])
class_pred <- model %>% predict(img)
class_id <- which.max(class_pred)
class_names[class_id]




img <- test_images[1200, , , , drop = FALSE]
plotImage(img[1,,,])
class_pred <- model %>% predict(img)
class_id <- which.max(class_pred)
class_names[class_id]
```

### Step 11: These models can be stored in the standard Keras HD5 Format and are transferrable to any other environment like Python or even Java !! 

```r
model %>% save_model_hdf5("CIFAR10-CNN-Model")
```

Loading saved models is easy as well !! 

```r
new_model <- load_model_hdf5("CIFAR10-CNN-Model")
```


## References:

1. [Tensorflow for R from R studio](https://tensorflow.rstudio.com/tutorials/){target="_blank"}
2. [CIFAR10 Data](https://www.cs.toronto.edu/~kriz/cifar.html){target="_blank"}
3. [Raster Image](https://www.rdocumentation.org/packages/graphics/versions/3.6.2/topics/rasterImage){target="_blank"}
4. [Keras Models](https://tensorflow.rstudio.com/reference/keras/){target="_blank"}
5. [Keras Optimizers](https://tensorflow.rstudio.com/reference/keras/loss_mean_squared_error/){target="_blank"}
6. [Image Plotting](https://stackoverflow.com/questions/62134844/how-to-plot-an-rgb-image-using-the-image-function-from-a-3d-array){target="_blank"}


