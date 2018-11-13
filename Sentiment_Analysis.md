# DeepLens-workshops

In this workshop you will learn how to build a sentiment analysis project for your DeepLens.

In this project you will learn to build a deep learning model to identify and analyze the sentiments of your audience

## In this workshop you will learn the following:

1. How to build and train a face detection model in SageMaker
2. Modify the DeepLens inference lambda function to upload cropped faces to S3
3. Deploy the inference lambda function and face detection model to DeepLens
4. Create a lambda function to trigger Rekognition to identify emotions
5. Create a DynamoDB table to store the recognized emotions
6. Analyze using CloudWatch

![image](https://user-images.githubusercontent.com/36491325/48433553-5df69180-e745-11e8-9961-ce9837b96485.png)

The workshop consists of 3 steps:


# Hands-on Step 1: Build and train a face detection model in SageMaker

In this step, you will build and train a face detection model. Follow instructions here: [SageMaker lab](https://github.com/mahendrabairagi/reInventWorkshop1/tree/master/SageMaker%20lab)

# Hands-on Step 2: Build a project to detect faces and send the cropped faces to S3 bucket

#### Recall Bucket:

Recall the bucket name you created in the SageMaker lab, we'll be using it shortly.

#### Create Inference lambda function:

A DeepLens **Project** consists of two things:
* A model artifact: This is the model that is used for inference.
* A Lambda function: This is the script that runs inference on the device.

Before we deploy a project to DeepLens, we need to create a custom lambda function that will use the face-detection model on the device to detect faces and push crops to S3.

#### Create Inference lambda function:

Go to [AWS Management console](https://console.aws.amazon.com/console/home?region=us-east-1) and search for Lambda

Click **Create function**

Choose **Blueprints**

In the search bar, type **greengrass-hello-world** and hit **Enter**.

Choose the python blueprint and click **Configure**/

**Name** the function: DeepLens-sentiment-your-name

**Role**: Choose an existing role

**Existing Role**: AWSDeepLensLambdaRole

Click **Create Function**.

Scroll down to **Function Code**. Once generated, replace the default script with the [inference script](https://raw.githubusercontent.com/mahendrabairagi/reInventWorkshop1/master/Inference%20Lambda/inference-lambda.py) here by copying and pasting.

You can select the inference script, by selecting Raw in the Github page and choosing the script using ctrl+A/ cmd+A . Copy the script and paste it into the lambda function (make sure you delete the default code).

**Note**: In the script, you will have to provide the name for your S3 bucket. Insert your bucket name in the code below

![code bucket](https://user-images.githubusercontent.com/11222214/38719807-b46169fa-3ea8-11e8-8ff2-69af5455ede7.jpg)

Click Save

![Alt text](/screenshots/deeplens_lambda_0.png)

You'll also click "Actions" and then "Publish new version".

![Alt text](/screenshots/deeplens_lambda_1.png)

Then, enter a brief description and click "Publish."

![Alt text](/screenshots/deeplens_lambda_2.png)


### Create & Deploy DeepLens Project

With the lambda created, we can now make a project using it and the built-in face detection model.

From the DeepLens homepage dashboard, select "Projects" from the left side-bar:

![Alt text](/screenshots/deeplens_project_0.png)

Then select "Create new project"

![Alt text](/screenshots/deeplens_project_1.png)

Next, select "Create a new blank project" then click "Next".

![Alt text](/screenshots/deeplens_project_2.png)

Now, name your deeplens project.

![Alt text](/screenshots/deeplens_project_3.png)

Next, select "Add model". From the pop-up window, select "deeplens-face-detection" then click "Add model".

![Alt text](/screenshots/deeplens_project_4.png)

Next, select "Add function". from the pop-up window, select your deeplens lambda function and click "Add function".

![Alt text](/screenshots/deeplens_project_5.png)

Finally, click "Create".

![Alt text](/screenshots/deeplens_project_6.png)

Now that the project has been created, you will select your project from the project dashboard and click "Deploy to device".

![Alt text](/screenshots/deeplens_project_7.png)

Select the device you're deploying too, then click "Review" (your screen will look different here).

![Alt text](/screenshots/deeplens_project_8.png)

Finally, click "Deploy" on the next screen to begin project deployment.

![Alt text](/screenshots/deeplens_project_9.png)

You should now start to see deployment status. Once the project has been deployed, your deeplens will now start processing frames and running face-detection locally. When faces are detected, it will push to your S3 bucket. Everything else in the pipeline remains the same, so return to your dashboard to see the new results coming in!

**Note**: If your model download progress hangs at a blank state (Not 0%, but **blank**) then you may need to reset greengrass on DeepLens. To do this, log onto the DeepLens device, open up a terminal, and type the following command:
`sudo systemctl restart greengrassd.service --no-block`. After a couple minutes, you model should start to download.


**Confirmation/ verification**

You will find your cropped faces uplaod to your S3 bucket.



# Hands-on Step 3: Identify emotions

**Step 3.1 - Create DynamoDB table**

Go to [AWS Management console](https://console.aws.amazon.com/console/home?region=us-east-1) and search for Dynamo

Click on Create Table.

Name of the table: recognize-emotions-your-name
Primary key: s3key

Click on Create. This will create a table in your DynamoDB.


**Step 3.2 - Create a lambda function that runs in the cloud**

The inference lambda function that you deployed earlier will upload the cropped faces to your S3. On S3 upload, this new lambda function gets triggered and runs the Rekognize Emotions API by integrating with Amazon Rekognition. 

Go to [AWS Management console](https://console.aws.amazon.com/console/home?region=us-east-1) and search for Lambda

Click 'Create function'

Choose 'Author from scratch'

Name the function: recognize-emotion-your-name.  
Runtime: Choose Python 2.7
Role: Choose an existing role
Existing role: rekognizeEmotions

Choose Create function

Replace the default script with the script in [recognize-emotions.py](https://github.com/mahendrabairagi/reInventWorkshop1/blob/master/Integrate%20with%20Rekognition/rekognize-emotions.py). You can select the script by selecting Raw in the Github page and choosing the script using ctrl+A/ cmd+A . Copy the script and paste it into the lambda function (make sure you delete the default code).

Make sure you enter the table name you created earlier in the section highlighted below:

![dynamodb](https://user-images.githubusercontent.com/11222214/38838790-b8b72116-418c-11e8-9a77-9444fc03bba6.JPG)


Next, we need to add the event that triggers this lambda function. This will be an “S3:ObjectCreated” event that happens every time a face is uploaded to the face S3 bucket. Add S3 trigger from designer section on the left. 

Configure with the following:

Bucket name: face-detection-your-name (you created this bucket earlier)
Event type- Object Created
Prefix- faces/
Filter- .jpg
Enable trigger- ON (keep the checkbox on)

Save the lambda function

Under 'Actions' tab choose **Publish**

**Step 3.3 - View the emotions on a dashboard**

Go to [AWS Management console](https://console.aws.amazon.com/console/home?region=us-east-1) and search for Cloudwatch

Create a dashboard called “sentiment-dashboard-your-name”

Choose Line in the widget

Under Custom Namespaces, select “string”, “Metrics with no dimensions”, and then select all metrics.

Next, set “Auto-refresh” to the smallest interval possible (1h), and change the “Period” to whatever works best for you (1 second or 5 seconds)

NOTE: These metrics will only appear once they have been sent to Cloudwatch via the Rekognition Lambda. It may take some time for them to appear after your model is deployed and running locally. If they do not appear, then there is a problem somewhere in the pipeline.


### With this we have come to the end of the session. As part of building this project, you learnt the following:

1.	How to build and train a face detection model in SageMaker
2.	Modify the DeepLens inference lambda function to upload cropped faces to S3
3.	Deploy the inference lambda function and face detection model to DeepLens
4.	Create a lambda function to trigger Rekognition to identify emotions
5.	Create a DynamoDB table to store the recognized emotions
6.	Analyze using CloudWatch
