# TumorOrganoidSegOCT
Development of an algorithm able to manage OCT images, which have a low signal-to-noise ratio and a high noise concentration, and which is able to segment the organoids present within the volumes of OCT images acquired from artificial cultures in vitro

# **NEURAL NETWORK ARCHITECTURE**

The network architecture used is a **UNet**, a 2D segmentation network based on an encoder-decoder structure. This network was implemented using the **PyTorch framework** and trained using an **Adam optimizer** with a **DiceCELoss loss function**. A type of normalization called batch normalization was used to ensure model stability during the training process. This implies that the input data is normalized in order to have a more uniform and stable distribution.
The loss function used is the Dice Cross-Entropy Loss, which combines the Dice Loss with the Cross-Entropy Loss. A sigmoid activation function was applied to the model output to produce a probability for each pixel.
The Adam optimizer was used to optimize the model with a learning rate of 1e-4. The smoothing coefficient L2 has been set equal to 1e-5, which means that a smoothing term has been included in the loss function to prevent overfitting.
Furthermore, a seed equal to 46 has been set, i.e. the initial conditions have been set in order to make the operations repeatable and produce consistent results in subsequent executions.

# **METRICS FOR EVALUATION OF RESULTS**

In this work, three evaluation metrics were used to quantify the performance of the segmentation algorithm: **dice**, **recall** and **precision**.
The Dice coefficient was used to quantify the similarity between the segmentation performed by the algorithm and the ground truth. The Dice of a single 3D volume was calculated as shown in Figure 1.

<img width="370" alt="Formula Dice" src="https://user-images.githubusercontent.com/119749266/223556789-f7ddaf26-1de1-43af-8600-7235338a5edb.png">

*Figure 1: Formula for the calculation of Dice 3D, where MM is the set of voxels belonging to the manual mask and AM is the set of voxels belonging to the automatic mask*




Recall and precision were used to understand if the algorithm tends to over- or under-segment the structures of the 3D image, respectively.

# **PRE-PROCESSING**

During the analysis phase it was observed that the images present not only high "salt and pepper" noise, but also the presence of artifacts on some of the image slices. Most of the artifacts consist of lines running throughout the image and have the same intensity as the pixels of the tumors. These artifacts can be caused by various sources, such as image capture problems

# **ARTIFACT MANAGEMENT**

In addition to point-type artefacts, which appear as pixels with an intensity similar to that of the tumor, linear artefacts were also detected during the analysis of the images, mainly along the horizontal dimension. These artifacts share the same intensity as the tumor and can be an obstacle in the tumor identification phase during model training, as they can be incorrectly recognized as part of the tumor tissue. In light of this problem, it was decided to create a function, called delartifact(), able to handle the linear artifacts present in the images.
The **delartifact()** function processes the previously filtered original image and a binary mask generated from the original image using the **img_to_mask()** function, which uses the global thresholding technique. The threshold was chosen equal to 1700, since through the 3D Slicer software it was observed that all the artifacts have an intensity similar to or greater than this value.

For each 2D slice of the original image the function starts by stretching the binary mask, thus filling any holes within the white regions. This operation is performed with a 3x3 kernel to join the identified elements (therefore the artifacts), which are often detached from each other by very few pixels. Subsequently, the mask is eroded with the same 3x3 kernel, overlaid on the original image and passed as input to the regionprops() function of the skimage.measure module.
The function traverses each region found in the mask and removes linear artifacts. Artifacts are identified based on their shape. In particular, elements with a length greater than a certain predetermined value and a width smaller than a certain predetermined value are considered linear artifacts. To remove the artifact from the image the function assigns to the pixel the minimum intensity of the original image. In figure 2 it is possible to observe the qualitative results.

![artefatti](https://user-images.githubusercontent.com/119749266/223556867-9fe9a7cb-7676-4d3f-8319-fe0c4a258a8f.png)

*Figure 2: In figure 2A the original image is represented, in figure 2B we see the binary mask with the red bounding boxes that highlight the elements identified through regionprops, in figure 2C we observe the binary mask with the blue bounding boxes that indicate the artifacts detected, while Figure 2D shows the original image corrected after artifacts are removed.*

# **PRE-PROCESSING - CONTRAST**

Contrast is a key element for OCT images, as it allows to increase the intensity of the tumor pixels and to decrease that of the background. This results in sharper and easier to interpret images. To increase the contrast, the AdjustContrast function of MONAI was used, which exploits the formula shown in Figure 3.

Where the variable intensity range is equal to the difference between the maximum value and the minimum value of the image given as input to the function, the variable min represents the minimum value of the pixels of the image itself and gamma is the parameter to be given as input to the function.
To improve the contrast of the images, a gamma value of 1.5 was set together with a median filter. In figures 3 and 4 it is possible to observe the qualitative results.

![senzaContrasto](https://user-images.githubusercontent.com/119749266/223557690-83766d64-bede-4d9e-9bda-34293697d83d.png)

*Figure 3: Figure 3A shows the original image without any pre-processing. Instead, in Figure 3B it is possible to observe the automatic mask produced by the segmentation model, with the red bounding boxes delimiting the segmented elements. Finally, Figure 3C shows the segmented mask via the manual operator*

![Concontrasto](https://user-images.githubusercontent.com/119749266/223557743-df0a8a5b-c66d-40cd-9fbc-6f8307ff4892.png)

*Figure 4: The figure follows the indexing of Figure 3, with the addition of the application of contrast.*

# **POST-PROCESSING**

In the post-processing phase it was decided to use the **binary_fill_holes** morphological operator of the Scipy library to improve the quality of the segmentation masks. In particular, this operator allows to fill any holes present in the masks, thus improving the continuity of the edges of the tumors.
In addition, the areas, mean intensities, and histogram of all operator-segmented tumors were evaluated using a separate script (**Analisi-tumori.ipynb**). The objective of this review was to compare these characteristics with those obtained from the segmentation model, in order to evaluate metrics capable of distinguishing artifacts (which have the same size and shape as tumors) from tumors and reduce the presence of false positives .
The script in question iterates through the batches of images and masks, superimposes the mask on each image and calculates the properties of the regions (tumors) present, using the regionprops function of the skimage library. For each region identified, the algorithm extracts the area, the average intensity of the pixels and the normalized histogram, which are then saved in a dictionary. Figure 5 shows the qualitative results.


![Post](https://user-images.githubusercontent.com/119749266/223557776-59b8b376-a4eb-4e24-b801-f7582be3c071.png)

*Figure 5: Figure 5A shows the segmentation output to the classifier without the application of the morphological operator. In figure 5B, the same segmentation, but with the application of the binary_fill_holes.*
