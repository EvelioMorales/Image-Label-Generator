# AWS Image Label Generator with Amazon Rekognition and S3

## Project Overview

This project uses Amazon Rekognition to analyze images stored in an Amazon S3 bucket and automatically generate descriptive labels with confidence scores.

The Python script connects to AWS using `boto3`, sends an image from S3 to Amazon Rekognition, retrieves detected labels, and displays the image with bounding boxes around detected objects.

This project demonstrates how cloud-based machine learning services can be integrated into Python applications for image analysis.

---

## Architecture Diagram

![Architecture Diagram](https://github.com/EvelioMorales/Image-Label-Generator/blob/main/Images/ImageLableGen_AWSRekognition.png)

---

## Real-World Use Case

This project simulates how businesses can use cloud-based image recognition to automatically classify images, detect objects, organize media files, or support content moderation workflows.

Examples include:

- Labeling uploaded product images
- Organizing image libraries
- Detecting objects in security images
- Supporting inventory or asset tracking
- Building AI-powered media applications
- Creating automated image classification systems

---

## Technologies Used

- Amazon Web Services
- Amazon S3
- Amazon Rekognition
- AWS CLI
- Python
- boto3
- matplotlib
- Pillow
- VS Code

---

## AWS Services Used

### Amazon S3

Amazon S3 is used to store the images that will be analyzed by Amazon Rekognition.

### Amazon Rekognition

Amazon Rekognition is used to analyze images and generate labels with confidence scores.

### AWS CLI

The AWS CLI is used to configure credentials and interact with AWS services from the command line.

---

## Skills Demonstrated

- Creating and configuring an Amazon S3 bucket
- Uploading images to Amazon S3
- Using Amazon Rekognition for image analysis
- Writing Python scripts with `boto3`
- Reading files from S3 using Python
- Visualizing image labels with `matplotlib`
- Drawing bounding boxes around detected objects
- Configuring AWS CLI credentials
- Applying IAM least-privilege concepts
- Troubleshooting AWS permission and credential issues
- Documenting a cloud project for GitHub

---

## Architecture Summary

```text
Local Python Script
        |
        | boto3 SDK
        |
Amazon S3 Bucket
        |
        | Image Object
        |
Amazon Rekognition
        |
        | DetectLabels API
        |
Label Results + Confidence Scores
        |
        | matplotlib + Pillow
        |
Image Display with Bounding Boxes
````

---

## Project Workflow

1. Create an Amazon S3 bucket.
2. Upload images to the S3 bucket.
3. Configure AWS CLI credentials.
4. Install required Python libraries.
5. Write a Python script using `boto3`.
6. Use Amazon Rekognition to detect labels.
7. Display detected labels and confidence scores.
8. Draw bounding boxes around detected objects.
9. Validate and troubleshoot the results.

---

## Repository Structure

```text
Image-Label-Generator/
│
├── Images/
│   ├── ImageLableGen_AWSRekognition.png
│   ├── S3bucket.png
│   ├── NameBucket.png
│   ├── uploaded-image.png
│   └── generated-labels.png
│
├── image_label_generator.py
├── requirements.txt
└── README.md
```

---

## Prerequisites

Before running this project, you need:

* An AWS account
* AWS CLI installed
* Python 3 installed
* VS Code or another code editor
* An S3 bucket with uploaded images
* IAM permissions for Amazon S3 and Amazon Rekognition

---

## Required Python Libraries

Install the required Python libraries:

```bash
pip install boto3 matplotlib pillow
```

Or create a `requirements.txt` file:

```text
boto3
matplotlib
pillow
```

Then install from the file:

```bash
pip install -r requirements.txt
```

---

## AWS CLI Configuration

Before running the Python script, configure AWS CLI credentials:

```bash
aws configure
```

You will be prompted to enter:

```text
AWS Access Key ID
AWS Secret Access Key
Default region name
Default output format
```

Example:

```text
Default region name: us-east-1
Default output format: json
```

> Security Note: Do not hardcode AWS access keys inside your Python script. Use AWS CLI profiles, environment variables, or IAM roles.

---

## Required IAM Permissions

The AWS user or role running this script needs permission to:

* Read objects from the S3 bucket
* Call Amazon Rekognition `DetectLabels`

Example IAM policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowRekognitionDetectLabels",
      "Effect": "Allow",
      "Action": [
        "rekognition:DetectLabels"
      ],
      "Resource": "*"
    },
    {
      "Sid": "AllowS3ReadAccessToImages",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::your-bucket-name/*"
    }
  ]
}
```

Replace:

```text
your-bucket-name
```

with the name of your actual S3 bucket.

---

## 1. Create an Amazon S3 Bucket

1. Log in to the AWS Management Console.
2. Search for **S3**.
3. Open the Amazon S3 service.
4. Click **Create bucket**.
5. Enter a globally unique bucket name.
6. Select an AWS region.
7. Leave the default settings for this lab.
8. Click **Create bucket**.

An S3 bucket acts as a cloud storage container where image files can be stored and accessed by AWS services.

![S3 Bucket](https://github.com/EvelioMorales/Image-Label-Generator/blob/main/Images/S3bucket.png)

---

## 2. Name the S3 Bucket

Choose a unique bucket name.

S3 bucket names must be globally unique across all AWS accounts and regions.

![Name Bucket](https://github.com/EvelioMorales/Image-Label-Generator/blob/main/Images/NameBucket.png)

---

## 3. Upload Images to the S3 Bucket

1. Open the S3 bucket.
2. Click **Upload**.
3. Select the image files from your local computer.
4. Click **Upload**.
5. Confirm that the image appears in the bucket.

The uploaded image will be analyzed by Amazon Rekognition.

![Uploaded Image](https://github.com/EvelioMorales/Image-Label-Generator/blob/main/Images/uploaded%20image.png)

---

## 4. Create the Python File

Create a Python file named:

```text
image_label_generator.py
```

This file will contain the code used to connect to AWS, analyze the image, and display the results.

---

## 5. Python Code

```python
import boto3
import matplotlib.pyplot as plt
import matplotlib.patches as patches
from PIL import Image
from io import BytesIO
from botocore.exceptions import ClientError


AWS_REGION = "us-east-1"
BUCKET_NAME = "labelgen1"
IMAGE_NAME = "animals.jfif"
MAX_LABELS = 10
MIN_CONFIDENCE = 70


def detect_labels(photo, bucket):
    """
    Detect labels in an image stored in Amazon S3 using Amazon Rekognition.
    """

    rekognition_client = boto3.client("rekognition", region_name=AWS_REGION)
    s3_resource = boto3.resource("s3", region_name=AWS_REGION)

    try:
        response = rekognition_client.detect_labels(
            Image={
                "S3Object": {
                    "Bucket": bucket,
                    "Name": photo
                }
            },
            MaxLabels=MAX_LABELS,
            MinConfidence=MIN_CONFIDENCE
        )

        print(f"\nDetected labels for: {photo}\n")

        for label in response["Labels"]:
            print(f"Label: {label['Name']}")
            print(f"Confidence: {label['Confidence']:.2f}%")
            print("-" * 30)

        image_object = s3_resource.Object(bucket, photo)
        image_data = image_object.get()["Body"].read()
        image = Image.open(BytesIO(image_data))

        display_image_with_bounding_boxes(image, response["Labels"])

        return len(response["Labels"])

    except ClientError as error:
        print(f"AWS ClientError: {error}")
        return 0

    except Exception as error:
        print(f"Unexpected error: {error}")
        return 0


def display_image_with_bounding_boxes(image, labels):
    """
    Display the image and draw bounding boxes around detected object instances.
    """

    plt.imshow(image)
    axis = plt.gca()

    for label in labels:
        for instance in label.get("Instances", []):
            bounding_box = instance["BoundingBox"]

            left = bounding_box["Left"] * image.width
            top = bounding_box["Top"] * image.height
            width = bounding_box["Width"] * image.width
            height = bounding_box["Height"] * image.height

            rectangle = patches.Rectangle(
                (left, top),
                width,
                height,
                linewidth=2,
                edgecolor="red",
                facecolor="none"
            )

            axis.add_patch(rectangle)

            label_text = f"{label['Name']} ({label['Confidence']:.2f}%)"

            plt.text(
                left,
                top - 5,
                label_text,
                color="red",
                fontsize=8,
                bbox={"facecolor": "white", "alpha": 0.7}
            )

    plt.axis("off")
    plt.show()


def main():
    label_count = detect_labels(IMAGE_NAME, BUCKET_NAME)
    print(f"\nTotal labels detected: {label_count}")


if __name__ == "__main__":
    main()
```

---

## 6. Update Script Variables

Before running the script, update these values:

```python
AWS_REGION = "us-east-1"
BUCKET_NAME = "labelgen1"
IMAGE_NAME = "animals.jfif"
```

Replace them with:

* Your AWS region
* Your S3 bucket name
* Your uploaded image file name

Example:

```python
AWS_REGION = "us-east-1"
BUCKET_NAME = "my-image-label-bucket"
IMAGE_NAME = "dog-photo.jpg"
```

---

## 7. Run the Python Script

Open the terminal in the same directory as the Python file and run:

```bash
python image_label_generator.py
```

If the script runs successfully, the output will show detected labels and confidence scores.

Example output:

```text
Detected labels for: animals.jfif

Label: Animal
Confidence: 99.25%
------------------------------
Label: Dog
Confidence: 98.14%
------------------------------
Label: Pet
Confidence: 95.73%
------------------------------

Total labels detected: 3
```

A pop-up window will also display the uploaded image with bounding boxes around detected objects.

![Generated Labels](https://github.com/EvelioMorales/Image-Label-Generator/blob/main/Images/generated%20labels.png)

---

## Validation

The project was validated using the following checks:

| Test                   | Expected Result                       | Status |
| ---------------------- | ------------------------------------- | ------ |
| Create S3 bucket       | Bucket is created successfully        | Passed |
| Upload image to S3     | Image appears in bucket               | Passed |
| Configure AWS CLI      | AWS credentials are available locally | Passed |
| Run Python script      | Rekognition returns labels            | Passed |
| Display image          | Image opens with bounding boxes       | Passed |
| Show confidence scores | Labels include confidence percentages | Passed |

---

## Troubleshooting

### AccessDeniedException

**Cause:**

The IAM user or role does not have permission to call Amazon Rekognition or read from S3.

**Fix:**

Confirm the user has the following permissions:

```text
rekognition:DetectLabels
s3:GetObject
```

---

### NoCredentialsError

**Cause:**

AWS credentials are not configured locally.

**Fix:**

Run:

```bash
aws configure
```

Then confirm your credentials are active:

```bash
aws sts get-caller-identity
```

---

### Image Not Found

**Cause:**

The image name in the Python script does not match the object name in S3.

**Fix:**

Check the exact file name in the S3 bucket and update:

```python
IMAGE_NAME = "your-image-name.jpg"
```

---

### Region Error

**Cause:**

The AWS region in the script does not match the region where the services are being used.

**Fix:**

Update:

```python
AWS_REGION = "us-east-1"
```

to match your AWS region.

---

### Bounding Boxes Do Not Appear

**Cause:**

Some labels returned by Amazon Rekognition do not include object instances or bounding boxes.

**Fix:**

This is expected behavior. Rekognition may detect general scene labels without returning object coordinates.

---

## Security Considerations

* Do not hardcode AWS access keys in the Python script.
* Use IAM least-privilege permissions.
* Keep S3 buckets private unless public access is required.
* Store credentials securely using AWS CLI profiles or IAM roles.
* Avoid uploading sensitive or personal images to public buckets.
* Delete test images when the project is complete.
* Rotate access keys regularly if using IAM user credentials.
* Use IAM roles when running this application on AWS compute services.

---

## Cost Management

Amazon Rekognition and Amazon S3 may create charges depending on usage.

To reduce unnecessary costs:

1. Delete test images after the project.
2. Delete the S3 bucket if it is no longer needed.
3. Monitor AWS Billing.
4. Use AWS Budgets to set spending alerts.
5. Avoid running the script repeatedly on large image sets unless needed.

---

## Cleanup

To clean up the project:

1. Go to the Amazon S3 console.
2. Open the bucket used for this project.
3. Delete the uploaded images.
4. Delete the S3 bucket if it is no longer needed.
5. Remove unused IAM permissions or access keys.

Optional AWS CLI cleanup:

```bash
aws s3 rm s3://your-bucket-name --recursive
aws s3 rb s3://your-bucket-name
```

---

## Lessons Learned

Through this project, I learned how to:

* Create and use an Amazon S3 bucket
* Upload images to AWS cloud storage
* Use Amazon Rekognition to detect image labels
* Connect Python applications to AWS using `boto3`
* Display image analysis results with bounding boxes
* Configure AWS CLI credentials
* Apply IAM permissions for AWS service access
* Troubleshoot common AWS and Python errors

This project helped strengthen my understanding of AWS AI services, cloud storage, Python automation, and image analysis workflows.

---

## Future Improvements

This project can be upgraded by adding:

* Support for multiple image uploads
* CSV export for detected labels
* DynamoDB storage for label results
* A web interface using Flask or Streamlit
* Automatic Rekognition analysis when an image is uploaded to S3
* AWS Lambda for serverless image processing
* Amazon API Gateway for an image analysis API
* Amazon CloudWatch for logging and monitoring
* Face detection using Amazon Rekognition
* Unsafe content detection using Rekognition moderation labels
* Terraform deployment for AWS infrastructure
* CI/CD deployment using GitHub Actions

---

## Recommended Serverless Architecture Upgrade

A stronger version of this project could use an event-driven AWS architecture:

```text
User Uploads Image
        |
        v
Amazon S3 Bucket
        |
        v
AWS Lambda Trigger
        |
        v
Amazon Rekognition
        |
        v
DynamoDB Stores Labels
        |
        v
CloudWatch Logs
```

This would turn the project from a local Python script into a production-style serverless image analysis pipeline.

---

## Conclusion

This project demonstrates how Amazon Rekognition and Amazon S3 can be used together to analyze images and generate labels automatically.

By combining AWS cloud storage, AI image recognition, Python scripting, and data visualization, this project shows a practical example of cloud-based machine learning integration.


