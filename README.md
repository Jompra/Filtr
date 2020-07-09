# Filtr

## Introduction
This Full stack app is an image editor for users to add colouring and interesting effects to their photographs and images. Although challenging this app was great fun to build. Find the deployed version [here](http://gkj.me.uk/filtr). Please be aware it could take up to 45 seconds for Heroku's dyno to wake up.

[Purvi Trivedi](https://github.com/purvitrivedi) and I completed this project in 7 days. We used Trello as our issue tracking and feature planning board.

![Screenshot of Filtr App](https://github.com/Jompra/filtr/blob/master/FiltrScreenshot.png)

## Install Instructions
I recommend using the deployed version of this app [here](http://gkj.me.uk/filtr). However, follow below if you want to run it locally.
I assume that you have an up to date version of Python 3, Pip, NPM, PostgreSQL, and pipenv or similar Python virtual shell available on your machine.

You will also need a Cloudinary account that includes an upload preset with these image transformation settings: c_limit,cs_srgb,h_500,w_1000.

* Fork and clone the repo
* `$ pipenv shell` to create and enter the Virtual shell
* `$ pipenv install` to install dependencies from the pipfile. (Be patient, they're sizeable)
* `$ brew services start postgresql` to confirm PostgreSQL is running, and start it if it isn't.
* `$ createdb image-editing-db` to create database
* `$ python3 manage.py migrate` to create default tables in database
* `$ python3 manage.py loaddata images/seeds.json` to add all filters to the database
* `$ cd frontend/` to move into frontend folder
* `$ echo REACT_APP_IMAGE_UPLOAD_URL=[YOUR CLOUDINARY UPLOAD URL] >> .env` to create environment variables and add your upload url
* `$ echo REACT_APP_IMAGE_UPLOAD_PRESET=[YOUR CLOUDINARY UPLOAD PRESET NAME] >> .env` to add the upload preset to environment variables
* `$ npm install` to install frontend dependencies
* `$ cd ..` to move back into root directory
* `$ python3 manage.py runserver` to start app
* In a browser (Chrome Recommended) navigate to http://127.0.0.1:8000 or the 127 address suggested in terminal if different.

## Technologies Used
* React.js (Hooks)
* Python
* Django
* PostgreSQL
* SASS
* Scikit image
* Pillow (PIL Fork)
* Base64 encoding
* Django rest_framework
* Matplotlib
* Numpy
* Konva
* Axios
* Bulma
* HTTP-proxy-middleware
* Git/GitHub

## Brief
Create a full stack Django and React application.

We decided that an image editor would be an interesting product to create, and totally different to our other projects. 

![Demo of Image editor working](https://github.com/Jompra/filtr/blob/master/Image-Edit-Demo.gif)

This gif is compressed to save space. Images edited in Filtr output at the same quality as the original.

## Planning and Prototyping
When we initially decided to create the image app using Scikit Image we quickly realized that we would have two major unknowns that if left unsolved could immediately halt the project. Firstly how do we get the image between the user's frontend and the python image libraries without burning through our Cloudinary credits, and secondly is it possible to use Matplotlib's graph outputs as a useable PNG or JPG.

To overcome the first unknown, I researched using Base 64 encoding to encode the images as a string of text so that the image could be included in JSON.

Secondly, we prototyped using Matplotlib's savefig function using Google Collaboratory, which is essentially a Jupyter Notebook hosted by Google. Although there were problems with the savefig function including whitespace and needing to remove graph axis, we had proof of concept and decided that we would be able to overcome these issues.

We mapped out the user journey to include the transfer of data to highlight any further blockers that we were likely to face.
![User Journey and Data Flowchart](https://github.com/purvitrivedi/image-editing-app/raw/master/frontend/src/assets/flowchart.png)


## Functionality

### Backend
The backend of the app utilizes Django and the Django Rest Framework to get information between the frontend interface and the backend image processing.

The app utilizes [Sci Kit Image](https://scikit-image.org/) for histogram matching, RAG Thresholding, and Gaussian Edge Detection to create pyplots from user's images. To deal with adding text and ensuring proper image sizing [Pillow](https://python-pillow.org/) is used.

![Back End Editing Options](https://github.com/Jompra/filtr/blob/master/Backend-Editing.png)

All of the filters are unique in how they achieve their output. Commented examples below are for the Tint and Histogram Filters.

#### Tint Filter
The tint filter takes the image, recalculates the image to grayscale, calculates where soft and hard gradients exist within the image and then re-renders the image in the user's chosen color space. Copy and paste it into a Collab or Jupyter notebook and have a play with the options. Change the image URL to another image and try changing color_space to 'blues', 'copper', or 'pink_r'.

```python
from skimage.color import rgb2gray
from skimage import filters, io, exposure
from matplotlib import pyplot as plt

image = 'https://images.unsplash.com/photo-1593720737821-ce72f91b3db8?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=934&q=80'


color_space = 'pink' # select the filter tint you would like to apply


# rgb2gray takes the 3D array created by io.imread and calculates the luminance of each pixel. 
# It then returns a 2D array containing only the luminance of each pixel

image = rgb2gray(io.imread(image))

# Gaussian filter helps reduce noise to keep the image quality high

edge = filters.gaussian(image, sigma=0.6)

# Plot converts 3D array of pixels into a graph of pixels.

fig, axes = plt.subplots(ncols=1, sharex=True, sharey=True, figsize=(12, 12))

axes.imshow(edge, cmap=color_space) #Then apply filter, save figure  with a color space.

axes.axis('off') # Remove the plot axis

# Save it as a PNG
output_filename = 'test'
plt.savefig(f'{output_filename}.png', bbox_inches='tight', pad_inches = 0)
im.save(f'{output_filename}.png')
```

#### Histogram Filter Option
The histogram filter takes the user's image, calculates the image's histogram ([Wikipedia Image Histogram](https://en.wikipedia.org/wiki/Image_histogram)) and matches this to a histogram of a reference image chosen by us. Information on how this algorithm works [here](https://en.wikipedia.org/wiki/Histogram_matching).

```python
reference = io.imread(ref)
image = io.imread(path)

# This checks whether the filter is being applied to a user's image or to a thumbnail. 
# Producing smaller images improves algorithmic efficiency for both memory and processing.
if thumbnail == True:
    size = (1, 1)
else:
    size = (8, 8)

# Checks whether the user's image or reference image pixel data includes an Alpha channel. 
# If it does it removes the alpha channel using Alpha Compositing.
print(image.shape)
if image.shape[2] == 4:
    image = rgba2rgb(image)

if reference.shape[2] == 4:
    reference = rgba2rgb(reference)

# Create an array of the user's image with the reference image's histogram applied
matched = match_histograms(image, reference, multichannel=False)

# Plot the array as a graph
fig, axes = plt.subplots(
    nrows=1, ncols=1, figsize=size, dpi=200, sharex=True, sharey=True)
axes.set_axis_off()

# Ensures that all of the data in the array is a uint8 data type (integer between 0 and 255)
axes.imshow(matched.astype('uint8'))

# Removes whitespace from around image
plt.tight_layout()

# Saves the image as a temporary file with a unique filename depending on whether 
# the image has been previously saved to filesystem or is a URL
if path[:4] == 'http':
    output_filename = path.split('/')[6]
else:
    output_filename = path.strip('.png')
print('hist ran ok')
plt.savefig(f'{output_filename}.png', bbox_inches='tight', pad_inches=0)
return output_filename
```

### REST API
Filtr uses [Cloudinary](https://cloudinary.com) to host the image that's originally uploaded by the user as well as to host the final edited image so that the user can access it via a url post editing. The easiest way to store and recall images is via a URL however as users can potentially test out filters an unlimited amount of times this would eat up our Cloudinary credits very quickly. To bypass this issue we make use of base 64 encoding of the image so that we can send the image in the JSON response to the frontend. This is also faster than uploading and downloading the image to a third-party service.

### Client Side Editing
CSS filters such as Grayscale, Contrast, Sepia, Saturation, and Blur add tweaking to the user's images that would be slow to update if done server side.
In order to allow the user to download their image after front end edits have been made we had to use a canvas element. To make this easier to implement we used [Konva](https://konvajs.org/). 
I was able to implement drag and drop emojis and allow users to add exciting icons to their images as well as basic image filters.

![Front End Editing Options](https://github.com/Jompra/filtr/blob/master/Frontend-Editing.png)

## Task Splitting
Purvi and I used Trello to ensure we didn't miss any features or bugs and meant that we were not at risk of working on the same feature at the same time. We both worked on every part of the app but we felt that it was important to lead features independently as this meant that features could be built quickly and efficiently.

I took ownership of:
* Proof of Concept for Tint, Histogram, and Artist Brush filters
* Image model routing
* Ensuring text on memes were centred and sized depending on user's input.
* Thumbnail generation
* CSS + Konva Filters.

Purvi took ownership of:
* Proof of Concept for Meme Filter
* Selecting filter options and quality control for Tint, Histogram & Artist Brush filters
* Ensuring users are able to memeify an image with a filter on it
* Front-end styling for all pages
* Resetting filters & hover effect.

During the project Purvi and I pair coded using Visual Studio Live Share whilst bug fixing and working on features that directly impacted features that we had taken individual ownership.

## Wins and Blockers

To calculate whether the meme characters should be broken into multiple lines I used Sketch to physically measure the average character width of the Impact font at different font heights. I worked out that Impact has an average width to height ratio of 0.505. Using pillow, I was then able to get the width of the image itself. Using these two pieces of data I was then able to decide whether the string should be split using a `\n` newline character. The position of the bottom text is then also modified to be more visually appealing if the text covers two lines.

A blocker that I personally struggled to overcome was outputting a base 64 dataURL from the canvas context so that the user could download it. I expected that this was due to my inexperience using Hooks in React. I reached out to Anton Lavrenov who created the Konva library and he helped shed some light on what I needed to fix.

## Key Takeaways
* HTML5 Canvas is an incredibly powerful element, but difficult to use in React. Thankfully tools such as Konva-React make it a lot easier to use.
* React Hooks are the future! Although initially using hooks in functional components seemed to complicate the program, in the end I believe that they contribute to better and more readable code.
* I learnt a lot about using 2D and 3D arrays in python. Scikit, pillow, Matplotib
