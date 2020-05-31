### Subject: Apply filters to smooth the image

#### Date: <u>May 13</u>   Author: <u>Chang Shu</u>

##### Purpose: 

Smoothing is a key step in digital image processing. In practice, LPF is usually implemented to smooth the high frequency signal in the image, namely making the difference between two neighbor pixels smaller. This can make color recognition much easier. 

##### Procedure:

This experiment contains two parts:

1. implement four different LPF on the image
2. plot the blurred images and compare them

##### Experiment:

The code is shown below.

```python
 	image = cv2.imread(args['image'])
    blur = cv2.blur(image,(5,5))
    GaussianBlur = cv2.GaussianBlur(image, (5, 5), 0)
    median = cv2.medianBlur(image,5)
    bilateralFilter = cv2.bilateralFilter(image,9,75,75)

    plt.subplot(231),plt.imshow(image[...,::-1]),plt.title("original")
    plt.subplot(232),plt.imshow(blur[...,::-1]),plt.title("blurred")
    plt.subplot(233),plt.imshow(GaussianBlur[...,::-1]),plt.title("Gaussian blurred")
    plt.subplot(234),plt.imshow(median[...,::-1]),plt.title("median filtered")
    plt.subplot(235),plt.imshow(bilateralFilter[...,::-1]),plt.title("bilateral filtered")
    plt.show()
```

##### Results:

![1590908948290](C:\Users\树畅\AppData\Roaming\Typora\typora-user-images\1590908948290.png)

As we can see from above, four blurred images look smoother than original image. Among them, I choose the GaussianBlur as the filter in our following process.

### Subject: <u>Fix the value range of each color in HSV space</u>

#### Date: <u>May 18</u>   Author: <u>Chang Shu</u>

##### Purpose: 

In our project, we need to recognize several colors, including orange, yellow, red, purple. In digital image processing, common practice has it that difference colors are separated in HSV color space. In order to separate different colors by their value ranges, I conduct the following experiment.

##### Procedure:

This experiment contains two parts:

1. Plot the histogram of each color in HSV color space to obtain a rough range
2. fine-tune this range on images from the webots camera with GUI

##### Experiment:

The code is shown below.

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

# USAGE: You need to specify a filter
#
# (python) color_thresholding --filter RGB --image /path/to/image.png
# or
# (python) color_thresholding --filter HSV --image /path/to/image.png

import argparse

import cv2
import numpy as np
from matplotlib import pyplot as plt


def callback(value):
    pass


def setup_trackbars(range_filter):
    cv2.namedWindow("Trackbars", 0)

    for i in ["MIN", "MAX"]:
        v = 0 if i == "MIN" else 255

        for j in range_filter:
            cv2.createTrackbar("%s_%s" % (j, i), "Trackbars", v, 255, callback)


def get_arguments():
    ap = argparse.ArgumentParser()
    ap.add_argument('-f', '--filter', required=True,
                    help='Range filter. RGB or HSV')
    ap.add_argument('-i', '--image', required=False,
                    help='Path to the image')
    ap.add_argument('-p', '--preview', required=False,
                    help='Show a preview of the image after applying the mask',
                    action='store_true')
    args = vars(ap.parse_args())

    if not bool(args['image']):
        ap.error("Please specify only one image source")

    if not args['filter'].upper() in ['RGB', 'HSV']:
        ap.error("Please specify a correct filter.")

    return args

def get_trackbar_values(range_filter):
    values = []

    for i in ["MIN", "MAX"]:
        for j in range_filter:
            v = cv2.getTrackbarPos("%s_%s" % (j, i), "Trackbars")
            values.append(v)

    return values

