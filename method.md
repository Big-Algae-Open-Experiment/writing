The image analysis pipeline has been developed in Python using the OpenCV computer vision library and consists of: i) Detection of the calibration window; ii) Image normalization; iii) Prediction of algal density.

Detection of the calibration window
-----------------------------------

Images are converted to greyscale and have a Gaussian blur applied to them before they are subjected to a threshold.
The threshold converts images into black and white images, to which a contour finding algorithm is applied.
In order to find the three anchor points of the calibration window the algorithm iterates over the list of contours and finds contours whose areas are in the correct ratios.
Contours are arranged into a tree like structure, with a contour's parent being the contour which contains it.
The ratios which the algorithm looks for are a 9:16 area ratio between a conotur and its parent and a 9:25 area ration between a contour and its grandparent.
Once three anchor points are found, the image is transformed to orient the picture correctly and correct any skewing of the image.

Image normalization
-------------------

Using the positions of the anchor points, the coloured squares and the transparent window are located in the image and the pixel information for them extracted.
To allow the algorithm to be run in reasonable time, the pixels may be down sampled.
To normalize the colours, a Gaussian process model is applied to each colour channel (red, green and blue) separately.
Gaussian processes are a probabilistic framework used to model unknown functions, and are used in this study because of the unknown non-linearities involved when normalizing photos taken using different equipment and lighting conditions.
They are implemented in the algorithm making use of the GPy Python library.
To parameterize the Gaussian process model, pixel information from the coloured squares and from the black squares within the anchor points are used.
Positional as well as RGB values of the pixels are combined to ensure that variation due to the camera and lighting can be taken into account.
The Gaussian process model is constructed using a linear kernel combined with a squared exponential kernel.
This allows the model to capture the general linear trend while also allowing for non-linearities to be considered by the model.
The pixel information from an image as well as from a reference image of the calibration window are used to train the model.
Once parameterized, the model is used to normalize the pixel information from the transparent window of the image which corresponds to the colour of the algae in the bioreactor.

Prediction of algal density
---------------------------

In order to relate the colour of the algal reaction mixture to algal density, a single colour value for each channel is required.
For each pixel normalized using the Gaussian process model, a normalized mean value and a variance value are given as output.
Finding the mean colour value of the normalized pixel means would not incorporate the variation observed for each pixel.
To incorporate the normalization variation, a colour value is sampled from a normal distribution parameterized using the mean and variance of the normalized pixel.
A mean and variance are then calculated from the sampled values to get an overall colour value for each channel based on the colour of the algal reaction mixture.
The colour values for the algal reaction mixture are used as inputs to another Gaussian process model to allow for prediction of the algal density measurements.
