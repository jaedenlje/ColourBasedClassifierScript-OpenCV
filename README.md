
# Colour Based Classifier Script (OpenCV)
## Description
This project is designed to detect specific shapes and colors in traffic light images. The program processes images, segments them by color using the HSV colour model, and identifies shapes such as circles and arrows using contour detection and matching techniques with shape specimens.

## Table of Contents
- [Installation](#installation)
- [Usage](#usage)
- [License](#license)

## Installation
Install [PyCharm Community Edition](https://www.jetbrains.com/pycharm/download/?section=windows)


1. Create a new project:

![Screenshot (5)](https://github.com/user-attachments/assets/505ebcc0-a23f-41de-8e75-bd82759452ce)


2. Open settings and select "Python Intepreter" under your project. Click on the '+' sign and search for "opencv-python" to install the OpenCV package:

![Screenshot (2)](https://github.com/user-attachments/assets/1bc46e42-2b96-404d-8a32-f3347c3db87d)
![Screenshot (6)](https://github.com/user-attachments/assets/a913794f-e252-47f4-84ee-5599aa880fb0)
![Screenshot (7)](https://github.com/user-attachments/assets/18c56eba-8351-470a-b263-fdf4a6077608)

3. Create a new Python file:

![Screenshot (4)](https://github.com/user-attachments/assets/7344ef74-ca51-4d8e-be36-91933edf2906)

## Usage
To use the script, you need to provide the paths to the shape specimen images and the test folder containing images. You can download the shape specimen images from the internet (e.g., Circle, Left Arrow, Right Arrow and etc.)

    1. Update the circle_contour, leftarrow_contour, straightarrow_contour, rightarrow_contour, and test_folder variables in the script with the appropriate paths.
    2. Run the script:

### Example
    import cv2
    import numpy as np
    import os

    max_contours = -10

    # Define color thresholds in HSV
    red_threshold = (0, 100, 100)
    green_threshold = (np.array([70, 90, 100]), np.array([100, 120, 140]))
    yellow_threshold = (30, 100, 100)

    # Define shape contours
    circle_contour = cv2.imread('/path/to/your/shape/specimen/image', 0)
    leftarrow_contour = cv2.imread('/path/to/your/shape/specimen/image', 0)
    straightarrow_contour = cv2.imread('/path/to/your/shape/specimen/image', 0)
    rightarrow_contour = cv2.imread('/path/to/your/shape/specimen/image', 0)

    # Load test image folder
    test_folder = "/path/to/your/test/folder"

    # Loop through all images in the test folder
    for filename in os.listdir(test_folder):
        # Load image

        image_path = os.path.join(test_folder, filename)
        image = cv2.imread(image_path)
        image2 = image[:, 0:512]

        # Convert image to HSV
        hsv_image = cv2.cvtColor(image, cv2.COLOR_BGR2HSV)

        # Segment image by color
        red_mask = cv2.inRange(hsv_image, red_threshold, red_threshold)
        green_mask = cv2.inRange(hsv_image, green_threshold[0], green_threshold[1])
        yellow_mask = cv2.inRange(hsv_image, yellow_threshold, yellow_threshold)


        # Find contours in each color segment
        red_contours, _ = cv2.findContours(red_mask, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)
        green_contours, _ = cv2.findContours(green_mask, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)
        yellow_contours, _ = cv2.findContours(yellow_mask, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)

        # Classify shapes
        for contours, color in [(red_contours, 'red'), (green_contours, 'green'), (yellow_contours, 'yellow')]:
            for contour in contours:
                area = cv2.contourArea(contour)
                x, y, w, h = cv2.boundingRect(contour)
                aspect_ratio = float(w)/h
                if cv2.arcLength(contour, True) > 0:
                    circularity = 4 * np.pi * area / (cv2.arcLength(contour, True) ** 2)
                else:
                    circularity = 0

                if circularity > 0.8 and aspect_ratio > 0.5:
                    # Circle shape detected
                    if color == 'red':
                        print(f'Red circle detected in {filename}')
                    elif color == 'green':
                        print(f'Green circle detected in {filename}')
                        contours, _ = cv2.findContours(green_mask, mode=cv2.RETR_EXTERNAL, method=cv2.CHAIN_APPROX_NONE)

                        if len(contours) > 1:
                            # get the [max_contours] biggest areas
                            contours = sorted(contours, key=cv2.contourArea)[max_contours:]
                            # mask where contours are filled
                            mask = np.zeros_like(image2)
                            # draw contours and fill
                            cv2.drawContours(mask, contours, -1, color=[255,255,255], thickness= -1)

                            cv2.drawContours(image2, contours, -1, 255, 2)
                            cv2.imshow("Result", np.hstack([image2, mask]))
                            cv2.waitKey(0)
                else:
                    # Arrow shape detected
                    if color == 'green':
                        match = cv2.matchShapes(contour, rightarrow_contour, cv2.CONTOURS_MATCH_I1, 0.0)
                        print(match)
                        if match < 0.1:
                            print(f'Green right arrow detected in {filename}')
                        else:
                            match = cv2.matchShapes(contour, leftarrow_contour, cv2.CONTOURS_MATCH_I1, 0.0)
                            if match < 0.1:
                                print(f'Green left arrow detected in {filename}')
                            else:
                                match = cv2.matchShapes(contour, straightarrow_contour, cv2.CONTOURS_MATCH_I1, 0.0)
                                if match < 0.1:
                                    print(f'Green straight arrow detected in {filename}')
                    elif color == 'red':
                        match = cv2.matchShapes(contour, straightarrow_contour, cv2.CONTOURS_MATCH_I1, 0.0)
                        if match < 0.1:
                            print(f'Red straight arrow detected in {filename}')
                        else:
                            match = cv2.matchShapes(contour, leftarrow_contour, cv2.CONTOURS_MATCH_I1, 0.0)
                            if match < 0.1:
                                print(f'Red left arrow detected in {filename}')
                            else:
                                match = cv2.matchShapes(contour, rightarrow_contour, cv2.CONTOURS_MATCH_I1, 0.0)
                                if match < 0.1:
                                    print(f'Red right arrow detected in {filename}')
                    else:
                        match = cv2.matchShapes(contour, straightarrow_contour,cv2.CONTOURS_MATCH_I1, 0.0)
                        if match < 0.1:
                            print(f'Yellow straight arrow detected in {filename}')
                        else:
                            match = cv2.matchShapes(contour, leftarrow_contour, cv2.CONTOURS_MATCH_I1, 0.0)
                            if match < 0.1:
                                print(f'Yellow left arrow detected in {filename}')
                            else:
                                match = cv2.matchShapes(contour, rightarrow_contour, cv2.CONTOURS_MATCH_I1, 0.0)
                                if match < 0.1:
                                    print(f'Yellow right arrow detected in {filename}')

## License
This project is licensed under the [MIT License](https://www.mit.edu/~amini/LICENSE.md).



