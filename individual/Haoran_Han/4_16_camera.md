## Subject: Webots Camera

### Date:  Apr 16   Author: <u>Haoran Han</u>

#### Purpose: 

To figure out the usage of webot camera specificaly.

#### 

#### 1. Basic Operation of Camera.

- Firstly, add the **"camera"** to the children part of the robot.

- The modifying process of the position and the direction is the same shown in above section.

- The direction that the camera towards is the **negtive direction along the z-axis** (opposite to the blue arrow).

- the upper part of the camera is along the **positive dirction along the y-axis** (towards the green arrow). The position of the camera that I select:

  ![Basic1](4_16_camera/Basic1.png)

- The typical Python code that calls the camera:

```
from controller import Robot,Camera
...
camera=Camera("test")
camera.enable(1)
...
image = camera.getImageArray()
```

- We need to import the `Camera` package. and  `camera=Camera("camera")`  calls the camera with a name property of "test". 
- Only after being `.enable(x)`, could the camera start to work. where `x` means the camera will use `x`ms to take a picture.
- By using `.getImageArray()`, we could get a **"list"** the represent the last picture that the camera get. The channel is **RGB**.
- By applying the **"Height"** and **"Weight"** property. we could get picture with different pixels 



#### 2. Usage of the Opencv package in the Webot:

- Firstly, simply import the package

```
import numpy as np
import cv2
```

- Applying `.getImageArray()`, we could get a list that represent the picture.

- By applying the next several lines of code, the picture could be process by the build in function of opencv

```
image = np.array(camera.getImageArray(),dtype="uint8")
r,g,b=cv2.split(image)
image=cv2.merge((b,g,r))
```

- The default type of the picture get by the Webot camera is `int32`, which could not be processed by the opencv. In that case, we need to set `dtype="uint8"` so that the opencv could process without error.
- The channel need to be changed.

if we dose not change the RGB channel:

![No_RGB](4_16_camera/No_RGB.png)

After change the RGB channel:

![RGB](4_16_camera/RGB.png)



### Date:  May 13   Author: <u>Haoran Han</u>

#### Change the direction of the image 

code:

```python
        (h, w) = image.shape[:2]
        center = (w // 2, h // 2)
        M = cv2.getRotationMatrix2D(center, -90, 1.0)
        image = cv2.warpAffine(image, M, (w, h))
```



If we do not rotate the image, the image will be 90 degree away from what we expect. 



After we add code to our previous code, The image will rotate to the original position.

![Rotate](4_16_camera/Rotate.png)



