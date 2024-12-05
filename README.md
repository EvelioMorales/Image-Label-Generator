# Image Label Generator

In this project, I will be building an image labels generator, using Amazon Rekognition. Once built, it will be able to recognize and label images. For example, if I have a photo of a cat, Amazon Recognition will be able to identify what it is, and label the image as a cat.

![Diagram]()

# Steps to be preformed

* Creating an Amazon S3 Bucket
* Uploading images to the S3 Bucket
* Importing libraries
* Adding detect_labels function
* Adding main function
* Running your python file

# Services 

* Amazon S3: For storing the images in the process of generating labels.
* Amazon Rekognition: To analyse images and generate image labels.
* AWS CLI: Interacting with AWS services through command line interface(CLI).


# Create an Amazon S3 Bucket

1. Log in to your AWS Management Console.
2. Navigate to the Amazon S3 service from the search bar. An S3 bucket is like a virtual storage box in the cloud where you can keep your files safe and easily accessible with permissions.

![S3bucket]()

3. Click on __Create Bucket__.
4. I will choose a unique name for the bucket.

![NameBucket]()

5. I will leave the default settings for the rest of the options and click __‘Create Bucket’__.
6. I will use this bucket to store the images on which labels are to be generated. Know I'll upload some images in the S3 bucket.

# Upload Images to S3 Bucket

1. Once the bucket is created, I will navigate to the bucket.
2. Click on the __‘Upload’__ button and select the images I want to analyse from my system.
3. Click on Upload. My image has now been uploaded in the S3 bucket.

![UPLOADED IMAGE]()


# Importing Libraries

1. I will open VSCode and create a .py file for my code.
2. I will open terminal and install libraries needed for this project.

  _pip install boto3_
  _pip install matplotlib_

3. Let's import the necessary libraries. We need:

  a. __boto3__ for interacting with AWS services.
  b. __matplotlib__ for visualization.
  c. __PIL (Python Imaging Library)__ for handling image data.
  d. __BytesIO__ from the io module to work with image data.

4. I will add the following code:

```python
import boto3
import matplotlib.pyplot as plt
import matplotlib.patches as patches
from PIL import Image
from io import BytesIO
```

# detect_labels function

Now, I'll define a function called __detect_labels__. This function takes a photo and bucket name as input parameters. Within the function:

  * We create a __Rekognition__ client using boto3.
  * We use the __detect_labels__ method of the Rekognition client to detect labels in the given photo.
  * We print the detected labels along with their confidence levels.
  * We load the image from the __S3 bucket__ using boto3 and PIL.
  * We use __matplotlib__ to display the image and draw bounding boxes around the detected objects.

```python
def detect_labels(photo, bucket):
    client = boto3.client('rekognition')

    response = client.detect_labels(
        Image={'S3Object': {'Bucket': bucket, 'Name': photo}},
        MaxLabels=10)

    print('Detected labels for ' + photo) 
    print()   

    # Print label information
    for label in response['Labels']:
        print("Label:", label['Name'])
        print("Confidence:", label['Confidence'])
        print()

    # Load the image from S3
    s3 = boto3.resource('s3')
    obj = s3.Object(bucket, photo)
    img_data = obj.get()['Body'].read()
    img = Image.open(BytesIO(img_data))

    # Display the image
    plt.imshow(img)
    ax = plt.gca()

    # Plot bounding boxes
    for label in response['Labels']:
        for instance in label.get('Instances', []):
            bbox = instance['BoundingBox']
            left = bbox['Left'] * img.width
            top = bbox['Top'] * img.height
            width = bbox['Width'] * img.width
            height = bbox['Height'] * img.height

            rect = patches.Rectangle((left, top), width, height, linewidth=1, edgecolor='r', facecolor='none')
            ax.add_patch(rect)

            label_text = label['Name'] + ' (' + str(round(label['Confidence'], 2)) + '%)'
            plt.text(left, top - 2, label_text, color='r', fontsize=8, bbox=dict(facecolor='white', alpha=0.7))

    plt.show()

    return len(response['Labels'])
```

# Main Function 

Next, I'll write a main function to test the detect_labels function. I'll specify a sample photo and bucket name, then call the detect_labels function with these parameters.

```python
def main():
    photo = 'animals.jfif'
    bucket = 'labelgen1'
    label_count = detect_labels(photo, bucket)
    print("Labels detected:", label_count)

if __name__ == "__main__":
    main()
```

# Running the Python file

1. Open the terminal in the directory where your Python file is present and run the command:

    _python name_of_python_file.py_  (I will replace the _name_of_python_file_ with the name of the file I have saved)

3. Onece the command is entered an out will show of the labels detected and their confidence levels and a pop-up screen will display the image that was uploaded to the S3 bucket with the bounding bnoxes present on the generated lables.

![generated labels]()
