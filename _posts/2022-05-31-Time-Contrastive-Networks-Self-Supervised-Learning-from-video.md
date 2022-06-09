# Reproduced paper: *Time-Contrastive Networks: Self-Supervised Learning from Video*
**Authors:** Max Waterhout (5384907) & Amos Yususf (4361504) & Tingyu Zhang (5478413) 

In this blog post, we present the results of our attempted replication and study of the 2018 paper by Pierre Sermanet et al. 
*Time-Contrastive Networks: Self-Supervised Learning from Video* [[1]](#1). This work is part of the CS4245 Seminar Computer Vision
by Deep Learning course 2021/2022 at TU Delft. This whole reproduction is done from scratch and can be found in our github: https://github.com/maxiew123/TCN_self_supervised_learning/tree/main

***

## 1. Introduction
In the computer vision domain, deep neural networks have been successful on a big range of tasks where labels can easily be specified by humans, like object detection and segmentation. A bigger challenge lies in applications that are difficult to label, like in the robotics domain. An example would be labeling a pouring task. How can a robot understand what important properties are while neglecting setting changes? Ideally, a robot in the real world can learn a pouring task purely from observation and understanding how to imitate this behavior directly. In this reproduction, we train a network on a pouring task that tries to learn the important features like the pose and the amount of liquid in the cup while being viewpoint and setting invariant. This pouring task is learned through the use of supervised learning and representation learning. In the following, we will provide a motivation for this paper, our implementation of the model, the results that we achieved against the benchmarks and lastly we discuss the limitations of our implementation. 

<p align="center">
  <img src="https://user-images.githubusercontent.com/95222839/171206978-348aa639-7d85-4911-a20c-c2981dd7a2b9.gif" width="300" height="500"/> 
  <br>
    <em>fig 1. An example sequence of a pouring task</em>
</p>

## 2. Motivation
Imitation learning has already been used for learning robotic skills from demonstrations [[2]](#2) and can be split into two areas: behavioral cloning and inverse reinforcement learning. The main disadvantage of these methods is the need for a demonstration in the same context as the learner. This does not scale well with different contexts, like a changing viewpoint or an agent with a different model. In this paper, a Time-Contrastive Network (TCN) is trained on demonstrations that are diverse in embodiments, objects and backgrounds. This allows the TCN to learn the best pouring representation without labels. Eventually with this representation a robot can use this as a reward function. The robot can learn to link its images to the corresponding motor commands using reinforcement learning or another method. In our blog, we do not cover the reinforcement learning part.


## 3. Implementation
For our implementation of the TCN we only use the data of the single-view data. The input of the TCN is a sequence of preprocessed 360x640 frames. In total 11 sequences of around 5 seconds (40 frames) are used for training. The framework contains a deep network that outputs a 32-dimensional embedding vector, see fig [2]. 

<p align="center">
<img src="https://user-images.githubusercontent.com/95222839/171224461-0ac7e6c2-46cc-40f1-8156-8109a7df10ad.png" width="500" height="350" alt="single view TCN"> 
<br>
<em>Fig. 2: The single-view TCN</em>
</p>

### 3.1 Training
The loss is calculated with a triplet loss [[3]](#3). The formula and an illustration can be seen in fig [3]. This loss is calculated with an anchor, positive and negative frame. For every frame in a sequence, The TCN encourages the anchor and positive to be close in embedding space while distancing itself from the negative frame. This way the network learns what is common between the anchor and positive frame and different from the negative frame. In our case the negative margin range is 0.2 seconds (one frame) and negatives always come from the same sequence as the positive. 

<p align="center">
<img src="https://user-images.githubusercontent.com/95222839/171224677-7de1c4ed-2f58-4d4e-9db8-c2388a18a855.png" width="700" height="105" > 
<br>
</p>

<p align="center">
<img src="https://user-images.githubusercontent.com/95222839/171224869-613abcca-6381-4150-b8f6-371b7b32c89e.png" width="600" height="161" alt="Training loss">
<br>
<em>Fig. 3: The triplet loss</em>
</p>

The main purpose of the triplet loss is to learn representations without labels and simultaneously learn meaningful features like pose while being invariant to viewpoint,scale occlusion, background etc.. 

### 3.2 Deep network
The deep network is used for feature extraction. This framework is derived from an Inception architecture initialized with ImageNet pre-trained weights. The architecture is up until the "Mixed-5D" layer followed by two 2 convolutional layers, a spatial softmax layer and a fully connected layer. Note that the spatial softmax layer outputs the x and y coordinates of the maximum activation from each channel. Since our reference paper did not give the convolution kernel size, we followed paper [[5]](#5), and used a 5x5 convolution plus ReLu as an activation function.

<p align="center">
<img src="https://user-images.githubusercontent.com/95222839/172599588-93a32770-d087-4432-bd42-a5a436606729.png" width="600" height="161" alt="Spatial softmax ">
<br>
<em>Fig. 4: The Spatial Softmax</em>
</p>


## 4. Results
For the results we used accuracy measured by video allignment. The allignment captures how well a model can allign a video. The allignment metrics that are used are the L2 norm and the cosine simularity. The metric matches the nearest neighbors, in embedding space, with eachother. In this way, for each frame the most semantically similar frame is returned. We state that a true positive is when a frame lies in the positive range from eachother. This way frame sequence: [1,2] gives the same accuracy as [2,1]. 
We compare our results against the pre-trained Inception-ImageNet model [[4]](#4). We use the 2048D output vector of the last layer before the classifier as a baseline. The same baseline is used in our reference paper.

### 4.1 Final result overview
The model is trained on the Google Cloud with one P100 GPU. SGD, SGD with momentum, and Adam were used during different training iterations. Between 1 to 800 iterations, the optimizer was the SGD and between 800 to 4200 iterations, we switched the optimizer to SGD with momentum because the improvement on the loss was slow. After 4200 iterations, we used Adam as the optimizer for the same reason. During the training, single view dataset was used and there were total of 17 videos. Each video lasts 7 seconds and contains scenes of pouring taking from the front view. 11 videos were used as training dataset and the rest were for testing. Because there was no validation set to select the best training model, we only saved models for every 200 iterations and for models that had the new minimum losses. In the end, we trained the model for 13k iterations and the training loss is shown in Fig [5]. The zigzaging behaviour is due to the 200 iterations gap as well as the missing data betweening 2000 to 6000 iterations after one virtual machine crash.   

<p align="center">
<img src="https://user-images.githubusercontent.com/95222839/171225019-834200ab-a7d2-4c42-8f0a-dbb9675b70e3.png" width="400" height="300" alt="Training loss"> 
 &nbsp; &nbsp; &nbsp; &nbsp;
<img src="https://user-images.githubusercontent.com/95222839/171225144-ec37da4d-98ea-4377-b55c-fe100254479a.png" width="400" height="300" alt="Figure 1 paper"> 
<br>
<em>Fig. 5: The training loss and testing accuracy</em>
</p>

The alignment accuracy from each saved network model for the testing set is plotted in Fig [6]. Various criterion were used to measure the similarity between two embedded frames, such as cosine similarity and euclidean distance (l2). We paid more focus on the l2 distance with one frame tolerence because this setup is closely related to the training procedure.  
The best accuracy measured with this criteria is from the model at the 7200th iteration. The average alignment accuracy is 80.11 percent whereas the Baseline method has an average accuracy of 71.04 percent. 

<p align="center">
<img src="https://user-images.githubusercontent.com/95222839/171226259-2c59dcdf-7457-47df-8ee2-2c8dc3c02acc.gif" width="700" height="700"> 
<br>
<em>Fig. 7: Video results. From left to right column: Ground truth, Baseline, TCN after 4 iterations, TCN after 7200 iterations </em>
</p>


### 4.2 Result overview
<p align="center">
<img src="https://user-images.githubusercontent.com/95222839/172601021-fb6ea1e1-32c8-4a22-b76e-d8eaca0f5545.png" width="900" height="300" alt="Results"> 
<br>
<em>Fig. 8: Results</em>
</p>

In the table above, data with * are from the reference paper[[1]](#1) and the k-neareast neighbour scheme was used for accuracy measurement. Our alignment accuracy was measured from l2 distance since we did not train any classifier. The two accuracy measurements give the similar score for Baseline model, 70.2% and 71% percent. Although our training iteration was limited by the hardware, the increment on the accuracy matches with historical data which means that the model did learn the water pouring representation from the triplet loss. We contribute our higher accuracy results to the small sample size because the reference model was trained on multi-view data set where as we only trained the model for single view dataset. 

## 5. Discussion and Limitations
test
### 5.1 Discussion
1. Overal performance on our results
2. reEmphasis on what we did: evaluation scheme l2. 
3. future: add multiview dataset. Evaluate network on imitation tasks 
### 5.2 Limitations
1. preprocessing unknow. (normalization and reshape size). sol: Inception net for normalization. readme for reshape
2. computing power limits and cost (google cloud)
3. code from author was expired

## References
<a id="1">[1]</a> Sermanet, P., Corey, L., Chebotar Y., Hsu J., Jang E., Schaal S., Levine S., Google Brain (2018). Time-Contrastive Networks: Self-Supervised Learning from Video. <i>University of South California</i>. [https://arxiv.org/abs/1704.06888]() \
<a id="2">[2] </a> J.A. Ijspeert, J. Nakanishi, and S. Schaal. Movement imitation
with nonlinear dynamical systems in humanoid robots. In
ICRA, 2002. \
<a id="3">[3] </a> X. Wang and A. Gupta. Unsupervised learning of visual
representations using videos. CoRR, abs/1505.00687, 2015. \
<a id="4">[4] </a> J. Deng, W. Dong, R. Socher, L.-J. Li, K. Li, and L. Fei-Fei. ImageNet: A Large-Scale Hierarchical Image Database. In
CVPR, 2009. \
<a id="5">[5] </a> C. Finn, X. Y. Tan, Y. Duan, T. Darrell, S. Levine, and
P. Abbeel. Learning visual feature spaces for robotic
manipulation with deep spatial autoencoders. CoRR,
abs/1509.06113, 2015.











