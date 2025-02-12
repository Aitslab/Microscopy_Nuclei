# Evaluation

For evaluating the models we have used validation and test sets from  [Nuclei](https://www.sciencedirect.com/science/article/pii/S2352340922009726).

We have compared two metrics as pixel-based Jaccard index and object-based F1-score where the threshold for comparing two objects are 90%.

The metrics that we have used for both architectures are Jaccard Index and F1-score where they both are defind based on Intersection over Union (IOU),
The IoU measures the overlap of predicted and annotated objects and is calculated according to the following equation:

```{math}
IOU(A,B) = \dfrac {A \cap B }{A\cup B}
```

IoU determines Jaccard Index as a pixel-based index that can range from 0 to 1, from no overlap to identical objects. 
Since overlap of predicted and original objects is rarely 100\% we selected a few different thresholds for comparison (50\% to 90\%). This is for calculating F1-score which is an object-based index. F1-score measures how many out of all predictions were true positives as the geometric average between the recall and precision.


## U-Net evaluation

For evaluating U-Net models we have used our three best models as Model3, Model12, and Model14 where in the main manuscript we have mentioned 
the different training sets that they were trained on. 
The whole evaluation process of U-Net models are described in evaluation/UNet/UNET_concise_Evaluation.ipynb

The required functions are mostly taken from  [carpenterlab](https://github.com/carpenterlab/unet4nuclei.git) repo and revised according to our models and data.
Besides, a classification of Tiny, Small, Normal and Large nuclei is also performed to see in which category the models work best.
Then Jaccard index were calculated based on IOU and then by assigning a threshold, F1-score were calculated.

The following table shows the results for each U-Net model on test set.




|   |F1_score_90   | Average Jaccard Index    |  False Discovery Rate |  Precision  | Recall |
|---|--------------|--------------------------|----------------------|-------------|--------|
|3  | 0.8099       | 0.87                     | 0.066                |    0.934    |  0.729 |
|12 | 0.8061       | 0.88                     | 0.038                |    0.962    |  0.702 |
|14 | 0.8461       | 0.88                     | 0.063                |    0.937    |  0.766 |


## HoVer-Net evaluation

For evaluting the performnace of HoVer-Net models on validation and test set (same as UNet from [Nuclei](https://www.sciencedirect.com/science/article/pii/S2352340922009726))
we have used loadmat function from scipy.io python library and then we compared the "inst_map" feature of ".mat" files as predicted masks to be able to compare to gold annotated segmentation masks.

Same as UNet evaluation process, small objects (minsize = 25) were removed during comparison. 
The whole evaluation process of HoVer-Net models are described in evaluation/HoVerNet/HoverNet_concise_evaluation.ipynb.

The following table shows the results for the best three top models of  HoVer-Net architecture valiadated on test set.


|   |F1_score_90   | Average Jaccard Index    |  False Discovery Rate |  Precision  | Recall |
|---|--------------|--------------------------|----------------------|-------------|--------|
|17 | 0.7551       | 0.83                     | 0.110                |    0.890    |  0.688 |
|23 | 0.7487       | 0.82                     | 0.128                |    0.872    |  0.690 |
|37 | 0.7725       | 0.83                     | 0.109                |    0.891    |  0.711 |
