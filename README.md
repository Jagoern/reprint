# PASCAL VOC 
## Object Classification
### 50.039 Theory and Practice of Deep Learning

The goal of this project is to recognize objects from a number of visual object classes in realistic scenes. There are 20 object classes:
1. Person
2. Bird, cat, cow, dog, horse, sheep
3. Aeroplane, bicycle, boat, bus, car, motorbike, train
4. Bottle, chair, dining table, potted plant, sofa, tv/ monitor

Specifically for the classification task, the goal is, for each of the classes predict the presence/ absence of at least one object of that class in a test image.

## Data
We will use Pascal VOC 2012 dataset for this project and the latest version of pytorch has Pascal VOC dataset class built-in. For the purpose of this project, we will only use training set and validation set of Pascal VOC. The ground truth annotation for the dataset contains the following information,
* Class: the object class. I.e. car or person
* Bounding box: an axis-aligned rectangle specifying the extent of the object visible in the image
* View: ‘frontal’ , ‘rear’, ‘left’ or right
* Difficult: an object marked as difficult indicates that the object is considered difficult to recognize without substantial use of context.
* Truncated: an object marked as ‘truncated’ indicates that the bounding box specified for the object does not correspond to the full extent of the object.
* Occluded: an object marked as ‘occluded’ indicates that a significant portion of the  subject image is within the bounding box occluded by another object.

For our task, we regarded all ‘difficult’ marked objects as negative examples.

## Loss function

The task is multi-label classification for 20 object classes, which is analogous to creating 20 object detectors, 1 for every class. Hence we have used binary cross entropy (with logits loss) as our loss function. The loss can be described as:

    .. math::
        \ell(x, y) = L = \{l_1,\dots,l_N\}^\top, \quad
        l_n = - w_n \left[ y_n \cdot \log \sigma(x_n)
        + (1 - y_n) \cdot \log (1 - \sigma(x_n)) \right],

    where :math:`N` is the batch size. If reduce is ``True``, then

    .. math::
        \ell(x, y) = \begin{cases}
            \operatorname{mean}(L), & \text{if size\_average} = \text{True},\\
            \operatorname{sum}(L),  & \text{if size\_average} = \text{False}.
        \end{cases}



In pytorch, the loss function is torch.nn.BCEWithLogitsLoss( ). Do note that this function provides numerical stability over the sequence of sigmoid followed by binary cross entropy. 

## Metrics
We used average precision as the metric to measure performance. It is simply the average of the maximum precisions at different recall values. 

Accuracy is a very poor metric to be used in this problem. I.e.: In a picture containing a person and dog, the model can output the result as train and get away with an accuracy of 85%!

# Model
We used ResNet18 as our deep learning architecture due to relatively low memory utilization since number of parameters are less. (We used our local machines to train, preserving AWS credits for final project).  

We used transfer learning method since the object classes are very similar to ImageNet classes.

## Hyper-parameters
Image size: 300x300
Batch size: 32
Initial learning rate: [1e-5 for resnet backbone, 5e-3 for the fully connected layer]
Optimizer: SGD 
Learning rate Scheduler: CosineAnnealing learning rate scheduler with Tmax = 12 
Momentum: 0.9
Epochs: 15

## Challenges
1. Hyperparameter search was an interesting challenge we faced. Initially our model faced overfitting problems and we were able to fix the problem by,
2. Using smaller learning rate for the pre-trained resnet backbone so that we do not disrupt imagenet weights drastically.
3. Using larger learning rate for the randomly initialized fully connected layer.
4. Using learning rate scheduler rather than using static learning rates.
5. Choosing good set of image augmentations to  add small amount of noise during training to make the model robust.

