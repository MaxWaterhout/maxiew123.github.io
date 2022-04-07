## 1. Introduction
This blogpost describes the implementation of reproducing the Deep Learning Paper: “Robust Visual SLAM Across Seasons” [[1]](#1) .The implementation has been described in two steps: Robust image matching and Sequence matching. In the first step, a descriptor has been extracted per image for two sequences of images by making use of the well-known Deep Convolutional Neural Network (DCNN): AlexNet. Subsequently, the similarity matrix has been computed by comparing each image of the query sequence with the database.In the second step, based on this similarity matrix, a minimum cost flow network problem has been formulated. With this formulation matching hypotheses between sequences have been computed. This resulting hypothesis is the loop closure information which could be used to formulate a graph based SLAM problem and compute the joint maximum likelihood trajectory. However this paper only reproduces until the loop closure information and does not implement the maximum likelihood trajectory. \\
We will first describe the purpose of the reproduced paper. After that we describe the implementation of the Robust image matching and the Sequence matching; this includes the difficulties found during reproduction such as missing hyperparameter values. Finally, we show our results and draw conclusions from them.

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
