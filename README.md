# Project: Vehicle Detection and Tracking

---
## Summary of Project 
In this project, a  detection algorithm is in Python with computer vision and supervised machine learning to detect and tracking vehicles in videos shot by front facing camera in car. The steps of this algorithm are described below:

* Perform a Histogram of Oriented Gradients (HOG) feature and color features extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Implement a sliding-window technique and use the trained classifier to search for vehicles in images.
* Implement a pipeline on a video stream and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

The algorithm is implemented and tested on a testing image set and different highway driving videos, and successfully demonstrates a detection accuracy of with 99.2 % in the testing image set, and a good vehicle detection and tracking capability in the testing videos. 

---

## Step 0: Load data
In order to train classifier that can be used to detect the vehicle, a data set contains pictures of car and non-car objects are loaded. Below is a demonstration of 10 randomly selected samples from each category:
![alt text](https://github.com/davidsky900/SelfDrivingCar-VehicleDetection/blob/master/figures/dataDemo.png)

The top row is 10 pictures of cars whereas the bottom row is 10 pictures of non-car objects. 
* Total number of car pictures: 8792
* Total number of non-car pictures: 8968

---

## Step 1: Extract features and train classifier
### a. Extract features (HOG and Color features)
The pictures loaded from Step 0 are mixed, randomly shuffled and splited into training and In order to extract features can be used for training the classfier, two types of feature are used. The first type of feature is the Histogram of Gradient (HOG feature) of the picture. The HOG feature is extracted by the function of  `skimage.hog()`, and the parameters are tuned in a trial and error process. There are two objectives need to be traded-off during the tuning

* to obtain high classification accuracy
* to lower computational burden

These two objective are competing with each other as high accuracy requires more features to be converted and will increase the computational burden. After tuning the final parameters are chosen as below:
* `orientations` = 16
* `pixels_per_cell` = 8
* `cells_per_block` = 2 x 2

The second type of feature is the histogram of color. The pictures are converted into `YCrCb` color space and the histogram of color are concatenated. The parameters are:
* number of bins = 32
* range of bins = (0 255)

The HOG feature and color features are concatenated into 1 dimensional vector, and normalized within the entire data set and splitted into training and validation set. The final format of the proccessed feature vectors are listed below:
* dimension of 1 feature vector: 1 x 9504
* number of training feature vectors: 13320
* number of validation feature vectors: 4440

The following pictures demonstrate the originial image (left), the extracted HOG feature (middle) and the concatenated and normalized feature vector. 
![alt text](https://github.com/davidsky900/SelfDrivingCar-VehicleDetection/blob/master/figures/featureCar2842.png)
![alt text](https://github.com/davidsky900/SelfDrivingCar-VehicleDetection/blob/master/figures/featureNonCar3108.png)

### b. Train classifier
I trained a linear SVM using `sklearn.svm`. The validation accuracy achieved 99.2 %. 

---

## Step 2: Sliding window search
### a. Description of method
In order to detect the vehicle within the image, a sliding window search approach is used. Since the car only appears in a certain portion of the image, it is more efficient to divide the image into several regions and perform slide window search within the region. The region chosen are shown as the green boxes in the picture below:
![alt text](https://github.com/davidsky900/SelfDrivingCar-VehicleDetection/blob/master/figures/slideWin0.png)

### b. Examples and optimization
Since the classifier was trained to classifier square image patch with 64 x 64 pixels, the windows need to be scaled to fit the dimension. In this case, the parameters used are listed below:
* scales used = 3, 1.25, 0.8
* cell per step = 2

Here are some examples:
![alt text](https://github.com/davidsky900/SelfDrivingCar-VehicleDetection/blob/master/figures/slideWin1.png)
![alt text](https://github.com/davidsky900/SelfDrivingCar-VehicleDetection/blob/master/figures/slideWin5.png)

---

## Video Implementation
Here's a [link to my video result](https://github.com/davidsky900/SelfDrivingCar-VehicleDetection/blob/master/output_videos/video2_out.mp4)

### a. Build pipeline for single image
In order to merger all the positive detections into correc number of detections in each frame of video. A pipeline of detection for single image is built. In this pipeline, all the positive detections are merged as heat map, which essientially is a vote of all the pixels with positive detection. The heat map is then converted into labels which indicates the location and occupation of the car within the image. The convertion from heat map into lables are done by `scipy.ndimage.measurements.label()`. The labels is shown with a bounding boxes. Below are some examples of the pipeline for test images:
![alt text](https://github.com/davidsky900/SelfDrivingCar-VehicleDetection/blob/master/figures/pipeline0.png)
![alt text](https://github.com/davidsky900/SelfDrivingCar-VehicleDetection/blob/master/figures/pipeline1.png)
![alt text](https://github.com/davidsky900/SelfDrivingCar-VehicleDetection/blob/master/figures/pipeline3.png)

The left is the images with positive detections, the middle is the heat map created fro detections and the right is the final detection of cars generated from heat map. 

### b. Eliminate false detection and smooth heat map
In order to smooth the detection and reduce the false detection, the heat maps are stored up to 12 frame and merged to smooth the detection. A threshold hold is also applied to filter out false detections. The relavent parameters are listed below:
* Number of buffered heat maps = 12
* Minimun detections to be counted into heat map = 2

---

## Discussion
The presented algorithm works decent with detection of vehicle, however there are few challenges needs to be addressed in the future works. 
a. The bounding box indicating the detection of vehicle still has some jitter, this is caused by the fact the edge of the heat map will change frame by frame. This could be addressed by increasing more levels of region and different levels of scales of search windown, which will potentially increase the density of the heat map and gives higher level of confidence of the exact dimenson of the occupation of the car in the scene. This inevitablly increase the computational burden and will needed to be rigrously explored with more powerful computational capability. 
b. When muiltiple vehicle overlaps in a single frame, the classifier might be lump them together under current algorithm. To address this issue. The detectioin result can return certain information that is unique with detected car, for instance, color of the car, so that when creating heat map and labels these positive result can be seperated from each other. 

## Disclaimer
The project is part of study of the Nano-degree of Udacity "Self-Driving Car Engineer". The learning material and the data is provided by Udacity, and the key implementation of the project is carried out by Yi Chen.

