## Subject: **Setting of Camera**

### Date:  March 17   Author: Haoran Han

- ### introduction

This experiment mainly introduce how to set the **camera** connector of the RaspberryPi. Moreover, how to install `opencv-python` in to the raspberry is also intruceded.

- ### preparation

Hardware: laptop,  RaspberryPi, Camera

## Experiment Procedure

In 2020/3/17, because of the Inappropriate  usage of the connector, the port of the camera is broken. In that case, my RaspberryPi cannot find any camera setting even though i connect my camera to it. 





### Date:  March 2   Author: Haoran Han

- ### preparation

Hardware: laptop,  RaspberryPi, a Camera with USB connection, a Camera with CSI connection

## Experiment Procedure

In 2020/3/17, my new camera comes to me. I bought two camera with different connection. One is USB, one is CSI.

1. After I connect the camera with a CSI connection to the RaspberryPi, it still cannot recognized this divice. Open the command line, type in `sudo raspi-config`, go to the Interfacing Options, enable Camera、Serial、Remote GPIO. If you do not enable Camera operation, RaspberryPi cannot recognize your device.
2. Following Song's work, I download the `.whl` file [opencv-contrib-python-4.1.0.25](https://www.piwheels.org/simple/opencv-contrib-python/opencv_contrib_python-4.1.0.25-cp37-cp37m-linux_armv7l.whl) into my laptop first. Then this file is transmitted to the Desktop of my Raspberry.
3. Typing `pip install /home/pi/Desktop/opencv_contrib_python-4.1.0.25-cp37-cp37m-linux_armv7l.whl` in the command line and the the downloading will finish very soon.
4. from now on, we could import module `cv2` in our `.py` file.

However, my Raspberry could only recognize the camera which is connected via **USB port**. Even though the RaspberryPi could use the camera connected via CSI port, the cv2 module could not recognize it.

- Solution: typing `sudo nano /etc/modules` first, add a line of `bcm2835-v4l2`  (lower case of L) at the end of the file. Save the file. Reboot the file.

then, we could find two of the camera could be used.





here is my final test code:

```python
import cv2

cap0 = cv2.VideoCapture(0)
ret0, frame0 = cap0.read()
frame0 = cv2.flip(frame0, 0)

cap1 = cv2.VideoCapture(1)
ret1, frame1 = cap1.read()
frame1 = cv2.flip(frame1, 0)

while (cap0.isOpened() and cap1.isOpened()):    
    if cv2.waitKey(1) & 0xFF == ord('q'):        
        break    
    ret0, frame0 = cap0.read()    
    frame0 = cv2.flip(frame0, 1)    
    cv2.imshow('frame0', frame0)    
    ret1, frame1 = cap1.read()    
    frame1 = cv2.flip(frame1, 1)    
    cv2.imshow('frame1',frame1)

cap0.release()
cap1.release()
cv2.destoryAllWindows()
```


