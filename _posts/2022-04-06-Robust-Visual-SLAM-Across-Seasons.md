# Reproduced paper: Robust Visual SLAM Across Seasons
## 1. Introduction
This blogpost describes the implementation of reproducing the Deep Learning Paper: “Robust Visual SLAM Across Seasons” [[1]](#1) .The implementation has been described in two steps: Robust image matching and Sequence matching. In the first step, a descriptor has been extracted per image for two sequences of images by making use of the well-known Deep Convolutional Neural Network (DCNN): AlexNet. Subsequently, the similarity matrix has been computed by comparing each image of the query sequence with the database.In the second step, based on this similarity matrix, a minimum cost flow network problem has been formulated. With this formulation matching hypotheses between sequences have been computed. This resulting hypothesis is the loop closure information which could be used to formulate a graph based SLAM problem and compute the joint maximum likelihood trajectory. However this paper only reproduces until the loop closure information and does not implement the maximum likelihood trajectory. \
We will first describe the purpose of the reproduced paper. After that we describe the implementation of the Robust image matching and the Sequence matching; this includes the difficulties found during reproduction such as missing hyperparameter values. Finally, we show our results and draw conclusions from them.

## 2. Motivation
Place recognition is a core element in Simultaneous Localization and Mapping (SLAM). Drastic errors in trajectory estimation appear when there are incorrect loop closures. Detecting loop closures across seasons is a big challenge since often systems only perform well on minor perceptual changes in the environment. Therefore, the paper: “Robust Visual SLAM Across Seasons” [[1]](#1) had its main focus on computing consistent trajectories over longer periods of time and aimed at achieving robust place recognition across season.

## 3. Implementation
### 3.1 Robust image matching

The goal for the image matching is to match two sequences of images. Whereas the first sequence of the dataset is refered to as 'database' which is a ordered set of images: D = ($$d_1$$, ..., $d_D$) and the latter is refered to as the query dataset which is the set: \mathcal{Q} = ($q_1$, ..., $q_Q$). The query set is recorded in a different season and is thereby different from the database image, for example there is snow on the roads or leafs have fallen of the tree. 
For classification of images 'Deep Convolutional Neural Networks' (DCNN) have set the benchmark since AlexNet \cite{c3}. DCNNs can learns features from millions of training images. DCNNs consists of convolution layers in the early stages where it can present abstract feature presentations like edges or lines, see fig \ref{fig:feature representation}. For this comparing task the AlexNet model is used which is pre-trained on the ImageNet dataset. The authors of paper \cite{c2} have reported that the Conv3 layer of AlexNet behaves more robust to seasonal changes on their datasets so that is the layer that we also chose to use.

\begin{figure}[H]
    \centering
    \includegraphics[width=0.5\textwidth]{images/conv3.png}
    \caption{The abstract feature representations}
    \label{fig:feature representation}
\end{figure}

For the image input we resize the images to 231x231 pixels to fit the input for the AlexnNet. Here we diverge from the original paper where they resize to 256x256 pixels and we follow \cite{c2}. This is because after going through the convolution layers we want an image descriptor with a dimension of 384x13x13, ultimately resized in a feature vector of size 64896.
The comparison between the quary and database are done with the cosine similarity. The cosine similarity calculates how much the two images look like each other, with image descriptors respectively $I_D_i$ and $I_D_j$. The full cosine similarity matrix is calculated with equation \ref{eq:3}. In this matrix image descriptors are compared to each other. 
\begin{equation}
\label{eq:3}
S_i,j = I_Q_i \cdot I_D_j
\end{equation}

With $S_{i,j}$ between [0,1] where $S_{i,j}$ = 1 is full similarity. 

```math
e^{i\pi} + 1 = 0
```

$a^2 + b^2 = c^2$.


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
