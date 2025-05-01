### The data.yaml file is modified (compared to the ones in cybele lab)
Modified file

------------------- 1:

train: images/train
val: images/valid
test: images/test

nc: 1
names: ['pole']

### Code walktrough

The notebook inference_with_analysis.ipynb contains functions that performs inference and analyzes model performace and behaviur. Resulting plots and metrics are saved in the folder "results". The notebook is defined both in yolov12_our_files and in yolov12_our_files. The two versions have same functionality, but are tailored to different yolo models 

The file modified_loss_function.txt is a modifed version of the yolov12 loss function (yolov12/ultralytics/utils/loss.py). Two terms are added to the classification loss function, which penalize all generated bounding boxes every iteration. 

- Penalizes boxes with aspect ratio over 0.25 (The highest ar in the dataset). Penalization increases linear with the deviation from 0.25
- Penalizes boxes with size that deviates from a regression line describing the y-position of the lower side of boxes vs their vertical length (height). The regression line basically looks at the y-position of a box, and then computes the optimal length of a box in that position. But we do not penalize length deviations directly, this leads to very wide boxes. Instead, we define a maximim aspect ratio of 0.25 and a minimum aspect ratio of 0.02, use these to calculate maximim and minimum acceptable areas of boxes based on their y-position, and penalize boxes that deviate from this limits. The penalization is linear.  

benchamrking.ipynb measures the speed of inference. It does not count computations or measure the speed analytically, it simply measures the time it takes to perform inference. Thus it is dependent on the state of the computer (GPU memory usage), which must be identical across all tests

### Yolov5 Autoanchors on RGB data returns these anchors:

3,26 (width=3, height=26) → aspect ratio ≈ 1:8.7
4,30 (width=4, height=30) → aspect ratio ≈ 1:7.5
3,42 (width=3, height=42) → aspect ratio ≈ 1:14
5,40 (width=5, height=40) → aspect ratio ≈ 1:8
5,67 (width=5, height=67) → aspect ratio ≈ 1:13.4
8,78 (width=8, height=78) → aspect ratio ≈ 1:9.8
21,73 (width=21, height=73) → aspect ratio ≈ 1:3.5
16,140 (width=16, height=140) → aspect ratio ≈ 1:8.8
41,129 (width=41, height=129) → aspect ratio ≈ 1:3.1
WARNING: Extremely small objects found: 15 of 392 labels are <3 pixels in size



