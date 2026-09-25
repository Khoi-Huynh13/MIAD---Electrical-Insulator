## Table of Contents

1. [Problem Definition](#1-problem-definition)
2. [Machine Learning Approach](#2-machine-learning-approach)
   - [2.1 Model Choice and Justification](#21-model-choice-and-justification)
   - [2.2 Training Procedure and Hyperparameters](#22-training-procedure-and-hyperparameters)
3. [Results & Discussion](#3-results--discussion)

<br>

## 1. Problem Definition
<p align="justify">
The objective of this project is to develop an unsupervised anomaly detection system for the outdoor maintenance inspection of electrical insulators. 
Electrical insulators support and isolate conductors from supporting structures. As a result, they are essential for maintaining the safety and 
reliable operation of electrical systems. However, electrical insulators can develop defects due to factors such as transportation damage, manufacturing faults, 
and handling errors. These defects may include localized surface damage, such as chips or broken sections, which can compromise the electrical and 
mechanical properties of the insulator. Figure 1a and Figure 1b show examples of good and defective electrical insulators, respectively. 
For the defective example, a red mask has been overlaid to highlight the damaged region.
</p>

<br>

<p align="center">
  <img width="477" height="220" alt="Screenshot 2026-09-25 162043" src="https://github.com/user-attachments/assets/85997456-af1b-445e-a5f4-d9aa764440b7" />
  <br>
  <em>Figure 1a. Good electrical insulators</em>
</p>

<br>

<p align="center">
  <img width="477" height="222" alt="Screenshot 2026-09-25 162038" src="https://github.com/user-attachments/assets/278201f2-5d2f-4636-80d8-06385259347e" />
  <br>
  <em>Figure 1b. Defective electrical insulators</em>
</p>

<br>

## 2. Machine Learning Approach

### 2.1 Model Choice and Justification
<p align="justify">
A Convolutional Autoencoder (CAE) was selected for unsupervised anomaly detection. An autoencoder consists of an encoder, bottleneck, and decoder. 
The encoder progressively transforms the input image into a lower-dimensional representation by extracting increasingly abstract features. 
Conversely, the decoder reconstructs the original image from this representation. Unlike a traditional autoencoder, which commonly represents the 
bottleneck as a vector, a CAE preserves the spatial structure of image data by representing the bottleneck as a feature map. This makes it well-suited for 
image reconstruction.
</p>

<br>

<p align="center">
  <img width="595" height="151" alt="Screenshot 2026-09-19 162916" src="https://github.com/user-attachments/assets/4ef26524-54d6-412d-90ba-737eda77457a" />
  <br>
  <em>Figure 2. CAE architecture</em>
</p>

<p align="justify">
The CAE is trained exclusively on anomaly-free electrical insulator images, allowing it to learn the visual characteristics and 
underlying representation of normal insulators. During inference, normal images are expected to be reconstructed relatively accurately because 
they are consistent with the distribution observed during training. Conversely, when presented with a defective image, the model is expected to 
reconstruct it according to the normal patterns learned during training and therefore fail to reproduce some characteristics of the defect. 
</p>

<p align="justify">
The difference between the input and reconstructed image is measured using Mean Squared Error (MSE) as the reconstruction error. Images producing 
a reconstruction error above a predefined threshold are classified as anomalies. The decision threshold is determined using reconstruction errors from the normal validation set, with the threshold defined as the validation mean 
plus two standard deviations. This ensures that defective test images are not used when establishing the anomaly criterion.
</p>

<br>

<p align="center">
  <img width="642" height="80" alt="image" src="https://github.com/user-attachments/assets/deca2293-6196-4c2e-bd4d-19fb073693ff" />
  <br>
  <em>Figure 3. Training pipeline</em>
</p>

### 2.2 Training Procedure and Hyperparameters
<p align="justify">
The CAE is trained exclusively on anomaly-free images from the training set. Each image is passed through the encoder and decoder to produce 
a reconstruction, and the MSE between the original and reconstructed images is used as the training loss. The loss is backpropagated through the 
network and the model parameters are updated using the optimizer.
</p>

<br>

<p align="center">
  <img width="401" height="397" alt="Screenshot 2026-09-17 131253" src="https://github.com/user-attachments/assets/cc299580-bafa-44fe-b6fe-1ac9949678a7" />
  <br>
  <em>Figure 4. Original vs Reconstructed image per 20 epochs</em>
</p>

<p align="justify">
Following each training epoch, the model is evaluated on the validation set. The validation reconstruction loss is monitored for early 
stopping, which terminates training when the model fails to improve for a predefined number of consecutive epochs. A learning-rate scheduler is 
also used to reduce the learning rate when validation performance reaches a plateau. Doing so allows the model to continue making smaller parameter updates 
when further optimisation becomes more difficult.
</p>

<br>

<p align="center">
  <img width="439" height="342" alt="Screenshot 2026-09-17 130759" src="https://github.com/user-attachments/assets/2d0a3c05-8640-4d52-894f-a6792b4a5c00" />
  <br>
  <em>Figure 5. Loss over time</em>
</p>

<p align="justify">
After training is completed, the reconstruction error is calculated independently for each validation image. The resulting errors are used to 
calculate the mean and standard deviation, which are subsequently used to establish the anomaly detection threshold. The key training hyperparameters 
used for the CAE are summarised below.
</p>

<div align="center">

| Hyperparameter | Value |
|:---:|:---:|
| Batch Size | 32 |
| Device | T4 GPU |
| Optimizer | Adam |
| Initial learning rate | 0.01 |
| Loss function | Mean Squared Error |
| Maximum epochs | 200 |
| Early stopping patience | 20 |
| Learning rate scheduler | ReduceLROnPlateau |
| Learning rate factor | 0.1 |
| Learning rate scheduler patience | 10 |

<em>Table 1. Hyperparameter table</em>
</div>

## 3. Results & Discussion

## 3. Results & Discussion
<p align="justify">
The CAE achieved an overall test accuracy of 0.4972 on the balanced test set, which is approximately equivalent to random-chance performance. 
The model achieved a defective-class recall and a good-class recall of 0.0328 and 0.9616, respectively. This indicates that very few defective insulators 
were successfully detected while demonstrating a strong tendency to classify images as normal.

These poor results suggest that the main limitation was not necessarily the CAE's ability to reconstruct normal images, but rather the ability of the 
global reconstruction error to distinguish normal and defective images. The initial hypothesis was that defective images would produce substantially 
higher reconstruction errors because the model was trained exclusively on normal insulators. However, this assumption did not hold for the defects 
present in the dataset.
</p>

<br>

<p align="center">
  <img width="471" height="166" alt="Screenshot 2026-09-19 173000" src="https://github.com/user-attachments/assets/9b8fef3c-c848-4db4-be3b-0db86c6d9bd8" />
  <br>
  <em>Figure 6a. Model performance on test set</em>
</p>

<br>

<p align="center">
  <img width="437" height="352" alt="Screenshot 2026-09-19 174753" src="https://github.com/user-attachments/assets/cbb9df11-2b1d-483c-a4f5-e498aba0051c" />
  <br>
  <em>Figure 6b. Confusion matrix of test set</em>
</p>

<p align="justify">
A key reason is that the defects are often small and localized, consisting primarily of only minor chips or surface damage. 
Since the reconstruction error is calculated using the global pixel-wise MSE across the entire 512 × 512 image, the contribution of a 
small defective region is diluted by the large number of normal pixels. Consequently, even when a defect is poorly reconstructed, its 
contribution may be insufficient to increase the overall reconstruction error beyond the anomaly threshold.
</p>

<br>

<p align="center">
  <img width="477" height="220" alt="Screenshot 2026-09-25 171741" src="https://github.com/user-attachments/assets/c6f2017b-3ee9-45aa-b0d6-45766bf0c904" />
  <br>
  <em>Figure 7. Difference between original defective image and its reconstruction</em>
</p>

<p align="justify">
This limitation is also evident in the reconstruction-error distributions for the good and defective classes, which exhibit substantial overlap (See Figure 8). 
The overlap indicates that a single global reconstruction-error threshold cannot reliably distinguish between the two classes. Overall, the results 
demonstrate that global image-level reconstruction error is insufficiently sensitive to the small, localised defects present in this dataset. 
Future improvements should therefore focus on methods that preserve and evaluate local spatial information, rather than relying solely on a single
reconstruction error calculated across the entire image.
</p>

<br>

<p align="center">
  <img width="480" height="330" alt="Screenshot 2026-09-19 175650" src="https://github.com/user-attachments/assets/2a0ba42c-88f9-47f9-8b4e-219a6b48a6fb" />
  <br>
  <em>Figure 8. Distribution of reconstruction error between 2 classes</em>
</p>
