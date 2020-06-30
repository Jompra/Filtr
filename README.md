# Filtr

## Introduction
This Full stack app is an image editor for users to add colouring and interesting effects to their photographs and images. Although challenging this app was great fun to build. Find the deployed version [here](http://filtr-app.herokuapp.com). Please be aware It could take up to 30 seconds for heroku's dyno to wake up, so please carry on reading whilst it loads.

[Purvi Trivedi](https://github.com/purvitrivedi) and I completed this project in 7 days. Using Trello as our issue tracking board

## Brief
Create a full stack Django and React application.

We decided that an image editor would be an interesting product to create.

![Demo of Image editor working](https://github.com/Jompra/filtr/blob/master/Image-Edit-Demo.gif)

The gif is compressed to save space. Images edited in Filtr are the same quality as the original.

## Functionality
### Backend
The backend of the app utilizes Django and the Django Rest Framework to get information between the user's frontend interface and our python based image processing.

The app utilizes [Sci Kit Image](https://scikit-image.org/) for histogram matching, RAG Thresholding, and Gausian Edge Detection to create pyplots from user's images. To deal with adding text and ensuring proper image sizing [Pillow](https://python-pillow.org/) is used.
![Back End Editing Options](https://github.com/Jompra/filtr/blob/master/Backend-Editing.png)

### REST Api
Filtr uses [Cloudinary](https://cloudinary.com) to host the image that's originally uploaded by the user as well as to host the final edited image so that the user can access it via a url post editing. The easiest way to store and recall images is via a URL however as users can potentially test out filters an unlimited ammount of times this would eat up our cloudinary credits very quickly. To bypass this issue we make use of base 64 encoding of the image so that we can send the image in the JSON response to the frontend. This is also faster than uploading and downloading the image to a third party service.

### Client Side Editing
CSS filters such as Grayscale, Contrast, Sepia, Saturation, and Blur add tweaking to the user's images that would be slow to update if done server side.
In order to allow the user to download their image after front end edits have been made we had to use a canvas element. To make this easier to implement we used [Konva](https://konvajs.org/). 
I was able to implement drag and drop emojis and allow users to add exciting icons to their images as well as basic image filters.
![Front End Editing Options](https://github.com/Jompra/filtr/blob/master/Frontend-Editing.png)