---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults
title: ""
layout: page
---
## Index
1. [Introduction](#introduction)
2. [Paper summary](#paper)
3. [Main project goals](#goals)
4. [Value of a reproduction](#value-of-a-reproduction)
5. [Method](#method)
6. [New data](#new-data)
7. [Hyperparams check](#hyperparams-check)
8. [Ablation study](#ablation-study)
9. [Conclusions and final remarks](#conclusions-and-final-remarks)
10. [Authors](#authors)
11. [References](#references)


## Introduction <a name="introduction"></a>
In this project, we tried to reproduce the results of the Learning to See in the Dark (Chen et al., 2018) paper as part of the Reproducibility Project for the CS4240 Deep Learning (2022/23 Q3) course at the Delft University of Technology. This paper tries to brighten low-light or seemingly black images such that details of the environment can be uncovered.

It can be pondered on why one would possibly want this capability. Besides the angle of it being scientifically interesting to explore, another angle could be to smarten the software we use, instead of blindly upgrading our hardware. Instead of trying different Bayer-patterns to possibly shoot lighter images in the dark, for instance, we could also deepen our understanding of the currently available hardware and how we can upgrade our software to get the most benefits out of it. While there might naturally be limitations, although these would also extend to hardware alterations, upgrading the software could definitely extend the applications of these hardware components.

The results the paper presents are definitely promising, so it definitely signals a possible path forward. For this blog post, we show and discuss our finding regarding the reproduction of the paper.

## Paper summary <a name="paper"></a>
This paper entailed two major contributions: the See-in-the-Dark (SID) dataset and a pipeline to brighten images with very little to no light present.

The SID dataset consists of images from two cameras: the Sony alpha-7S II and the Fujifilm X-T2. These images come in sets. The ground truth image, sometimes also referred to as the long image, is an image with a high exposure time, which brightens the image in a non-artificial way. This image is paired with one or multiple short exposure time images, also referred to as the short or input image, which the network will try and learn how to transform to the ground truth images. All these images are provided in the RAW format of their respective brands. This means it is raw sensor data compared to a compressed PNG or JPEG. This format is used since it contains the most amount of direct information from the scene. The dataset was created since the authors noted that at the time of writing there was no dataset available with truly dark images such as the ones in the SID dataset.

The Learning to See in the Dark pipeline directly uses these RAW format images for the brightening. This pipeline uses end-to-end training of a fully convolutional network. They explicitly state that more traditional image processing pipelines perform poorly on these dark images. They measure the output quality by two different scores: the Structural Similarity (SSIM) and the Peak Signal-to-Noise Ratio (PSNR). The scores given in the paper for both cameras in the SID dataset are as follows:

|		Device		 |	PSNR     |	SSIM     |
|--------------------|-----------|-----------|
|   Sony alpha-7S II |	 28.88	 |   0.79    |
|   Fujifilm X-T2    |   26.61   |	 0.68    |

## Main project goals <a name="goals"></a>
In this blog post we will showcase the reproduction that we performed of the paper. After reading the paper we formulated a set of goals.

First of, in the paper the authors mention that if the network is trained for one sensor, that it will probably not generalise well to other sensors. As a first goal we therefore set out to test this theory by creating more datasets. The authors mentioned that the network would probably also perform worse on lower bit depth camera's such as those on phone camera's. We therefore also wanted to vary the devices we used to create the datasets. This would then serve the purpose of being able to judge the generalisation power of the network, and determine if it would indeed perform worse for lower bit depth camera's. Finally, we wanted to see whether or not generalisaton would work better for similar bit depth and quality devices rather than from a phone camera to a professional camera.

The second goal was to test how well the network was tuned and how sensitive it is to changes. For this purpose we wanted to perform a hyperparameter check on different hyperparameters in the network.

Finally, we wanted to test the robustness of the pipeline, and explore the functionality of its layers. For this purpose an ablation study was performed.

## Value of a reproduction <a name= "value-of-a-reproduction"></a>
Doing a reproduction project is important for many different reasons. First of all, in science it is important to stay sceptical. If a paper has never been reproduced, it is not possible to know if the findings are actually valid or not. It is possible that the original authors made some mistake, or that they cherry-picked the results. By reproducing the results of a paper, we can try and see if the results are accurate or not. 

Secondly, it shows how applicable or usable a method is. If it is very difficult to reproduce a result, it may be that, while the findings of a paper are still important or valid, they may not be very applicable in the real world. Rather than the method being difficult, it might also be that if the reproduction is difficult,  the authors did not present their findings clearly enough. Which would be a shame for useful research to go to waste.

Another example is that the network presented might be larger than need be. Or that other optimizations can be applied to improve the performance which were missed in the original paper. This would extend the original research and push it to a new level. This is not possible if the original results cannot be recreated. 

## Method <a name="method"></a>
The SID dataset is too large to use for the many different experiments we wanted to run during our limited timeframe. Therefore we picked a subset of 10 long train images and 10 long test images, and their respective short images. Half of the train and test set are images taken inside and half of the train and test set are images taken outside. These images were only selected from the Sony dataset, since the Bayer pattern on this camera is more generally used than that of the Fuji camera.

To keep the experiments fair we used the same train and test set for all experiments, except for the new data experiments. For the new data experiments we created new datasets which also consist of 10 long train images and 10 long test images, and their respective short image. For each new dataset half of the images was taken inside and half of the images was taken outside. Each dataset was created using a different camera sensor (device). 

When training and testing special attention had to be paid to the bit depth of the images of each device. The data has to be adjusted for differing bit depths, otherwise the results will come out wildly different. This is an example of an image with bit depth 14, while the code specified a bit depth of 10:

![](./images/bit_depth_14_on_10.png)

## New data <a name="new-data"></a>
For this part of our experiments, the goal was to discover how well the network generalises to different camera sensors once it has been trained. In order to do so, 5 new datasets were created with the following devices:
- OnePlus Nord 2 (phone)
- OnePlus 7 (phone)
- Samsung Galaxy S22 (phone)
- Canon 90D (camera)
- Canon EOS 100D (camera)

Here we made the distinction of camera's on phones, or dedicated camera's. This is because we wondered if the generalisation would be better for comparable sensors. The specifications of each camera are given in the table below:

| Device				| Sensor Brand | Bit depth | Bayer filter  | Dimensions  |
|-----------------------|--------------|-----------|---------------|-------------|
| OnePlus Nord 2		| Sony         | 10        | Quad          | 4096 x 3072 |
| OnePlus 7				| Sony         | 10        | Quad          | 4000 x 3000 |
| Samsung Galaxy s22	| ISOCELL      | 12        | Tetrapixel    | 4000 x 3000 |
| Canon 90D				| Canon        | 14        | Quad          | 6984 x 4660 |
| Canon EOS 100D		| Canon        | 14        | Quad          | 5208 x 3476 |

Each dataset consists of long and corresponding short images, totaling 10 long training images and 10 long test images each. For each of these train and test sets half of the images were taken inside and half of the images were taken outside. Together with the subset of the SID dataset, this gives the following sets with their abbreviations:
- S: SID-subset
- C1: [Canon 90D](#making-of-dataset-c1)
- C2: [Canon EOS 100D](#making-of-dataset-c2)
- P1: [OnePlus Nord 2](#making-of-dataset-p1)
- P2: [OnePlus 7](#making-of-dataset-p2)
- P3: [Samsung Galaxy S22](#making-of-dataset-p3)

In the abbreviations the P stands for phone, indicating the subset was created using the camera of a phone. C indicates that the subset was created using a dedicated camera. 

The table shows the dimensions of each device. The dimensions of the images in dataset S are 4256 x 2848. For the experiments we limited the images to be 4000 x 2800 to fit every dimension equally. In order to cut the images to size, the top left corner was used.

To test how well the network generalised, we planned on training a model for each type of dataset, and then testing these models also each on one type of dataset:
- (S, S): train model on dataset S, test on dataset S.
- (S, P1): train model on dataset S, test on dataset P1.
- (S, C1): train model on dataset S, test on dataset C1.
- (C1, S): train model on dataset C1, test on dataset S.
- (C1, P1): train model on dataset C1, test on dataset P1.
- (C1, C1): train model on dataset C1, test on dataset C1.
- (P1, S): train model on dataset P1, test on dataset S.
- (P1, P1): train model on dataset P1, test on dataset P1.
- (P1, C1): train model on dataset P1, test on dataset C1.

When making the datasets we ran into several challenges. These challenges are first explained before finally considering the results.

### Making of Dataset C1 <a name="making-of-dataset-c1"></a>
The Canon 90d has a 32,5 Megapixel APS-C CMOS-sensor sensor that can shoot CR3 14-bit RAW images and has a Bayer filter ("Canon EOS 90D-camera", n.d.). Since the Sony images created by the authors of the paper also has a Bayer filter, we can use the same training file for these images. We used a tripod to keep the camera steady while taking the different images. However, since we did still have to touch the camera to change the settings and take the images it was impossible to prevent the camera from slightly shifting between different shots. 

To create the images we took bright images and dark images to function as the long and short images in the datasets. In the paper they show that both their ISO and exposure time varies between the long and short images. So, to create the Canon 90d dataset we played around with the ISO and exposure time for each photo, trying to get images that looked to be of similar light levels as the original dataset. However, what we did not pay sufficient attention to was the fact that they mention that the exposure of the long photo is 100-300 times larger than for the short photo. For the photos in this dataset the ratio between the exposure is significantly smaller for most pairs of photos, with for quite a few photos this ratio being below 10. This turned out to be a problem.
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

During the training of the model the code only uses the exposure time, not the ISO. When you change the ISO instead of keeping the ratio correct, the results are not good. So, the results of this new dataset show how important it is to keep in mind that the ratio between the exposure time for the long images and the short images should be between 100 and 300. Only then should the ISO be changed to get a sufficiently (un)lit image. It also shows how difficult it is to get a good dataset. Of course it is also possible that the problem lies somewhere else, but the results from these experiments seem to point in this direction.

### Making of Dataset C2 <a name="making-of-dataset-c2"></a>
For the making of the dataset C2, the Canon EOS 100D was used. This camera has a 18 MP CMOS sensor that can shoot 14 bit-depth images in CR2 format. This is a Canon version of the RAW format. The sensor has a quad bayer filter and a max ISO of 12800.
  
Difficulties experienced while creating the C2 dataset is that this camera did not have a tripod. Also, the night in question it had been raining. In order to protect the camera a stool had been taken to put the camera on when making pictures. This means every outside image is taken from a relatively low angle. 
  
Another difficulty was that the uncomfortable low angle meant it was hard to manually focus the camera. This is why auto-focus was first used, sometimes in combination with the flash, so that the camera would adjust its own sharpness. Then manual focus was turned on, and the flash was turned off to make the actual pictures. The flash was used since auto-focus did not work at lower light levels, since it would have nothing to focus on. This did mean that the images had to be taken in closer proximity to the objects, so that the flash would illuminate it.
  
These two challenges combined, made the dataset automatically biased towards close proximity objects from a lower angle.
  
Finally, as with the other datasets it was challenging not to move the camera while having to change the settings.

![C2_dataset](./images/F2_dataset.png)
  
### Making of Dataset P1 <a name="making-of-dataset-p1"></a>
Dataset P1 was created using the main camera of the OnePlus Nord 2. This camera has a Sony IMX766 sensor that can shoot 50 MP images with bit-depth of 10 in DNG format. DNG is an open source version of the RAW format. The sensor has a quad bayer filter and a max ISO of 6400.
  
One challenge is that no stand or other tool was available to hold the phone in place while taking pictures. This is why the phone was positioned on top of the surface of an object to try and mitigate movement while taking pictures. This method stopped the phone from moving up, down, left and right during the making of the images. However, it did not stop the angle of the phone from moving, a.k.a. tilting the phone more or less. This resulted in the long and short images still varying w.r.t. one another. This variation was seen when training the model on this dataset, because double lines started appearing in the output images. This can for example especially been seen in this result from the training at epoch 4000. On the left if the ground truth patch taken for training, and on the right is the same patch as output.

![P1_dataset_lines](./images/F1_dataset_lines.png)

This did impact the results of this dataset, as the images are relatively good, but with double lines, as can be seen in this test image.

![P1_dataset_results](./images/F1_dataset_results.png)

These results lead us to believe the network can be well utilised also for phone camera's, as long as a good dataset can be made.
  
### Making of Dataset P2 <a name="making-of-dataset-p2"></a>
The OnePlus 7 has a Sony IMX 586 camera sensor ("OnePlus 7", n.d.). We were not able to find the bit-depth for this camera, and therefore have assumed it was 10. The results for these images were on par as for the Canon 90d. 
When the ISO is set manually, the maximum is 3200. However, when it is automatic the maximum is 6400. Therefore we kept the ISO on automatic and only changed the exposure between shots. However, this caused the ratio to not be within the correct range. 

![](./images/OnePlus7_1.png)
The first time training we did not have the correct exposure time in the image names either, resulting in the outputs in the image above. 

![](./images/OnePlus7_2.png)
However, just like for the Canon 90d, changing the exposure times to the correct ones did not make the results better. Doing this resulted in the outputs in the image above. While they are better than for the first trained model, they are still far off of the ground truth images.

To make sure the problem did not have anything to do with the bit depth, we also trained the model using a bit depth of 8. However, these results were significantly worse, so it is likely that this is not the problem. 

For these images the exposure ratio again is mostly not between 100 and 300. Therefore it is likely that the ratio is again the problem. 

### Making of Dataset P3 <a name="making-of-dataset-p3"></a>
The sensor for the Sony camera is a full-frame Bayer sensor. While all other devices complied with this, we found out that the mobile phone for this section, the Samsung galaxy S22, did not. This phone has a Tetrapixel RGB Bayer pattern, which should just be another terminology for the Quad-Bayer pattern used for all other devices, but we did not manage to get it working with the provided pipeline. Normally, using the following line:

`im = raw.raw_image_visible.astype(np.float32)`

would result in the network giving an image with a depth of one, but for the Samsung Galaxy S22 pictures it gave images with a depth of four. To make this images usable, we took the average of the four layers as follows:

`im = 0.25*im[:,:,0] + 0.25*im[:,:,1] + 0.25*im[:,:,2] + 0.25*im[:,:,3]`

Granted, this does limit the usability of the results, but even with this method, the network does show some promosing results. Given a set of images like the following images:

![](./images/samsung_sample.png)

The following images can be optained:

![](./images/samsung_on_samsung.png)

These results are ofcourse part of the images the network is trained upon, but the following results are from a model trained on dataset M1 for instance:

![](./images/samsung_on_oneplus.png)

This does show that details can actually be retrieved from this images with this method.

### Results of Experiments
Given the challenges described in the previous sections, it was difficult to perform all of the earlier mentioned experiments. We did still run the experiments shown in the table below.
  
| Train set | Test set  | PSNR  | SSIM  |
|-----------|-----------|-------|-------|
| S         | S         |	30.43 |	0.83  |
| P1        |	P1        |	30.65 |	0.7   |
| P1        | S         | 28.61 | 0.59  |
| P1        | P3        |	28.02 |	0.57  |
| P1        |	C1        |	28.07 |	0.39  |

From these results it can be seen that the network is indeed not well generalisable to different devices. We think the small decrease in quality from (S, S) to (P1, P1) is not due to the fact that P1 was made using a phone and S using a camera. Instead we think it is because the dataset long and short images of P1 were slightly shifted, causing double lines to appear in training. However, a better dataset needs to be created using a phone to be sure. If this is the case, the network can be utilised for both professional as well as lower quality camera's. 

The generalisation performing better or worse dependent on the device types cannot be confirmed, since there were issues with datasets P2 and P3. With those issues, (P1, P3) and (P1, S) seem to be comparable in PSNR and SSIM. However, it cannot be said this would be the same if P3 could have been tested properly. (P1, S) does provide a good example of generalising from a phone with bit depth 10, to a camera with bit depth 14 where the quality clearly degrades.

![](./images/new_data_results.png)

## Hyperparams check <a name="hyperparams-check"></a>
In this section we discuss the results of the hyperparameters we have tested.

### Amplification
One interesting hyperparameter we found in the paper was the amplification. In the paper it was mentioned that this needed to be manually determined, but that they defined it as the ratio between the exposure time of the ground truth image (long), and that of the input image (short). In the code we found that a maximum was also defined at 300. 
  
  `ratio = min(gt_exposure / in_exposure, 300)`
  
This ratio is then factored with the values in the image after subtracting the black levels. This magnifies the values making it closer to what is expected in the ground truth which had a longer exposure time. With the way the SID dataset was created, the ratio was always either 100, 250 or 300. 

First we wondered what would happen if we removed the ratio altogether. We did this by hardcoding ratio to be 1. The expectation would be that the image would be much darker. This happened in the most part, as the images were mostly black and white. Something that we did not expect is that the images also had seemingly random applications of bright red and yellow colours. The red seemed to mostly organise in blotches, while the yellow seemed to create borders.
  
  ![amp_1_results](./images/amp_1_results.png)

One hypothesis for the red colouring is that it overtrained on one image of the training set. Red is not very common in those images, except for an image of red flowers.
  
  ![red_flowers](./images/red_flowers.png)

One thing that obviously was clear is that the amplification was a very important hyperparameter for training the network.
  
Next we wanted to know if it mattered what value it was, as long as it was either 100, 250 or 300. The expectation was that it would not matter for the images. The result mostly support this, except for that the outlining that occurred in yellow for ratio = 1 seemed to also occur but than in the 'correct' colour.
  
   ![amp_hardcoded_results](./images/amp_hardcoded_results.png)

So it seems that it is still important to dynamically decide the ratio, rather than hardcoding it to be one value.
  
Next we wanted to see what would happen if the maximum of the ratio was taken away. However, nothing interesting happened in this case, since it dynamically seemed to only go up to 300. Instead we hardcoded the ratio again to be double the current maximum, 600. We wondered if this would make the images brighter. As can be seen above, it did not change the results much from the other hardcoded values.
  
Finally, we also trained and tested on a randomized ratio as a baseline for the results.  
  
The results in PSNR and SSIM can be seen in the table below. This mostly confirms what was already visible in the images above. The default way of dynamically setting the ratio to the ratio between the exposure time of the ground truth and input image is the best method. Hardcoding values made the results worse, but none of the hardcoded values differed much from each other. Not having a maximum seemed to perform worse, but since it came down to the default method this might have been random chance from the patch size training. Randomizing did about as well as hardcoding the ratio.
  
  
|   Ratio   |	PSNR    |	SSIM    |
|-----------|-----------|-----------|
|   Default |	30.43	|   0.83    |
|   1   	  |   28.75   |	0.73    |
|   100     |	29.43	|   0.75    |
|   250     |	29.32   |	0.74    |
|   300     |	28.75   |	0.72    |
|   600     |	29.58   |	0.76    |
|   no max  |	29.97	|   0.79    |
|   random  |	29.05   |	0.72    |

### Epochs
Epochs is another hyperparameter that could be varied in the code. During the first couple times training the network we noticed that it already showed relatively good results at epoch 500 (the first training checkpoint in the code). So, we wanted to test an extremely low amount of epochs to see if this also already had good results. This is why we selected `epoch = 10`. Next, we wanted to test at the first checkpoint, `epoch = 500`. The default is 4000. To try and see if we could get the network to overtrain we also tried `epoch = 8000`.
  
![epoch_results](./images/epoch_results.png)

What can be seen in the results is that at epoch 10 you can already clearly make out what the object in the image is. For so little epochs this was a surprising result. At epoch 500 the colours are starting to be defined and the image is a bit sharper. The default is epoch 4000. Here it is hard to find differences between the ground truth and the output. Epoch 8000 seems comparable. These findings are also largely supported in the PSNR and SSIM results shown in the table below. One thing that wasn't immediately clear is that the quality at epoch 8000 seems slightly less than at 4000. 
  
|   Epochs  |	PSNR    |	SSIM    |
|-----------|---------|---------|
|   10      |	28.25	  | 0.65    |
|   500     |	29.42   |	0.74    |
|   4000    |	30.43   |	0.83    |
|   8000    |	30.16   |	0.81    |

### Learning rate
For the learning rate, we wanted to find out why the specific values were chosen. In the default version, the first 2000 epochs use a learning rate of 1e-4, whereas the final 2000 epochs use a learning rate of 1e-5. To inspect the effect of the learning rate, we try different learning rates or different combinations there of. The following learning rates are used, with the corresponding PSNR and SSIM scores:

|		Learning rate value		| PSNR  | SSIM  |
|-------------------------------|-------|-------|
| Default (1e-4&1e-5)			| 30.10 | 0.80  |
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

![](./images/lr_bike_gt&default.png)

#### Increasing the learning rate
Increasing the learning rate would intuitively result in faster learning, with the added risk that the steps taken are too large for the problem to converge to some minimum. The following bike image is retrieved from a model solely trained with a learning rate of 1e-3. Some resemblance of a bike can be detected, but this is not a desirable result.

![](./images/lr_bike_1e-3.png)

#### Decreasing the learning rate
Decreasing the learning rate would intuitively result in more stable and smooth learning, with the added risk that the steps taken are too small for the problem to converge in a reasonable amount of time. The following bike images are retrieved from models trained with learning rates of 1e-4, 1e-5, 1e-6 and 1e-7. These images show that decreasing the learning rate directly impacts the amount of time necessary to get visible images, with the images getting more blurry and more dark for a decreasing learning rate.

![](./images/lr_bike_1e-4&1e-5&1e-6&1e-7.png)

#### Combination of 1e-3 and 1e-4
Combining the highest learning rate with the best single learning rate, we wanted to experiment if it is possible for the algorithm to go from the psychedelic images from the higher learning rate to something that more resembles the ground truth. We started out with the higher learning rate for both 1000 epochs, 500 epochs and 100 epoch. For the former two, it was not successful into getting the loss below 1, and the images remained very colourful. For the 100 epochs variant, however, the results are actually very muted. This shows 2 interesting things however: 1) An overly colourful and saturated image can eventually be transformed to something more conform the ground truth, and 2) there are artifacts present from this more colourful past. Notice that for example the yellow bike has a certain glow that none of the presented images thus far have.

![](./images/lr_bike_1e-3&1e-4.png)

#### Combination of 1e-3, 1e-4, 1e-5 and 1e-6
Solely out of curiosity, we decided to see what would happen with four learning rates: the one that is too radical, and the three learning rates which are all reasonable. This gave the following result. This hearkens back tot the combination of 1e-3 and 1e-4, but with more detail, be it also psychedelically styled.

![](./images/lr_quad_combi.png)

From this investigation, we can see why the paper chose to combine the learning rates of 1e-4 and 1e-5, since this is almost the best scoring option from this investigation as well. The only option which scores better is the sole use of a learning rate of 1e-4, which is only marginally better here. Intuitively it could make sense to have a more direct and a more nuanced learning rate, so not solely using one learning rate could make sense in that regard.

### Patch size
For the learning rate, we wanted to find out why the specific patch size was chosen. In the default version, a patch size of 512x512 is chosen to train on. To inspect the effect of the size, we try different patch sizer or different combinations there of. The following patch sizes are used, with the corresponding PSNR and SSIM scores:

|					Patch size value					| PSNR  | SSIM  |
|-------------------------------------------------------|-------|-------|
| Default (512)											| 30.10 | 0.80  |
| 1														| 28.35 | 0.25  |
| 64													| 29.66 | 0.77  |
| 128													| 29.85 | 0.78  |
| 256													| 30.01 | 0.79  |
| 650													| 30.17 | 0.80  |
| Random from [1, 2, 4, 8, 16, 32, 64, 128, 256, 512]	| 29.82 | 0.76  |
| Random from [8, 16, 32, 64]							| 28.87 | 0.74  |
| Random from [16, 32, 64, 128, 256, 512]				| 28.90 | 0.77  |
| Random from [64, 128, 256, 512]						| 30.10 | 0.79  |

These different learning rate (combinations) will visually be evaluated towards the following baseline:

![](./images/ps_gt&default.png)

#### Minimizing or increasing patch size
Altering the training patch size seems to effect the coloration of the images the most. In the extreme case of a patch size of 1, very little actual color seems to be present. However, even a patch size of 64 shows more red/blue-ish hues compare to larger patch sizes. Note that a patch size of 650 is the largest patch size we could get the script to accept, so the effect of drastically increasing the patch size remains to be explored. Besides the minimal effect on the score, there was a substantial decrease in computation time. Iterations with a patch size of 64 would run approximately 10 times faster than iterations with a patch size of 512. Although both are still (on the system tested upon) fractions of a second, in the long run this could make a significant difference.

![](./images/ps_singles.png)

#### Randomly selecting patch sizes
Instead of having a static patch size, another option could be to use multiple patch sizes. We initially tried to explore how the algorithm would handle this, and if it would possibly enhance the end result. None of these experiments score drastically better or worse compared to the other options, but these scores are marginally better than most static patch sizes. This could give a good balance between the pipeline result and computation time, since iterations with a smaller patch size might be significantly faster.  

![](./images/ps_randoms.png)

## Ablation study <a name="ablation-study"></a>
During the ablation study, we used the original training code (with the changes made to allow it to run) and only changed the network each experiment. Our ablation study consists of 10 experiments. These experiments mostly existed of removing layers, but for 1 of the experiments we added layers. The train and test images are the same subset consisting of 10 train and 10 test images from the original dataset that we talked about in the Method section.

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

One discussion point we wanted to address outside of the experiments we ran is the metrics used to determine image quality. SSIM is a measure of structural similarity on the grayscale of the images. This means that PSNR is the only value judging the colours of the images. However, it seems like a bad metric. The PSNR mentioned in the paper for their default run on Sony is 28.88. We found across all of our experiments that the PSNR of seemingly terrible results was comparable or even better than the PSNR mentioned as the default in the paper. An example is the amplification being hardcoded to 1 experiment. Here the PSNR was 28.75, which is comparable to the value in the paper. Even though, as shown earlier, the colour quality of those output images should have scored really low.

This discrepency could have many reasons. This could, for example, be due to the fact that they have more images in their test and train set and that this would make the overall average lower. However, if the average of output images was equal to the images in the amplification = 1 set we would argue that is not a very good result. Another reasoning is that the average is overall sensitive to outliers, and maybe there were just some outliers in their output. On one hand the outliers would have been interesting to know about, but on the other hand possibly using the mean of the PSNR and SSIM would have been a better measure to combat this issue. Finally, the PSNR could just also be a bad metric to determine image quality. Since it is then the only judge of the colour quality of an image, a different metric should have been chosen. 

Whatever caused this oddity, it cannot be determined since it was not shared how the SSIM and PSNR were determined. This might stand to reason since these are standardised values. However, it would have been still been helpful in determining what caused this seemingly odd scoring.

## Conclusions and Final remarks <a name="conclusions-and-final-remarks"></a>
When creating new datasets to use on the network, the environment for creating datasets needs to be strictly defined. It needs to be paid special attention to that the camera does not move in between taking a long and short image of the scene, the ratio of the exposure time needs to be a difference of at least 100 for the network to produce viable results, and only then can the ISO be varied. Furthermore, the creation of the datasets showed that the network is indeed not generalisable to other sensors once trained. However, how different bit depths and device qualities play into this could not be determined due to challenges while making the datasets. Finally, when choosing the device it needs to be tested if the Bayer filter and RAW output is easily translatable to the network, since this is often not very clearly defined between brands and not always the same for the network.

When testing the different hyperparameters and doing the ablation study the overall conclusion is that the hyperparameters and network were very well tuned. Although it also showed that the network is not very robust, since any change degraded the output quality.

Overall the paper presented its work very well. Some details, such as to make sure the exposure times varied enough, could have been highlighted a little more. It would have also been good to discuss how the PSNR and SSIM values were achieved, so that we could make sure we did it the same way rather than having to speculate about the perceived oddities. However, the network itself performed very well, and is definitely a good tool to enhance extremely low light images.

## Authors <a name="authors"></a>
#### Group 5
Remco Huijsen (5650844) r.huijsen@student.tudelft.nl: New data, Hyperparams check<br>
Floor Joosen (4814495) f.e.joosen@student.tudelft.nl: New data, Hyperparams check<br>
Marije Tromp (4933656) m.r.tromp@student.tudelft.nl: New data, Ablation study<br>

#### Data
The images used can be found here: https://drive.google.com/drive/folders/1-WvhaLVh7EzWN6PMxix_heBFjCvESS6h?usp=sharing.
If the files are no longer available, please contact one of us at the e-mail addresses above. 

## References <a name="references"></a>
Canon (n.d.). __Canon EOS 100D__. Retrieved April 27, 2023, from https://www.canon.nl/for_home/product_finder/cameras/digital_slr/eos_100d/ 
Canon (n.d.). __Canon EOS 90D-camera__. Retrieved April 27, 2023, from https://www.canon.nl/cameras/eos-90d/specifications/
Chen, C., Chen, Q., Xu, J., & Kotlun, V. (2018). Learning to See in the Dark. __arXiv__, 1805.01934, https://doi.org/10.48550/arXiv.1805.01934
OnePlus (n.d.). __OnePlus 7__. Retrieved April 27, 2023 from https://www.oneplus.com/nl/7/specs
OnePlus (n.d.). __OnePlus Nord 2 5G__. Retrieved April 27, 2023 from https://www.oneplus.com/nl/nord-2-5g
