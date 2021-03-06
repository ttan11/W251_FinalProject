## 1. Introduction

Distance learning or e-learning has been trending in these years because of its advantage in providing education opportunities at reasonable costs to those who cannot be on campus all the time. Distance learning can be uptaken in multiple ways, including computer-based learning (e.g. software assisted learning) and internet-based learning (e.g. pre-recorded online lecture). Among all the options available, real-time interactive virtual classroom is closest to traditional face-to-face teaching and learning and is also most technologically intense. 

By the end of 2019, virtual classrooms have been mostly used to deliver higher education courses, but since the breakout of COVID-19 in early 2020, virtual classrooms started to take over the K-12 education as well when parents, children and teachers hastily tried to figure out a safe learning environment. According to a AACSB Quick-Take Survey, globally 51 percent of respondents had converted face-to-face courses to online or virtual formats, with some regional differences. 

There are many commercial softwares to facilitate virtual classrooms, with Zoom, the video chatting application as the leading player. Compared to December 2019, the daily users in Zoom meetings increased by 20 folds in March 2020 and has kept increasing as of today.

<img src="images/intro.png" width="800">

Despite all the benefits virtual classrooms can bring in, there is still concern that the productivity may not be comparable to that of the face-to-face classrooms. Generally, people are not quite sure about the effectiveness of course delivery and the engagement of students. Here we would like to introduce Virtual TA, a prototype of deep learning based facial expression recognition application, which monitors students’ instant facial expressions and helps the teacher to address confusion or misunderstanding immediately and to maintain the students’ engagements and motivations. 

In order to get an idea of what the potential users of Virtual TA want to get out from this application, we did a mini market survey. We sent out a survey with 6 questions to the instructors who teach virtual classes. According to the responses, the major pain point for instructors in teaching a virtual class is how to figure out whether the students understood the concept and how to get them to engage and participate. About 80% of instructors would like to use Virtual TA if it’s available and also 80% think a summary of facial expressions will help them on the teaching work. Specifically, 60% think the most helpful expression is the confused and they would like to see statistics on level of interest and engagement, overall mood or sentiments during a lecture and across sections. 

The model used by Virtual TA for facial recognition and expression classification is deep neural network, which is trained on cloud and deployed on edge such as the laptop or the Nvidia Jetson TX2. Virtual TA has two aspects of functions: 1. It can capture faces from a virtual classroom using a web camera, and then classify the expression for real time; 2. It can real time process online conference images, classify the expressions and output statistics of interest, for example the overall confused level during some period of a lecture. The success of Virtual TA largely depends on the model performance. We assume that with more usage, more data can be obtained and used to further train the deep neural network, making it more accurate and more generalized. 

## 2. Data and EDA
### 2.1 Kaggle competition dataset
Initial dataset was downloaded from Kaggle competition “Facial expression recognition with deep learning”. The dataset consists of 36000 images for 7 classes, which are Angry, Disgust, Fear, Happy, Sad, Surprise, and Neutral. The images are 48x48 pixels and grey in color. However, the problem of using this dataset is first, it doesn’t have any asain face and second, it doesn’t have the class of most interest. We tried to generate the missing part of data and combine it with the Kaggle dataset but didn’t see significant improvements on model performance. We hypothesized that higher resolution may be needed so we decided not to use this dataset for model training and validation.

<img src="images/kaggle_dataset.png" width="300">

### 2.2 Cohn-Kanade (CK) AU-Coded Facial Expression dataset
The Cohn-Kanade (CK) AU-coded Facial Expression dataset includes 2000 images sequences from over 200 subjects (university students). This dataset was analysed by Cohn, Zlochower, LIen, & Kanade (1999) and by Lien, Kanade, Cohn, & Li (2000). These papers can be downloaded from http://www.cs.cmu.edu/~face. These images are 480x480 pixels with grey scale. The image sequences from neutral to target motion and the target emotion is the last frame (as shown below). The final frame of each image sequence was coded using FACS (Facial Action Coding System) which describes subjects’s expression in terms of action units (AUs). An Excel spreadsheet containing these FACS codes is available for our analysis. We did a round of investigation of these images and did not use this set of images in our final model to the complexity of the FACS coding system as well as the image sequence. The image sequence makes it harder for us to tell which image we need to use in our training model.

