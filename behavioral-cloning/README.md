# Self-Driving Car Engineer Nanodegree
# Deep Learning
## Project: Behavorial Cloning

### Goal:
Use deep learning to help a car drive by providing the steering angle. It uses the simulator provided by Udacity.

### Dataset:
Although it is possible to generate data with the simulator, I wasn't able to drive the car well enough with my laptop keyboard to get good data. Therefore, I used the dataset provided by Udacity.
The dataset is composed of the following: 
- images from 3 different cameras placed at the center, left and right of the car and 
- their associated steering angle, throttle, brake and speed.

### Data analysis:
- Some points in the dataset were recorded with a very low speed. These may be outliers and I decided to remove them.
- Steering angles have an uneven distribution: there are more sharper left turns in the data set. Also, there is a huge peak around 0, which means that the model may have a bias to go straight. If we include the left and right cameras with an steering angle offset, this can fix the problem.

![Reshaped image](images_readme/distributions.png)

### Validation set:
In order to pick the best model and avoid overfitting, 10% of the training dataset was randomly selected for validation.
- Total number of examples: 7,332
- Training examples: 6,598
- Validation examples: 734

### Model description:
I implemented the model described in the [NVIDIA paper](http://images.nvidia.com/content/tegra/automotive/images/2016/solutions/pdf/end-to-end-dl-using-px.pdf) in Keras. I used the same layers but fed RGB images instead of YUV images since RGB images gave better results.

![NVIDIA model from paper](images_readme/NVIDIA_model.png)
![NVIDIA model implementation in Keras](images_readme/NVIDIA_model_keras.png)

More implementation details:
- Adam optimizer with learning rate of 10e-4.
- Metrics to optimize: minimize mean_squared_error.
- Batch generator in order to generate more random images from the dataset. 
- ModelCheckpoint to save model weights after each epoch. This was useful since I realized that the validation loss was not always better correlated to the performance on the track so I was able to easily run the simulator for models of different epochs.

### Data preparation:
- I cropped the 20 pixels at the bottom of the image to hide the car and 50 pixels at the top to remove information about the environment such as the sky.
- I resized the images to 64x200x3 since that's the expected input for the NVIDIA model.
![Reshaped image](images_readme/reshaped_image.png)

The original dataset includes only around 7,000 images.
My first try using center camera images only without additional data generation performed poorly. 
I then decided to generate additional data in this chronological order:

- Horizontal flip of the image and take the opposite of the steering angle.

![Horizontally flipped image](images_readme/hor_flip.png)

- Translate the image horizontally by a random number of pixels from 0 to 50 pixels in each direction. For each translated pixel, I adjusted the steering angle by 0.008 degres. After trying different values of the adjustment factor from 0.001 to 0.01 degres per pixel, I chose 0.008 since it provided the best results.

![Translated image](images_readme/translated_image.png)

- Use images from all 3 cameras and adjust the steering angle for left/right images. I've tried many different values for the adjustment factor, from 0.1 to 0.5. 0.3 was the adjustment angle that provided the best results.

![Images from Left, Right and Center cameras](images_readme/L_R_C_cameras.png)

- To show the importance of data augmentation: I recorded the track performance of different models with more or less data augmentation. The [Video](http://www.youtube.com/watch?v=EfSK5nApej8) is available on Youtube.


### Model training:
I included around 28,160 examples in each epoch.
I ran the model for at most 10 epochs. I initially selected the model with the lowest validation loss. However, after noticing that some of my earlier models with less data generation had lower validation loss by performed worse on the track, I started to test the simulator for models at different epochs from the one with the lowest validation loss.

I was able to train the model locally on my MacBook Pro in less than 3 minutes per epoch, so 30 minutes total for 10 epochs.

Note: validation loss can be lower that the training loss because training includes more data transformation. Validation only includes the 3 cameras without data augmentation (i.e. no horizontal flipping, no random translation etc.). This choice was made so that I could compare the validation loss across different models.


- Plot of training and validation loss (mean_squared_error)
![Training/Validation Loss](images_readme/training_validation_loss.png)

- Plot of the steering angles of the car driving on the 1st track using my best model
![Steering angles](images_readme/steering_angles.png)

### Output of first and second convoluational layers
For image resized above:
- Output of First convolutional layer:
![Output First Convolutional Layer](images_readme/output_conv1.png)
- Output of Second convolutional layer:
![Output Second Convolutional Layer](images_readme/output_conv2.png)

### Other data generation tried that did not work:
- Change brightness
- Add image rotation
- Convert image to YUV space

### Running the simulation:
- I updated the drive.py file to make the necessary image transformations to the input image before feeding it to the model: crop top and bottom of image and resize it to 200x64x3.
- For Track 2, I increased the throttle to make the car go up the hill (if speed between 5 and 10 mph, increase throttle to 0.5). It is 0.2 by default.

### Observations:
- Car drives well on both tracks.
- At higher speeds, the car makes more zigzags. I wonder if it is a limitation of the simulator that is not fast enough.
- The simulator is not performing well when other applications are running as well (e.g. training another model)
- Some models have lower validation losses but perform poorly on the track.
- When using the silmulator at higher screen resolutions, the model performs as well.
- When using the silmulator with highest graphic quality, my laptop struggles and the car does not drive as well. We could generate random shadows to improve it.

### Images source:
Check out to see the code that generates the images included in this README.

### Architecture of project:
- Train model: Run `python model.py`
- Final model saved weights: model.h5
- Final model saved architecture: model.json
- model.ipynb has the code to generate the graphs included in this README and some extra ones.
- drive.py is used to generate the steering angles used by the simulator
