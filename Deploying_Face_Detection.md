## Deploy Face Recognition project

![architecture](https://user-images.githubusercontent.com/36491325/48433430-107a2480-e745-11e8-9069-2f6a19ae5dce.png)

### Step 0 - Login to AWS DeepLens Device & AWS Account

In this workshop, you have an AWS DeepLens device in front of you connected to a monitor, keyboard, and mouse. AWS DeepLens runs an Ubuntu OS. Login to the device with the password **Aws2017!**.

We have already pre-registered your devices to workshop accounts. You can find the information for your account on the card in front of you taped to your monitor.

Open a Firefox browser on the left panel. Once Firefox is open, type **console.aws.amazon.com** into the url bar.
(**Note**: If the login page says "Root user sign in" and there's already an email showing, select **Sign in to a different account** and then type in your **AWS Account** number on your card.)

Once your login page shows three fields, please enter the following:
* Account ID or alias: the **AWS Account** number on your card
* IAM user name: the **User** name on your card
* Password: **Aws2017!**

Next, make sure you're in N. Virginia region, and navigate to the DeepLens Dashboard.

### Step 1- Create Project

The console should open on the Projects screen, select Create new project on the top right (if you don’t see the project list view, click on the hamburger menu on the left and select Projects)

![create project](https://user-images.githubusercontent.com/11222214/38657905-82207e44-3dd7-11e8-83ef-52049e229e33.JPG)

Choose, Use a **project template** as the Project type, and select **Face Detection** from the project templates list.

![project template](https://user-images.githubusercontent.com/11222214/38657922-958edd7c-3dd7-11e8-830b-ec129d9363e6.JPG)

Scroll down the screen and select **Next**

![project template-next](https://user-images.githubusercontent.com/11222214/38657930-a3f6c1a4-3dd7-11e8-96a9-3f3cebb1712e.JPG)

Change the Project name as Face-detection-your-name

![face detection your name](https://user-images.githubusercontent.com/11222214/38657948-b8cc049a-3dd7-11e8-948f-1d32948408d1.JPG)

Scroll down the screen and select **Create**


![click create](https://user-images.githubusercontent.com/11222214/38657969-d573db7c-3dd7-11e8-9f45-fc6d1eb25a4b.JPG)


### Step 2- Deploy to device
In this step, you will deploy the Face detection project to your AWS DeepLens device.

Select the project you just created from the list by choosing the radio button.
**Note**: You may see your project in the Projects list, but are not able to deploy it. This means it's still being created, and it may take up to a minute. Refresh until you see a **Creation Time** for your project; now you will be able to deploy it.


Select Deploy to device.


![choose project-edited-just picture](https://user-images.githubusercontent.com/11222214/38657988-eb9d98b6-3dd7-11e8-8c94-7273fcfa6e1b.jpg)

On the Target device screen, choose your device (your device name is on your card as **Device**) from the list by clicking a radio button, and select **Review.**.

![target device](https://user-images.githubusercontent.com/11222214/38658011-088f81d2-3dd8-11e8-972a-9342b7b3e291.JPG)

Select Deploy.

![review deploy](https://user-images.githubusercontent.com/11222214/38658032-223db2e8-3dd8-11e8-9bdf-04779cd0e0e6.JPG)

On the AWS DeepLens console, you can track the progress of the deployment. It can take a few minutes to transfer a large model file to the device. Once the project is downloaded, you will see a success message displayed and the banner color will change from blue to green.

## View Output

This default projects give inference output in two forms:
* Messages published to an IoT topic in the cloud via MQTT Protocol
* Inference video stream locally on the device

We will look at both.

### IoT 

Once your project has deployed, scroll down your device page to the **Project Output** panel.

![project output](https://user-images.githubusercontent.com/36491325/48432499-7d3fef80-e742-11e8-9740-43711b7df651.png)

Click **Copy** to copy the IoT topic id unique to your device (this is the topic your Project is publishing messages to).

Then click the link to the **AWS IoT Console**. Once there, paste your IoT topic id into the **Subscription topic** field, then click **Subscribe**.

![iot dash](https://user-images.githubusercontent.com/36491325/48432695-1ec74100-e743-11e8-8af6-3268234b7233.png)

You should now start to see the messages being published to your topic from your device.

![iot topic](https://user-images.githubusercontent.com/36491325/48432734-3e5e6980-e743-11e8-83d7-3bfc46f10de6.png)

These messages are the real-time results of our model. We get a label of what is detected (in this case, it's always a face) as well as the confidence score (how confident the model is that it's a face).

IoT topics are an easy to transfer information from edge devices back into the cloud. Other functionality can be built around IoT topics; one example would be to monitor an IoT topic during hours you're away from home, and send a notification to yourself if a face is detected.

### Video Stream

We've seen in the IoT topics that our model outputs a label and a confidence score, but since it's a *detection* model it also outputs a *localization*, which in this case is bounding box coordinates. The best way to get a sense of this is to visualize the output for yourself.
 
Aside from publishing messages, the default project also streams inference output to a file locally on disk. If you register your own device, it's possible to view this over brwoser; in this lab, you are on the device itself, and so you can easily visualize it fom the local stream.

To view the output, open a terminal (on the Deeplens desktop UI, choose the top left button and search for terminal) and enter the following command:

`mplayer -demuxer lavf -lavfdopts format=mjpeg:probesize=32 /tmp/results.mjpeg`

Please visit https://docs.aws.amazon.com/deeplens/latest/dg/deeplens-viewing-device-output-on-device.html for more options to view the device stream