<img src="images/CK_dataset.png" width="400">

### 2.3 Custom dataset
For our final model, we eventually chose to use our own custom dataset. This dataset contains more than 4000 images from 6 subjects and the sample images are shown below. The images are 224x224 pixels with color scale (RGB). This dataset contains 3 classes which are confused, happy, and surprised. The image was captured by Jetson TX2, face is cropped to 224x224 using the open-cv face detection using Haar Cascades. After faces are cropped out and categorized into different classes, images are uploaded to VM in ibm cloud for training/testing the model. 

<img src="images/us_dataset.png" width="500">

## 3. Model
### 3.1 Model 1 - Kaggle Competition model
Kaggle hosted the competition for facial emotion recognition in 2016. In this competition, 7 classes of facial emotions (Angry, Disgust, Fear, Happy, Sad, Surprise, and Neutral) are classified using CNN model. For our project, we are interested in identifying students’ confusion and distraction emotion. Hence we add two more classes (confused and distraction) to the input with custom labelled data.  The model architecture is shown below with 4 convolution layers and 2 fully connected dense layers. Batch normalization and dropout are used to future improve model performance. The model loss function is categorical cross-entropy and the model optimizer is Adam. Model loss and accuracy were plotted for at different epochs. The train accuracy is 0.75 while test accuracy is 0.62. Based on the confusion matrix (normalized) results, one can see that the model has poor performance. The model generalized the facial emotions more to “happy”, “neutral” and “sad”. We also conduct real-time tests with jetson TX2 webcam and the model is not able to correctly classify facial emotions. One possible reason for low model performance is the resolution of the image and the color of the image (grey). Lower resolution and grey color mean that there are less features to extract and generalize the model prediction. Therefore we tried to improve the model performance by using customized RGB images with high resolution (224x224).

<img src="images/kagglemodel.png" width="800">

### 3.2 Model 2 - littleVGG
The second model was littleVGG and the architecture is shown below (Fig 3-2). It has 6 convolutional layers and one dense layer. The dataset for this model is a Kaggle dataset with 48x48 pixel images in greyscale. For this model, we incorporated data augmentations, such as rescale 1/255, rotation rage = 30, shear range = 30, zoom range =30, as well horizontal flip. The loss function is categorical cross entropy, and the optimizer is Adam with a learning rate of 0.0001 and decay of 1e-6. 

Experiments have been performed with this model trained for 9 classes, 7 classes and 5 classes, and we obtained roughly similar results. All experiments (9,7,5 classes) contained the customized classes (CONFUSED and DISTRACTED). For simplicity, we only show the model results for 7 classes. The normalized confusion matrix results of 7 classes are shown below (Fig 3-2).  LittleVGG model can detect most of the classes except ANGRY and SAD. 
The model performance of littleVGG (model 2) is better than that of Kaggle competition model (model 2). We tested littleVGG model on Jetson TX2 for real-time emotion classification and the result is not optimum, hence we decided to further improve the model. 

<img src="images/littleVGG.png" width="800">

### 3.3 Model 3 - Transfer Learning with Resnet50
Transfer learning is a research problem in machine learning that focuses on storing knowledge gained while solving one problem and applying it to a different but related problem. In the domain of deep learning, transfer learning usually involves using a pre-trained model, which has been previously trained on a dataset and contains the weights and biases that represent the features of the dataset, to “learn” something else and by doing so, the training time will be significantly reduced without compromising the model performance. 

