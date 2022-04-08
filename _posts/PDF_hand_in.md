Authors: \
Max Waterhout - 5384907 - m.waterhout@student.tudelft.nl\
Ynze ter Horst - 4701682 - Y.Y.G.F.terHorst@student.tudelft.nl\
Emma Allemekinders - 5608198 - E.A.Allemekinders@student.tudelft.nl\
Andreas Zwanenburg - 5413494 - A.zwanenburg-1@student.tudelft.nl

# Reproduced paper: Robust Visual SLAM Across Seasons
## 1. Introduction
This blogpost describes the implementation of reproducing the Deep Learning Paper: “Robust Visual SLAM Across Seasons” [[1]](#1) .The implementation has been described in two steps: Robust image matching and Sequence matching. In the first step, a descriptor has been extracted per image for two sequences of images by making use of the well-known Deep Convolutional Neural Network (DCNN): AlexNet. Subsequently, the similarity matrix has been computed by comparing each image of the query sequence with the database.In the second step, based on this similarity matrix, a minimum cost flow network problem has been formulated. With this formulation matching hypotheses between sequences have been computed. This resulting hypothesis is the loop closure information which could be used to formulate a graph based SLAM problem and compute the joint maximum likelihood trajectory. However this paper only reproduces until the loop closure information and does not implement the maximum likelihood trajectory. \
We will first describe the purpose of the reproduced paper. After that we describe the implementation of the Robust image matching and the Sequence matching; this includes the difficulties found during reproduction such as missing hyperparameter values. Finally, we show our results and draw conclusions from them.

## 2. Motivation
Place recognition is a core element in Simultaneous Localization and Mapping (SLAM). Drastic errors in trajectory estimation appear when there are incorrect loop closures. Detecting loop closures across seasons is a big challenge since often systems only perform well on minor perceptual changes in the environment. Therefore, the paper: “Robust Visual SLAM Across Seasons” [[1]](#1) had its main focus on computing consistent trajectories over longer periods of time and aimed at achieving robust place recognition across season./ 
SLAM is also needed because just using the best score for every query image with respect to the database in the similarity matrix can lead to false matchings since the best similarity might not be the true positive, see fig. [1] for an example. With SLAM we also take the temporal dimension into account. 


<p align="center">
    <img src="https://user-images.githubusercontent.com/95222839/162386555-332f7ad3-bb67-497c-8c3f-72da22a764e3.png" width="400" height="300">
    <br>
        <em>Fig. 1: The highest similarities scores as dots</em>
</p>



## 3. Implementation
### 3.1 Robust image matching

The goal for the image matching is to match two sequences of images. Whereas the first sequence of the dataset is refered to as 'database' which is a ordered set of images: <img src="https://render.githubusercontent.com/render/math?math=D = (d_1,... ,d_2)"> and the latter is refered to as the query dataset which is the set: <img src="https://render.githubusercontent.com/render/math?math=Q = (q_1,... ,q_2)">. The query set is recorded in a different season and is thereby different from the database image, for example there is snow on the roads or leafs have fallen of the tree. 
For classification of images 'Deep Convolutional Neural Networks' (DCNN) have set the benchmark since AlexNet [[3]](#3). DCNNs can learns features from millions of training images. DCNNs consists of convolution layers in the early stages where it can present abstract feature presentations like edges or lines, see fig [2].

<p align="center">
    <img src="https://user-images.githubusercontent.com/95222839/162166314-35dc55b2-e684-4980-8295-ad4903ac4a95.png" width="400" height="200">
    <br>
        <em>Fig. 2: The abstract feature representations</em>
</p>

For this comparing task the AlexNet model is used which is pre-trained on the ImageNet dataset. The authors of paper [[2]](#2) have reported that the Conv3 layer of AlexNet behaves more robust to seasonal changes on their datasets so that is the layer that we also chose to use.

For the image input we resize the images to 231x231 pixels to fit the input for the AlexnNet. Here we diverge from the original paper where they resize to 256x256 pixels and we follow [[2]](#2). This is because after going through the convolution layers we want an image descriptor with a dimension of 384x13x13, ultimately resized in a feature vector of size 64896.
The comparison between the quary and database are done with the cosine similarity. The cosine similarity calculates how much the two images look like each other, with image descriptors respectively <img src="https://render.githubusercontent.com/render/math?math=I_{Q_i}"> and <img src="https://render.githubusercontent.com/render/math?math=I_{D_j}">. The full cosine similarity matrix is calculated with equation [1]. In this matrix image descriptors are compared to each other. 
<p align="center">
    <img src="https://render.githubusercontent.com/render/math?math=S_{i,j} = (I_{Q_i} \cdot I_{D_j}) \quad (1)  "> 
</p>

With <img src="https://render.githubusercontent.com/render/math?math=S_{i,j}"> between [0,1] where <img src="https://render.githubusercontent.com/render/math?math=S_{i,j} = 1 "> is full similarity. 

### 3.2 Sequence matching
For sequence matching, we implemented a minimum cost flow network, which has a structure representing the problem of matching images while taking into account the temporal nature of these images. Specifically, a single flow needs to be found from a source node to a sink node through a network of nodes that indicate either a match between a certain database-query image pair or a non-match between them. The nodes are connected with edges of a certain weight, dependent on a range of hyperparameters and the similarity matrix presented in the previous section. The higher the similarity between a database-query pair, the lower the cost of reaching the node representing a match. The accumulative cost of the weights of edges in a flow should be minimized, creating a sequence matching.

Setting the hyperparameters for this problem is quite cumbersome and the reproduced paper does not specify any details as to its settings. Therefore we had to do some heuristic search to find suitable ones. Unfortunately these settings also seemed to be very dependent on the problem and a small tweak could make the difference between convergence and divergence. \
A final remark with respect to the shortcomings of our implementation is the fact that the paper references a previous implementation for the minimum cost flow network but extends this implementation with an additional component which is not described in detail. Therefore, we needed to make some design choices with respect to set of edges aimed at reducing ambiguity in database images, referenced by epsilon h in the paper.\\

To show the matching, a simple GUI was employed to show the matches in topological order. Such a match consists of the database image, query image and identifiers of both.

## 4. Results
In figure 3 below, a few examples of matches in a certain sequence can be seen.

<p align="center">
    <img src="https://user-images.githubusercontent.com/95222839/162180656-91e67a1e-6202-47e3-809c-dfaacfb68795.png" width="300" height="300">
    &nbsp; &nbsp; &nbsp; &nbsp;
    <img src="https://user-images.githubusercontent.com/95222839/162181051-a0116b09-743f-4734-85f1-a1bc94bcc31a.png" width="300" height="300">
    <br>
        <em>Fig. 3: Examples of matched sequences</em>
</p>

There is no easy way of visually showing the entire matching. But to give an impression of the quality of a matching or the behaviour of the minimum cost flow network, the matching can be mapped onto the similarity matrix. In figure 4 and 5 some of these can be seen.

<p align="center">
    <img src="https://user-images.githubusercontent.com/95222839/162182140-d786785d-5276-44da-a2d4-7e1639b6261b.png" width="600" height="300">
    <br>
        <em>Fig. 4: Example of  the minimum cost flow </em>
</p>

In figure 3, the black opaque dots are the maximal similar-
ity across the query dimension. It is clear from the hues of the
background (representing the similarity) and these black dots
that the query sequence is a concatenation of two sequences
corresponding to the single database sequence. In this case
they are recorded in winter and autumn, while the database
sequence is recorded in summer. The red dots are the matches
found by the network, it matches the first sequence rather than
the second, which is to be expected judging from the maximal
similarities.

<p align="center">
    <img src="https://user-images.githubusercontent.com/95222839/162182059-d48b0cc6-c2aa-4d49-a056-b2184748b666.png" width="300" height="300">
    <br>
        <em>Fig. 5: Another example of the minimum cost flow </em>
</p>

Figure 4 shows the same for a different dataset, without maximal similarities and transposed. In this particular case, the first part of the first sequence is matched but the matching skips the remaining part and matches the rest of the second query sequence instead.

## 5. Conclusion and Discussion
In this project, the reproduction of "Robust image matching" and "Sequence matching" from the paper “Robust Visual SLAM Across Seasons” [[1]](#1) has been done.
We were able to match similar locations based on dashcam images as was done in the original paper. By applying specific layers of the pre-trained AlexNet on the images, we generated feature scores that we compared with cosine similarity between the dataset and query set. We visualised the similarity matrix from the cosine similarity as a heatmap. The correct matchings were not clearly visible on the heat map, but by combining minimal cost flow with sequence information we were able to reproduce a robust matching algorithm. 

A big challenge was to choose the right hyperparameters for sequence matching since this was not clearly defined in the paper. We made ourselves design choices which will not be exactly the same as in the reproduced paper. We have used the same dataset as mentioned in the original paper, but these were quite hard to implement as of query size, combining/retrieving ground truths and so on.  To further improve this work, the dataset with ground truths can be implemented and results can be compared. To conclude, our final implementation shows promising results and implements two main topics of the paper despite the lack of important information in the paper.

## References
<a id="1">[1]</a> 
T. Naseer, M. Ruhnke, C. Stachniss, L. Spinello and W. Burgard, "Robust
visual SLAM across seasons," 2015 IEEE/RSJ International Conference
on Intelligent Robots and Systems (IROS), 2015, pp. 2529-2535, doi:
10.1109/IROS.2015.7353721. \
<a id="2">[2]</a> 
N. S ̈underhauf, F. Dayoub, S. Shirazi, B. Upcroft, and M. Milford, On
the performance of convnet features for place recognition, arXiv preprint
arXiv:1501.04158, 2015. \
<a id="3">[3]</a> 
A. Krizhevsky, I. Sutskever, and G. E. Hinton, Imagenet classication with
deep convolutional neural networks, in Advances in Neural Information
Processing Systems 25, 2012, pp. 1097 1105.

Contributions:\
Max : Preprocessing data with robust image matching\
Ynze : Sequence matching\
Andreas : Results \
Emma : Preprocessing data with Robust image matching


