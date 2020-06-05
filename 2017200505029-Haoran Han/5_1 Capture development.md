## Subject: Capture Function Development

### Date:  May 1   Author: <u>Haoran Han</u>

#### Purpose: 

To define the Capture function that could save image from the camera, this function will be used by Bo Wen. 

#### Experiment：

```Python
def capture(self, key):
        '''
        capture images from cameras to test/camera/[key]/
        '''
        capture_path = '../../../test/camera/' + str(key) + '/'
        if not os.path.exists(capture_path):
            os.makedirs(capture_path)
        image = self.get_image(key)
        cv2.imwrite(capture_path + time.strftime("%Y-%m-%d-%H-%M-%S", 	time.localtime()) + '.png', image)
        info('Captured! 📸')
```