Here we used a pre-trained model Resnet50 to conduct transfer learning for facial expression classification. Resnet50 was built on top of VGG19 and has more layers and better performance. For Resnet50, researchers reformulate the layers as learning residual functions with reference to the layer inputs, instead of learning unreferenced functions. With an ensemble of these residual nets, Resnet50 achieved an error rate of as low as 3.57% on the ImageNet test set, and championed in the ILSVRC 2015 classification task. 
 
The architecture of the transfer learning model consists of Resnet50 and one additional dense layer. The model used sgd as an optimizer with learning rate of 0.1, decay = 1e-6, and momentum = 0.9. The loss function is categorical cross entropy. The model has 10 epochs and the train/test accuracy reached almost 100% after 3 epochs, indicating very fast convergence and high level of performance. The model weights were saved in .h5 format and loaded on the edge device (Jetson TX2) for inference. 

<img src="images/resnet50.png" width="800">

## 4. Pipeline
The pipeline of virtual classroom TA prototype is shown below. The local computer was used to collect data, preprocess data, and manage the virtual machine. The preprocessing steps included resizing the image, data augmentation, changing image color to RGB or grey, and so forth. The local computer was also used to conduct EDA. 

The customized dataset was collected using Jetson TX2. Customized python scripts were run to collect images from an external webcam camera with 30 FPS speed. After that, the individual face is cropped by using the openCV face detection Haar Cascades model. We collected a total of 4000 images in 3 classes (happy, confused, surprised), split the images into train/test with 7:3 ratio, and then uploaded the images to the IBM cloud virtual instance. 

One P100 virtual server is provisioned on IBM cloud with Ubuntu 18.4 operating system and tensorflow, CUDA 10.0. We containerized the code via Docker image and the framework for our model is Keras. We leverage transfer learning from ResNet50 with pre-trained weights and add one last layer to make the prediction for 3 classes. We also conducted hyperparameter tuning to optimize the model performance. In this part (on VM in the cloud), it is an iterative process. We tried with various datasets (section 2) to build different models (section 3). Eventually customized dataset (224x224 pixel color image) and transfer learning model were selected as our final model. 

Prototype outputs are visualized using Jetson TX2. For real-time face emotion detection, we load the final model weights in Jetson TX2, and output the emotion classes on the screen with classification text. The second product is emotion timeline, we basically run a python script on Jetson TX2 to crop out the face and make prediction of emotion using our final model. Plot the emission summary curve graph as our final output. For more details please see section 5 for end products. 

<img src="images/pipeline.png" width="600">

## 5. End Product
### 5.1 Real-time face emotion recognition and notification application

Real time analysis of the facial expressions is the first critical feature of our prototype. Using camera input, it is expected to capture faces, classify the expressions and display the last emotion detected (figure 5-1). This feature allows a virtual class instructor to tell instantly whether the student understands or gets confused on the concept. Accordingly, the instructor can address misunderstanding or confusion immediately, maintaining the virtual class productivity. 

<img src="images/Fig5-1.png" width="600">

### 5.2 Emotion Timeline

The commercial virtual class software (e.g. zoom) usually grants the instructors access to the video of the recorded lecture. Therefore, we designed the other feature of the prototype, which is to process a recorded video, analyze the facial expression and output the statistics of interest. This feature allows instructors to retrospect the virtual class for a comprehensive analysis of the moments when students get confused or surprised. 
In the demo, we created a pipeline of real-time facial recognition during an online conference session. After the conference ended, all faces captured were analyzed for expression classification. Overall confused levels by time were plotted as shown below.
<img src="images/Fig5-2.png" width="600">

## 6. Model Performance

### 6.1 Real Time emotion recognition and notification application
Testing methodology:
Test model on Jetson Tx2 using external USB camera
Test against 4 different users
Take 20 pictures per emotions per users
Run prediction after each picture

