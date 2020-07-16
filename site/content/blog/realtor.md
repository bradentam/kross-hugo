---
title: "Photo-realistic Neighborhood Image Synthesis"
date: 2020-07-15
image: "images/blog/realtor/realtor.png"
description: "This is meta description."
draft: false
---


For the last 2 months of the [UBC Master of Data Science](https://masterdatascience.ubc.ca/) program, I had the amazing opportunity to do my capstone project in deep learning and computer vision. We worked with [realtor.com](https://realtor.com) and the main objective of the project was to develop a deep learning model that is able to generate images of houses that are unique and realistic for use on realtor.com's website. This post assumes basic knowledge of deep learning.

#### Background information

If you browse on realtor.com, you'll notice that many of the listings have missing images, images from google maps, or images that don't look very professional. This problem hinders the website's visual appearance and takes away from user experience and satisfaction. The simple solution would be to use stock images, hire a photographer, or buy unique images. The former would affect the google search optimization because stock images aren't unique and the latter two would be quite expensive. Our much cooler solution was to leverage something called generative adversarial networks (GANs) to be able to generate unique, synthetic images.

{{< figure src="https://raw.githubusercontent.com/bradentam/kross-hugo/master/site/static/images/blog/realtor/neighbourhoods.png" width="1110" float="center" >}}

#### Main components

To be honest, this project was quite technical and didn't require much domain knowledge as the project was focused on being a proof of concept. However, this project wasn't as simple as just building a model and calling it a day. There were 3 main components of this project:

- ##### 1. Data preparation

- Despite working with images, a large portion of the project was just getting the data cleaned. Realtor.com provided us with over 217,000 images from 10 different cities in the US. The goal of the model was to be able to generate images given a city and type of image (either an exterior image of a house or a landscape image). In order to do this, we required labelled images, which we had for cities but not for type of images. On top of this problem, there was a wide range of images that weren't useful for training. 

{{< figure src="https://raw.githubusercontent.com/bradentam/kross-hugo/master/site/static/images/blog/realtor/example_images.png" width="1050">}}

- Training the model on data that is highly variable will not produce ideal results because it will be difficult for the model to learn the consistent features of the images, therefore clustering the data set was critical. After manually sifting through some of the images, we found that there were 5 main distinct clusters: exterior, landscape, floorplan, interior, and miscellaneous images. We decided to use the VGG-16 pre-trained image classification model to cluster our images. This works by removing the last prediction layer of the VGG-16 network and replacing it with k-means clustering. [Here](https://medium.com/@franky07724_57962/using-keras-pre-trained-models-for-feature-extraction-in-image-clustering-a142c6cdf5b1) is a great article that explains how to do this in python. The initial results weren't as good as we'd hope because it was purely unsupervised so we decided to fine-tune the network. This required us to do some manual labelling of a couple thousand images, which was honestly not too time-consuming thanks to the [`pigeon`](https://pypi.org/project/pigeon-jupyter/) package that provides a nice user interface within jupyter. After clustering, we threw out the floorplan, interior, and miscellaneous images. 


- ##### 2. Model building and training

- If you've ever heard of deep fakes, GANs are responsible for making such convincing images possible. In a nutshell, GANs are composed of 2 competing neural networks: the generator which acts like an art forger and the discriminator which acts like an art authenticator. The goal of this architecture is for the generator to be able to produce realistic images that will fool the discrimator. I won't go into too much detail about GANs because I want the focus of this post to be about the project as a whole. [Here](https://pathmind.com/wiki/generative-adversarial-network-gan) is a great introductory article if you want to learn more about GANs. GANs are typically very difficult to train due to the nature of consisting of 2 separate neural networks and having complex structures. Generally, we kept both networks very similar in terms of the parameters and the number of layers. For realtor.com, the main objective was to be able to generate images for a specific city and type of image, therefore we developed a conditional GAN by combining the input of the model with respective labels.

- The model training process was by far the most challenging and time-consuming aspect of the project. Making improvements on our model was quite time consuming because in order to see if the changes we implemented took effect, we would need to train the model for at least an hour. For example, training on ~100,000 100 by 100 pixel images took ~3 mins for 1 epoch. Generally, producing high quality results requires at least 300 epochs, which translates to ~15 hours for a full experiment. It was also extremely difficult determining which factors were actually affecting the results because the architecture of GANs is quite multifactorial. Most of our experimentation was done on Google Colab and luckily, we had access to a GPU optimized EC2 instance on AWS which allowed us to run full experiments without too much of a hassle. 

- ##### 3. Model evaluation

- The tricky part with GANs is that there's no objective evaluation method so we mainly evaluated our model via visual inspection. This meant manually going through randomly generated images until we were satisfied with the results. To test whether or not our model was producing images that were actually new and unique, we used cosine similarity to find the closest real image to ensure it wasn't just being copied and pasted. 

{{< figure src="https://raw.githubusercontent.com/bradentam/kross-hugo/master/site/static/images/blog/realtor/most_similar.png" width="1100">}}

#### Results

After 200 epochs, we were able to generate exterior images of houses that looked fairly realistic. Unfortunately, the results for landscape images were not great as we were getting odd images with a mixture of elements. We suspected that the landscape images varied too much to fully take advantage of the GAN architecture.

{{< figure src="https://raw.githubusercontent.com/bradentam/kross-hugo/master/site/static/images/blog/realtor/gan_process.png" width="1100">}}

We also developed a [streamlit](https://www.streamlit.io/) application to demonstrate the model. It allows the user to select a city and image to display a randomly generated image as well as be able to set the seed to reproduce the same image. Because this project was a proof of concept, we weren't too worried about if a generated image of Boston looked like a house you could actually find in Boston. If this project were to go into production, there would be a disclaimer saying that the images are synthetically generated and are simply representatives of what a house in a given city could potentially look like. To our knowledge, no one has ever implemented GANs to successfully be able to generate images of houses.

{{< figure src="https://raw.githubusercontent.com/bradentam/kross-hugo/master/site/static/images/blog/realtor/streamlit.png" width="1100">}}

#### What I learned 

- This may seem very obvious but the word *data* in data science is extremely important and often underappreciated. Real-world data can be quite messy and preparing good quality data is so crucial for setting a strong foundation for a project. A model can only be as good as its training data. We were provided with ~217,000 images but after clustering and removing outliers, we were left with ~186,000 usable images. I'm sure if we put even more time into clustering then that number would be even smaller. 

- There's no point in trying to reinvent the wheel when there are so many good tools online we can take advantage of. It would've been completely unnecessary trying to cluster our training data from scratch. It's amazing how simple and useful transfer learning can be. 

- For this project, we made sure that we were constantly progressing week to week. We created a minimal viable product as quick as possible and iterated the model from there. Especially for GANs, I learned that it takes the same amount of effort to get from 0% to 90% satisfaction as it does to get from 90% to 100% (we never got to 100%). Just like any sort of classification model, the additional effort put into getting from 95% accuracy to 99% accuracy is not worth it unless you're trying to win a kaggle competition. In industry, it's more important to develop a model that has a good balance between accuracy and interpretability, while meeting a deadline and satistfying client needs. 

- If cloud computing didn't exist, we wouldn't have been able to work on this project to the extent that we did. Recently, cloud computing has kick-started a machine learning revolution where we're capable of performing a massive amount of computation using a massive amount of data. Throughout this project, I became familiar with AWS S3 buckets and EC2 instances via their command line interface.

- Although this project was incredibly interesting, unique and used bleeding edge technology, I would've preferred a more business-oriented project so that I could develop my analytical skills greater. Nonetheless, I'm happy I got the chance to experience working with GANs, get more familiar with deep learning topics, and work with real-world data.