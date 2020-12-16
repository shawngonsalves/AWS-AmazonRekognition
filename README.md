# CS446_AmazonRekognition

Deployment and execution :-

First download the AWS CLI from aws official website.
1. Download this project directory on your local machine.
2. Once you downloaded, open command promt and navigate to the project directory & then navigate to /code subdirectory and install the following:
   pip install requests_aws4auth --target .
   pip install elasticsearch --target .
3. Create S3 bucket for deployment enter the following command :-
   aws s3 mb s3://cs446reokognition --region us-east-1
4. Open the Rek_Demo.yaml with your notepad and change the change the BucketName property of 'RekS3Bucket' to your choosing (say: S3Bucket). This is the bucket which will house the input images and the blacklist images to the StepFunctions state machine.
5. Package the contents and prepare deployment package using the following command :-
   aws cloudformation package --template-file Rek_Demo.yaml --output-template-file Rekogoutput.yaml --s3-bucket cs446reokognition --region us-east-1
6. Deploy the package :-
   aws cloudformation deploy --template-file Rekogoutput.yaml --stack-name cs446Stack --capabilities CAPABILITY_IAM --region us-east-1
7. After performing all the above steps, navigate to CloudFormation in aws console there you'll see stack is created after deployment and stack and simultaniously when you navigate to S3 bucket the new S3 bucket with the name 'cs446reokognition-us-east-1' will be created as part of stack. Similarly, inside Lambda service all code(.py) files for our project will be deployed as lambda is serverless which runs your code to events and automatically manages underlying computing resouces for you. Similarly, inside AWS StepFunctions service, the state machine is created as a part of stack & in the AWS ElasticSearch the domain end point is created where the results of our project will be logged.

Execution :-

The StepfunctionInput.json file contains sample invocations that you can use to trigger the worklow. The Step Functions state machine expects input to be passed in a particular format. You can change the values, but the structure is expected by the lambda functions that are triggered by the state function.

Upload Images:
Upload the images from the project-directory/images/test/ folder to S3Bucket/reInvent2017/RekognitionDemo/InputImages/.
Upload the images from the project-directory/images/blackList/ folder to S3Bucket/reInvent2017/RekognitionDemo/BlacklistImages/.
Navigate to your Step Functions console, choose the state machine that was deployed.
Click on 'New Execution'.
In the input window, you can paste any of the sample inputs from StepfunctionInput.json, or point to your own image file.
Click on 'Start Execution'.

Output:-

In the AWS ElasticSearch the domain end point is created where the results of our project will be logged.

Steps of our project execution :-

Once the image lands on the S3 bucket, the step functions flow can be triggered using a S3 event + Lambda (The S3 event triggering code is not included in this repository). Step functions then executes a series of checks with Amazon Rekognition and builds a json template of its findings, as it progresses through the workflow. The process is successful if all the steps succeed and unsuccessful if any of them fails. The results are logged into Amazon Elasticsearch service and you use Kibana to visualize the results.

This repository contains the sample code for Lambda functions and Step functions as well as a SAM template to deploy the resources in an AWS region of your choice.

The following checks are implemented as a part of this workflow:

1.Check for faces in the uploaded image.
  Fail if no face or more than one face detected.
  Fail if mouth is detected as open, eyes detected as closed, face not looking in the right direction etc.

2.Check for known celebrities
  Fail if the uploaded image matches a known celebrity.

3.Image moderation filter
  Fail if image contains explicit nudity or suggestive content

4.Blacklist / duplicate check
  Fail if the image is already uploaded by somebody else
  Fail if the image matches a custom black list that you maintain
