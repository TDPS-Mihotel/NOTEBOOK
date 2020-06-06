## Subject: Object Detection

### Date:  May 27   Author: <u>Haoran Han</u>

#### Purpose: 

Wen Bo has already provide the **Bridge Detection** in `.mat` file, I need to translate it into the `.py` language and function.  

#### Experiment：

```Python
  def bridge_detection(self, image):
        '''
        This algorithm gives whether we detect the bridge\n
        if bridge is detected, return `True`, else return `false`
        Written by Wen Bo, modified by Han Haoran
        '''
        # Binarization
        # cv2.imshow('bridge', image)
        binary_map = np.zeros(shape=image.shape)
        binary_map[image < 149] = 1
        binary_map[image > 153] = 1
        # delete the noise
        kernel = np.ones([3, 3], np.uint8)
        dilate = cv2.dilate(binary_map, kernel, iterations=1)

        counter = np.sum(dilate == 0)
        # if the x index of the bridge is in no farther than x_range*width from the 			center,
        # then the bridge is in the center
        if counter > 240:
            location = np.argwhere(binary_map == 0)
            f_x = np.mean(a=location, axis=0)[1]
            x_range = 0.02
            if np.abs(f_x - image.shape[1] // 2) <= x_range * binary_map.shape[1]:
                detectedInfo('time: ' + self.signals['Time'] + ' | Bridge detected.')
                return True
        return False
```

The for loop in Wen Bo's original code that could eliminate the noise point is replaced by  the `dilate`  build-in function.







### Date:  May 28   Author: <u>Haoran Han</u>

#### Purpose: 

Wen Bo has already provide the **Gate Detection** in `.mat` file, I need to translate it into the `.py` language and function.  

#### Experiment：

```Python
	def gate_detection(self, image_gray):
        '''
        This algorithm gives whether we detect the gate\n
        if gate is detected, return `True`, else return `False`
        Written by Wen Bo, modified by Han Haoran
        '''
        # Get the  gradient map
        gradient = cv2.Sobel(image_gray, cv2.CV_64F, 1, 0, ksize=3)
        # Binarization
        binary_map = np.zeros(shape=image_gray.shape)
        binary_map[gradient > 400] = 1
        # Counting the pixel along a column where the gradient exceed the threshold
        counter = np.sum(binary_map, axis=0)
        edge = np.argwhere(counter > 10)
        if edge.shape[0] >= 4:
            f_x = np.mean(edge)
            x_range = 0.65
            if np.abs(f_x - image_gray.shape[1] // 2) <= x_range * binary_map.shape[1]:
                detectedInfo('time: ' + self.signals['Time'] + ' | Gate detected.')
                return True

        return False
```


