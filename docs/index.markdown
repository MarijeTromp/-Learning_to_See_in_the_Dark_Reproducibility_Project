---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults
title: ""
layout: page
---

## Index
1. [Introduction](#introduction)
2. [Value of a reproduction](#value-of-a-reproduction)
3. [Paper summary](#paper)
3. [Main project goals](#goals)
4. [Method](#method)
5. [New data](#new-data)
6. [Hyperparams check](#hyperparams-check)
7. [Ablation study](#ablation-study)
8. [Conclusions and final remarks](#conclusions-and-final-remarks)
9. [Authors](#authors)
10. [References](#references)


## Introduction <a name="introduction"></a>
In this project we tried to reproduce the results of the Learning to See in the Dark (cite) paper as part of the Reproducibility Project for the CS4240 Deep Learning (2022/23 Q3) course at Delft University of Technology.

## Value of a reproduction <a name= "value-of-a-reproduction"></a>
Doing a reproduction project is important. If a paper has never been reproduced it is not possible to know if the findings are correct or not. It is possible that the original authors have made a mistake, or that they cherry picked certain results. By trying to reproduce the results of a paper we can try and see if the results are accurate or not. Reproducing the results is also important because it shows how applicable a method is. If it is possible to reproduce a result but very difficult, it can be that while the findings of a paper are still important, they are not very applicable in the real world.

## Paper summary <a name="paper"></a>
This is a paper

## Main project goals <a name="goals"></a>


## Method <a name="method"></a>
The original data set from the authors of the paper is too large to use for the many different experiments we wanted to run during our limited timeframe. Therefore we picked a subset of 10 train images and 10 test images. Half of the train and test set are images taken inside and half of the train and test set are images taken outside. 

To keep the experiments fair we used the same train and test set for all experiments, except for the new data experiments. For the new data experiments we created new data sets which also consist of 10 train images and 10 test images. For each new data set half of the images was taken inside and half of the images was taken outside. 

## New data <a name="new-data"></a>
For the new data section of this project we created 5 new data sets with 5 different devices. Each data set exists of 10 train images and 10 test images. For each of these train and test sets half of the images were taken inside and half of the images were taken outside. 

### Canon 90d (camera)
The Canon 90d has a 32,5 Megapixel APS-C CMOS-sensor sensor that can shoot CR3 14-bit RAW images and has a Bayer filter ("Canon EOS 90D-camera", n.d.). Since the Sony images created by the authors of the paper also has a Bayer filter, we can use the same training file for these images. We used a tripod to keep the camera steady while taking the different images. However, since we did still have to touch the camera to change the settings and take the images it was impossible to prevent the camera from slightly shifting between different shots. 

To create the images we took bright images and dark images to function as the long and short images in the data sets. In the paper they show that both their ISO and exposure time varies between the long and short images. So, to create the Canon 90d data set we played around with the ISO and exposure time for each photo, trying to get images that looked to be of similar light levels as the original data set. However, what we did not pay sufficient attention to was the fact that they mention that the exposure of the long photo is 100-300 times larger than for the short photo. For the photos in this data set the ratio between the exposure is significantly smaller for most pairs of photos, with for quite a few photos this ratio being below 10. This turned out to be a problem.
![](./images/Canon90d_1_small.png)
The image above shows how the output images looked after training the model for the first time.
As we will see later some of these results are quite similar to results for the differing amplification rates. This pointed to the fact that there was something wrong with the ratio of the exposure between the long image and the short image, which was indeed the case. For these images the exposure time in the image names was not correct. The train and test code uses the exposure time from the image name, so therefore it is important for these to be correct. Note: for this trained model the ratios were between 100 and 300 for each long and short image pair. 

After the training the model for a second time with the correct exposure times in the image name we got the following outputs:
![](./images/Canon90d_2.png)
As we can see, these images are somehow even worse than the ones with the incorrect exposure times. The average PSNR/SSIM calculated on the test set for this model turned out to be 28.61/0.45. 

To try and fix the apparent problem in the training process we used the exact same method that was used to train the model on the images made with the other camera. This turned out to give even worse results:
![](./images/Canon90d_3.png)

The results of these three training sessions point to the fact that the ratio between the exposure time of the long images and the short images. To determine if this is indeed the case we ran one more experiment. Instead of using the ratio between the exposure time of the long image and the short image we just set the ratio to 100. The following images are the results of this experiment:
![](./images/Canon90d_4.png)

While the output is far from perfect, it is significantly better than when we use the actual ratio. The average PSNR/SSIM for the output of this model is 29.20/0.48.


During the training of the model the code only uses the exposure time, not the ISO. When you change the ISO instead of keeping the ratio correct, the results are not good. So, the results of this new data set show how important it is to keep in mind that the ratio between the exposure time for the long images and the short images should be between 100 and 300. Only then should the ISO be changed to get a sufficiently (un)lit image. It also shows how difficult it is to get a good data set. Of course it is also possible that the problem lies somewhere else, but the results from these experiments seem to point in this direction.

### OnePlus 7 (phone)
The OnePlus 7 has a Sony IMX 586 camera sensor ("OnePlus 7", n.d.). We were not able to find the bit-depth for this camera, and therefore have assumed it was 10. The results for these images were on par as for the Canon 90d. 

![](./images/OnePlus7_1.png)
The first time training we did not have the correct exposure time in the image names either, resulting in the outputs in the image above. 

![](./images/OnePlus7_2.png)
However, just like for the Canon 90d, changing the exposure times to the correct ones did not make the results better. Doing this resulted in the outputs in the image above. While they are better than for the first trained model, they are still far off of the ground truth images.

To make sure the problem did not have anything to do with the bit depth, we also trained the model using a bit depth of 8. However, these results were significantly worse, so it is likely that this is not the problem. 

For these images the exposure ratio again is mostly not between 100 and 300. Therefore it is likely that the ratio is again the problem. 

## Hyperparams check <a name="hyperparams-check"></a>

### Learning rate
For the learning rate, we wanted to find out why the specific values were chosen. In the default version, the first 2000 epochs use a learning rate of 1e-4, wheras the final 2000 epochs use a learning rate of 1e-5. To inspect the feect of the learning rate, we try different learning rates or different combinations there of. The following learning rates are used, with the corresponding PSNR and SSIM scores:

|		Learning rate value		| PSNR  | SSIM  |
|-------------------------------|-------|-------|
| Default						| 30.10 | 0.80  |
| 1e-3							| 28.22 | 0.006 |
| 1e-4							| 30.09 | 0.80  |
| 1e-5							| 29.85 | 0.76  |
| 1e-6							| 28.99 | 0.70  |
| 1e-7							| 28.23 | 0.32  |
| 1e-3&1e-4 (after 100  epochs) | 29.02 | 0.60  |
| 1e-3&1e-4 (after 500  epochs) | 28.20 | 0.004 |
| 1e-3&1e-4 (after 1000 epochs) | 28.21 | 0.007 |
| 1e-3&1e-4&1e-5&1e-6			| 28.20 | 0.007 |

These different learning rate (combinations) will visually be evaluated towards the following baseline:

![](./images/bike_gt&default.png)

## Ablation study <a name="ablation-study"></a>
During the ablation study, we used the original training code (with the changes made to allow it to run) and only changed the network each experiment. Our ablation study consists of 10 experiments. These experiments mostly existed of removing layers, but for 1 of the experiments we added layers. The train and test images are the same subset consisting of 10 train and 10 test images from the original data set that we talked about in the Method section.

![](./images/OriginalNetwork.png)
The original network is shown above. As we can see in the image, the network mostly consists of pairs of convolutional layers. These are the layers we targeted in our ablation study. 

![](./images/AblationNetworks.png)
In the image above we show the networks for each respective experiment. Experiment 2 is the experiment in which we added layers to the network. We also tried to do another experiment where we added 2 pairs of layers as in the middle of the original network, but this took too long to run. 


| Ablation | PSNR  | SSIM |
|----------|-------|------|
| Default  | 30.43 | 0.83 |
| 1        | 30.08 | 0.79 |
| 2        | 30.06 | 0.80 |
| 3        | 30.09 | 0.77 |
| 4        | 30.04 | 0.79 |
| 5        | 29.79 | 0.73 |
| 6        | 29.53 | 0.67 |
| 7        | 29.90 | 0.73 |
| 8        | 30.00 | 0.78 |
| 9        | 30.18 | 0.79 |
| 10       | 28.20 | 0.39 |

The table above shows the results for all of the experiments that we ran. As we can see in the table the general trend is that the more layers we remove from the network, the more the performance goes down. We will now look a bit more in depth into these results.

The fourth point of note is the performance difference between the experiments. While the performance of the networks goes down as we remove layers, it is not the same for each pair. 

|             | Difference (PSNR/SSIM) |
|-------------|------------------------|
| Default - 1 | 0.35/0.04              |
| 2 - Default | -0.37/-0.03            |
| Default - 3 | 0.34/0.06              |
| 3 - 4       | 0.05/-0.02             |
| 4 - 5       | 0.25/0.06              |
| 5 - 6       | 0.26/0.06              |
| 6 - 10      | 1.33/0.58              |
| Default - 9 | 0.25/0.04              |
| 9 - 8       | 0.18/0.01              |
| 8 - 7       | 0.10/0.05              |

The table above shows the difference in performance between pairs of experiments that correspond to the steps in removing layers. 
As we can see the difference between the pairs varies a lot. For the pair of experiment 3 and 4 the difference is negligible since the PSNR goes down but thee SSIM goes up. 
We can see that the largest difference is between the pair of experiments 6 and 10. This is to be expected, since for experiment 10 the network consists of only 1 layer. 

The second point of note is that for experiment 2, where we added a layer to each pair, the performance is lower than the default. Instead, the performance is similar to the performance of the network in experiment 1. 

The third point of note is that the middle pair of layers, g_conv5_1 and g_conv5_2, improve the performance of the model significantly. We can see this by comparing the performance between sets of experiments where the only difference is whether those two middle layers are there or not. These pairs are:

| Without/With | Without (PSNR/SSIM) | With (PSNR/SSIM) | Difference (PSNR/SSIM) |
|--------------|---------------------|------------------|------------------------|
| 3 & Default  | 30.09/0.77          | 30.43/0.83       | 0.34/0.06              |
| 4 & 9        | 30.04/0.79          | 30.18/0.79       | 0.14/0.00              |
| 5 & 8        | 29.79/0.73          | 30.00/0.78       | 0.21/0.05              |
| 6 & 7        | 29.53/0.67          | 29.90/0.73       | 0.37/0.06              |

For all of these pairs there is a significant performance increase by adding this pair of layers. When we compare the difference in performance in this table with the difference in performance in the table for the first point of note, we can see that adding these two layers gives a similar and in some cases larger improvement of performance than adding multiple pairs of layers. 

The fourth point of note becomes clear when visually inspecting the output the networks created from the test set.
![](./images/AblationImagesSmall.png)
The image above shows the output for the same image for each experiment. While the quality of the images does degrade slightly (e.g. more noise, colours less vivid) as we remove layers, the images overall still look pretty good for every experiment except for experiment 10. For output image of experiment 10 the colours look similar to the one shown. 
When we ignore experiment 10, experiment 6 and 7 are the experiments that have the least amount of layers. While the image quality is not as good as the for the other experiments, the overall image quality is still relatively high. 
Lastly we'd like to note a few things with respect to the colours in the images. For experiment 2, the only experiment in which we added extra layers, the colours differ quite a lot from the others. These colours are not as close to the ground truth as some of the other outputs. For experiment 9, where we removed 2 pairs of layers, the colours are actually closer to the ground truth than the default image. The grass is greener and the train track is slightly more blue. 

So, in this ablation study we have shown that while all layers of the network are important for creating output, the gain per added layer is not large. However, it is important to note that these results are achieved by training the networks on only 10 images. It is definitely possible that there would be larger differences when training on the entire train set. 

Note: If you are doing an ablation study, make sure that the network in the test file is the same as the one from the train file. Otherwise the model will only give a black image as output. 

## Conclusions and Final remarks <a name="conclusions-and-final-remarks"></a>


## Authors <a name="authors"></a>
#### Group 5
Remco Huijsen (5650844) e-mail: New data, Hyperparams check<br>
Floor Joosen (4814495) e-mail: New data, Hyperparams check<br>
Marije Tromp (4933656) m.r.tromp@student.tudelft.nl: New data, Ablation study<br>

## References <a name="references"></a>
"Canon EOS 90D-camera". (n.d.). Retrieved from https://www.canon.nl/cameras/eos-90d/specifications/
"OnePlus 7". (n.d.). Retrieved from https://www.oneplus.com/nl/7/specs