Model performance on real-time facial expression classification is shown in the table below. The average accuracy is 0.91, 0.61, 0.85 for classes of happy, surprised, and confused, respectively. Interestingly, we didn’t expect to see the lowest model accuracy on the surprised class because both happy and surprised classes have more pronounced face expressions, while confused class involves more subtle and less uniformed facial expressions, such as frown, crooked mouth, blink one eye, etc. Some testers could emphasize a frown forehead with a crooked mouth, but others were not naturally able to do so. These more pronounced expressions could yield better accuracy. To improve our accuracy, our model could be more generalized if we have better training and validation images. 

<img src="images/Fig6-1.png" width="600">

### 6.2 Emotion Timeline
Testing methodology:
Zoom Conference
Test against 3 different users
Perform 20 secs happy, 20 secs confused, and 20 secs surprised
Real time picture capturing 1 picture per sec and record timestamp for each picture.
Run prediction after each picture straight after conference concluded
Input: 1 min pictures of 3 different users.
Output: Group and individual predicted emotion charts

Shown below are the plots of individual expression classification with x-axis as the video timestamp and y-axis as the predicted probability on a certain class of facial expression. Each dot represents one second prediction and the curved line represents the moving average probability of 3 seconds. Based on the plots, our model can classify happy most accurately, followed by confused, and then surprised. The results are consistent with that of real-time analysis. One can see clearly the time borderline when students change their facial emotion every 20 seconds. This clearly demonstrates that our prototype can work appropriately to help teachers identify and trace student emotion changes and their learning behaviors. 

<img src="images/Fig6-2.png" width="800">

## 7. Future Steps
Virtual classrooms are undoubtedly the new normal. To address the uncertainty of the effectiveness of online classes in comparison to the face-to-face classrooms, we have developed Robo-Sensei the Virtual TA, a prototype of deep learning based facial expression recognition application. With Robo-Sensei, virtual class instructors can identify confusion or misunderstanding among students instantly and can address that in time to maintain the students’ engagements and motivations. 

Robo-Sensei has two key features: 1.  It can capture faces from a virtual classroom using a web camera, and then classify the expression for real time; 2. It can real-time process an online conference, classify the expressions and output statistics of interest, for example the overall confused level during the period of a lecture. The current version of Robo-Sensei is able to detect 3 main emotions -- happy, surprise and confused -- with high accuracy.

Throughout the development work, we had faced several challenges. One key challenge is the lack of data in good quality. The public dataset we had located does not have any asian features and image resolution varies. We also found that our model is very sensitive to lighting. Therefore, most of our tests have to be done during daylight.  There is no available API that we can use to tap into the zoom conference to real time extract of the images during the conference. Due to the lack of resources, we had innovatively created a python script to capture the conference discussion images.

We see great opportunities and advances in Robo-Sensei. One future step is to replace  current openCV Haar Cascades face detection model with Multi-task cascaded convolutional neural network(MTCNN). Secondly, it will bring great benefit to provide additional data dimension, for example associating images with voice to provide information on which phrases or topics are resulting in certain emotions. Finally, it would be a great addition to enhance the model with more classes, such as distraction or concentration. This will further help teachers to understand student emotions and learning behaviors.

## Reference
Real-time: <br>
1. https://medium.com/datadriveninvestor/real-time-facial-expression-recognition-f860dacfeb6a <br> 

Facial Expression Databases: <br>
2. https://en.wikipedia.org/wiki/Facial_expression_databases#cite_note-8 <br>
3. https://insights.insofe.com/index.php/2019/05/13/facial-emotion-recognition-for-classroom-video-analytics/#:~:text=Facial%20Emotion%20Recognition%20is%20the,academic%20and%20online%20teaching%20institutes.<br>
4. https://www.kaggle.com/jonathanoheix/face-expression-recognition-with-deep-learning<br>
5. http://app.visgraf.impa.br/database/faces/<br>
6. https://www.kaggle.com/apollo2506/facial-recognition-dataset?select=Testing<br>
7. https://towardsdatascience.com/real-time-face-recognition-an-end-to-end-project-b738bb0f7348<br>
8. https://github.com/martin-chobanyan/emotion<br>