def main():
    args = get_arguments()

    range_filter = args['filter'].upper()

    # Compare different filters
    image = cv2.imread(args['image'])
    # blur = cv2.blur(image,(5,5))
    GaussianBlur = cv2.GaussianBlur(image, (5, 5), 0)
    # median = cv2.medianBlur(image,5)
    # bilateralFilter = cv2.bilateralFilter(image,9,75,75)

    # plt.subplot(231),plt.imshow(image[...,::-1]),plt.title("original")
    # plt.subplot(232),plt.imshow(blur[...,::-1]),plt.title("blurred")
    # plt.subplot(233),plt.imshow(GaussianBlur[...,::-1]),plt.title("Gaussian blurred")
    # plt.subplot(234),plt.imshow(median[...,::-1]),plt.title("median filtered")
    # plt.subplot(235),plt.imshow(bilateralFilter[...,::-1]),plt.title("bilateral filtered")
    # plt.show()


    # Plot Histogram in HSV Color Space
    blur = cv2.cvtColor(GaussianBlur,cv2.COLOR_BGR2HSV)
    # Plot HSV Histgram
    fig, ax = plt.subplots()
    hsvColor = ('y','g','k')
    bin_win = 3
    bin_num = int(256/bin_win)
    xticks_win = 2

    ax.set_title('HSV Color Space')
    lines = []

    for cidx, color in enumerate(hsvColor):
        cHist = cv2.calcHist([blur], [cidx], None, [bin_num], [0,255])
        line = ax.plot(cHist, color=color, linewidth=8)
        lines.append(line)

    labels = [cname + ' Channel' for cname in 'HSV']

    plt.legend(lines, labels, loc='upper right')
    ax.set_xlim([0, bin_num])
    ax.set_xticks(np.arange(0, bin_num, xticks_win))
    ax.set_xticklabels(list(range(0, 256, bin_win*xticks_win)),rotation=45)
    plt.show()


    # Start thresholding
    if range_filter == 'RGB':
        frame_to_thresh = GaussianBlur.copy()
    else:
        frame_to_thresh = cv2.cvtColor(GaussianBlur, cv2.COLOR_BGR2HSV)

    setup_trackbars(range_filter)

    while True:

        v1_min, v2_min, v3_min, v1_max, v2_max, v3_max = get_trackbar_values(
            range_filter)

        thresh = cv2.inRange(
            frame_to_thresh, (v1_min, v2_min, v3_min), (v1_max, v2_max, v3_max))
        thresh = cv2.threshold(thresh, 127, 255, cv2.THRESH_BINARY_INV)[1]

        if args['preview']:
            preview = cv2.bitwise_and(image, image, mask=thresh)
            cv2.imshow("Preview", preview)
        else:
            cv2.imshow("Original", image)
            cv2.imshow("Thresh", thresh)

        if cv2.waitKey(1) & 0xFF is ord('q'):
            break

if __name__ == '__main__':
    main()
```

The histogram of different colors are obtained using Python and plotted as follows. Then I use the Python GUI to fine-tune the range on images from webots.

###### Orange:

![orange_hsv](E:\python\pycharm\TDPS\imgs\orange_hsv.png)

<center>pure orange in HSV Space<center>

![orange_thresh](E:\python\pycharm\TDPS\imgs\orange_thresh.png)

<center>The image with only orange kept<center>

As is shown above, orange is preserved in white while other colors are filtered with the thresholds in the trackbars.

###### Yellow:

![yellow_hsv](E:\python\pycharm\TDPS\imgs\yellow_hsv.png)

<center>pure yellow in HSV Space<center>

![yellow_thresh](E:\python\pycharm\TDPS\imgs\yellow_thresh.png)

<center>The image with only yellow kept<center>

As is shown above, yellow is preserved in white while other colors are filtered with the thresholds in the trackbars.

###### Red:

![red_hsv](E:\python\pycharm\TDPS\imgs\red_hsv.png)

<center>pure red in HSV Space<center>

![red_thresh](E:\python\pycharm\TDPS\imgs\red_thresh.png)

<center>The image with only red kept<center>

As is shown above, red is preserved in white while other colors are filtered with the thresholds in the trackbars.

###### Purple:

![purple_hsv](E:\python\pycharm\TDPS\imgs\purple_hsv.png)

<center>pure purple in HSV Space<center>

![purple_thresh](E:\python\pycharm\TDPS\imgs\purple_thresh.png)

<center>The image with only purple kept<center>

As is shown above, purple is preserved in white while other colors are filtered with the thresholds in the trackbars.

##### Results:

 Finally, we can obtain a color value range table as follows.

| color  | [$h_{min}$, $h_{max}$] | [$S_{min}$, $S_{max}$] | [$V_{min}$, $V_{max}$] |
| ------ | ---------------------- | ---------------------- | ---------------------- |
| orange | [14, 21]               | [199, 208]             | [234, 240]             |
| red    | [0, 3]                 | [246, 255]             | [144, 154]             |
| purple | [132, 138]             | [246, 255]             | [168, 178]             |
| yellow | [27, 33]               | [243,252]              | [146, 160]             |



### Subject: <u>Use green block as the beacon</u>

#### Date: <u>May 25</u>   Author: <u>Chang Shu</u>

##### Purpose:

Since we are allowed to use 2 beacons in the project, I choose the green block as the beacon. Then we transform the beacon detection problem into a color detection problem.

##### Procedure:

1. place a green block on targeted place
2. find its histogram in HSV space
3. use Python GUI to fine-tune the range value

##### Experiment:

Here, we place a green block on the left side of the bridge.

![green](E:\python\pycharm\TDPS\imgs\green.jpg)

The histogram of green is shown below.

![green_hsv](E:\python\pycharm\TDPS\imgs\green_hsv.png)

<center>pure green in HSV space<center>

Then I use the GUI to find the range value.

![green_thresh](E:\python\pycharm\TDPS\imgs\green_thresh.png)

##### Results:

Finally, I obtain the range value of green in HSV space as follows.

| color | [$h_{min}$, $h_{max}$] | [$S_{min}$, $S_{max}$] | [$V_{min}$, $V_{max}$] |
| ----- | ---------------------- | ---------------------- | ---------------------- |
| green | [55, 63]               | [236, 255]             | [156, 173]             |



