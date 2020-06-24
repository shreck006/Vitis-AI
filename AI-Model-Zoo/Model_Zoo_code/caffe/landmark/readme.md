### Contents
1. [Installation](#installation)
2. [demo](#demo)
3. [Preparation](#preparation)
4. [Train/Eval](#traineval)
5. [Performance](#performance)
6. [Model_info](#model_info)

### Installation
1. Get Xilinx caffe-xilinx code.
  ```shell
  unzip caffe-xilinx.zip
  cd caffe-xilinx
  ```

2. Build the code. Please follow [Caffe instruction](http://caffe.berkeleyvision.org/installation.html) to install all necessary packages and build it.
  ```shell
  # Modify Makefile.config according to your Caffe installation.
  cp Makefile.config.example Makefile.config
  make -j8
  make pycaffe
  ```

3. opencv-python
  ```shell
  #You may need to make sure opencv-python is installed
  pip install opencv-python
  ```
 
 4. protobuf
 ```
 Because the Python layer is used in the code,  protobuf needs to be installed and consistent with the version used when compiling caffe-xilinx.
 ```


#### demo
1. run demo
  ```shell
  cd code/test/
  sh demo.sh
  # Ouput wil be found in test/output.
  ```

### Preparation

1. dataset describle.
   Landmark training and test datasets are private.

2. prepare dataset.
   The model provided is obtained by multi-task learning. It requires multiple labels.
   If you want to retrain model, you can prepare your data according to the following data format.
   ```
   train dataset:
   step1: Crop out faces detected by your detector and resize them to (H:96, W:72)
   step2: prepare label and generate list file.  
          landmark label: Take picture imagename.jpg as an example, the format of txt ground truth list is as follows:
          imagename x1 x2 x3 x4 x5 y1 y2 y3 y4 y5
          (five points, left-eye, right-eye, nose, left-mouth-corner, right-mouth-corner)
          
          age and sex label: These two labels are included in the image name, the format is as follows:
          imageName_age_sex.jpg, e.g. 22462_40_1.jpg
          sex:0-female, 1-male
        
   test dataset: The method is consistent with the train dataset
   ```
3. Dataset Directory Structure like:
   ```
    + data
        + landmark_train_image
            + 1.jpg
            + 2.jpg
        + landmark_test_image
            + 1.jpg
            + 2.jpg
        + age_sex_train_image 
            + aa_36_0.jpg
            + bb_28_1.jpg
        + age_sex_test_image
            + cc_36_0.jpg
            + dd_28_1.jpg
        + landmark_train_list.txt
        + landmark_test_list.txt
        + age_sex_train_list.txt
        + age_sex_test_list.txt
   ```

### Train/Eval

1. Train your model.
  ```shell
  cd code/train
  sh train.sh 
  ```

2. Evaluate caffemodel.
  ```shell
  cd code/test/
  # evaluate  
  sh test.sh
  #You can also run the following command to test.
  #python landmark_evaluate.py --testImgList ../../data/landmark_test_list.txt  --inputImgPath ../../data/landmark_test_image
  ```

### Performance

```
points-l1-loss:0.1162
weighted points-l1_loss: 19.52
```

### Model_info

data preprocess
```
1. data channel order: BGR(0~255)                  
2. resize: 96 * 72(H * W)
3. mean_value: 127.5, 127.5, 127.5
4. scale: 1 / 127.5
```

Quantize the network with calibration mode

1. Replace the "Input" layer of the test.prototxt file with the "ImageData" data layer.
2. Modify the "ImageData" layer parameters according to the date preprocess information
3. Provide a "quant.txt" file, including image path and label information with fake value(like 1).
4. Give examples of data layer and "quant.txt":

```shell
layer {
  name: "data"
  type: "ImageData"
  top: "data"
  top: "label"
  include {
    phase: TRAIN
  }
  image_data_param {
    source: "quant.txt"
    batch_size: 32
    new_width: 72
    new_height: 96
  }
  transform_param {
    mirror: false
    mean_value: 127.5
    mean_value: 127.5
    mean_value: 127.5
    scale: 0.00784315
  }
}
```
```
# quant.txt: image path label
images/000001.jpg 1
images/000002.jpg 2
images/000003.jpg 3
...
```