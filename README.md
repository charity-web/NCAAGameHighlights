**Project Description**: This project uses RapidAPI to fetch NCAA highlights in a Docker container, then converts them with AWS MediaConvert.

**FEATURES**

Fetch Highlights: Obtain the highlights from the API and store them in an S3 bucket as a JSON file.

Process Videos: Extracts the first video URL from within the JSON file. Downloads the video file from the internet into the memory using the requests library. Saves the video as a new file in the S3 bucket.

AWS MediaConvert: Convert video files from one format to another, also configures the audio settings and stores the processed video back into an S3 bucket.

Storage: Use Amazon S3 for storing large files, including video content.



**FILE OVERVIEW**:

  The **"config.py"** script performs the following actions: 
  
	  Imports necessary environment variables and assigns them to Python variables, providing default values where appropriate. 
	
	  This approach allows for flexible configuration management, enabling different settings for various environments (e.g., development, staging, production) without modifying the source code.

  The **"fetch.py"** script performs the following actions:

	  Establishes the date and league that will be used to find highlights. We are using NCAA in this example because it's included in the free version. 
	
	  This will fetch the highlights from the API and store them in an S3 bucket as a JSON file (basketball_highlight.json)

  **"process_one_video.py"** performs the following actions:

	  Connects to the S3 bucket and retrieves the JSON file. 
	
	  Extracts the first video URL from within the JSON file. 
	  
	  Downloads the video file from the internet into the memory using the requests library. 
	  
	  Saves the video as a new file in the S3 bucket under a different folder (videos/) 
	  
	  Logs the status of each step

  **"mediaconvert_process.py"** performs the following actions:

	  Creates and submits a MediaConvert job 
	  
	  Uses MediaConvert to process a video file - configures the video codec, resolution and bitrate. Also configures the audio settings 
	  
	  Stores the processed video back into an S3 bucket

  **"run_all.py"** performs the following actions: 
  
	  Runs the scripts in a chronological order and provides buffer time for the tasks to be created.

  **".env"** file stores all over the environment variables, these are variables that we don't want to hardcode into our script.

  **"Dockerfile"** performs the following actions: 
  
	  Provides the step by step approach to build the image.
  
**TECHNOLOGIES**
  
Cloud Provider: AWS

Services used:

    AWS MediaConvert
    Amazon Simple Storage Service
    IAM
  
Programming Language: Python 3.x

Containerization: Docker

IAM Security: Custom least privilege policies for ECS task execution and API Gateway

**PREREQUISITES**: 

Before running the scripts, ensure you have the following:

1. **Create Rapidapi Account**

    Rapidapi.com account, will be needed to access highlight images and videos.

    For this example we will be using NCAA (USA College Basketball) highlights since it's included for free in the basic plan.

    Sports Highlights API is the endpoint we will be using.

2. **Verify prerequites are installed**

     Docker should be pre-installed in most regions docker --version

     AWS CloudShell has AWS CLI pre-installed aws --version

     Python3 should be pre-installed also python3 --version

3. **Retrieve AWS Account ID**
   
    Copy your AWS Account ID Once logged in to the AWS Management Console 
    
    Click on your account name in the top right corner, you will see your account ID Copy 
    
    Save this somewhere safe because you will need to update codes in the labs later
   
4. **Retrieve Access Keys and Secret Access Keys**

   You can check to see if you have an access key in the IAM dashboard Under Users, click on a user and then "Security Credentials" Scroll down until you see the Access Key section You will not be able to retrieve your secret access key so if you don't have that 
   somewhere, you need to create an access key.


**TECHNICAL DIAGRAM**

  ![Game Highlight processor pic](https://github.com/user-attachments/assets/2cf1d05c-1314-4699-a797-29e490ce3bd0)


**PROJECT STRUCTURE**

 ``` 
   src/
├── Dockerfile
├── config.py
├── fetch.py
├── mediaconvert_process.py
├── process_one_video.py
├── requirements.txt
├── run_all.py
├── .env
├── .gitignore
└── terraform/
    ├── main.tf
    ├── variables.tf
    ├── secrets.tf
    ├── iam.tf
    ├── ecr.tf
    ├── ecs.tf
    ├── s3.tf
    ├── container_definitions.tpl
    └── outputs.tf
```

**START HERE - Local**
**Step 1: Clone The Repo**
```
git clone https://github.com/charity-web/NCAAGameHighlights.git
cd src
```

**Step 2: Add API Key to AWS Secrets Manager**

```
aws secretsmanager create-secret \
    --name my-api-key \
    --description "API key for accessing the Sport Highlights API" \
    --secret-string '{"api_key":"YOUR_ACTUAL_API_KEY"}' \
    --region us-east-1
```
    
**Step 3: Create an IAM role or user**

In the search bar type "IAM"

Click Roles -> Create Role

For the Use Case enter "S3" and click next

Under Add Permission search for AmazonS3FullAccess, MediaConvertFullAccess and AmazonEC2ContainerRegistryFullAccess and click next

Under Role Details, enter "HighlightProcessorRole" as the name

Select Create Role

Find the role in the list and click on it Under Trust relationships Edit the trust policy to this: Edit the Trust Policy and replace it with this:

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": [
          "ec2.amazonaws.com",
          "ecs-tasks.amazonaws.com",
          "mediaconvert.amazonaws.com"
        ],
        "AWS": "arn:aws:iam::<"your-account-id">:user/<"your-iam-user">"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

**Step 4: Update .env file**
1. RapidAPI_KEY: Ensure that you have successfully created the account and select "Subscribe To Test" in the top left of the Sports Highlights API
2. AWS_ACCESS_KEY_ID=your_aws_access_key_id_here
3. AWS_SECRET_ACCESS_KEY=your_aws_secret_access_key_here
4. S3_BUCKET_NAME=your_S3_bucket_name_here
5. MEDIACONVERT_ENDPOINT=https://your_mediaconvert_endpoint_here.amazonaws.com
```
aws mediaconvert describe-endpoints
```
6. MEDIACONVERT_ROLE_ARN=arn:aws:iam::your_account_id:role/HighlightProcessorRole

**Step 5: Secure .env file**
```
chmod 600 .env
```

**Step 6: Locally Buikd & Run The Docker Container**
Run:
```
docker build -t highlight-processor .
```
Run the Docker Container Locally:

```
docker run --env-file .env highlight-processor
```
This will run fetch.py, process_one_video.py and mediaconvert_process.py and the following files should be saved in your S3 bucket:

Optional - Confirm there is a video uploaded to s3:///videos/first_video.mp4

Optional - Confirm there is a video uploaded to s3:///processed_videos/

![image](https://github.com/user-attachments/assets/1dd6dff0-7517-43ae-8606-946e82050ddd)



**What We Learned**
1. Working with Docker and AWS Services
2. Identity Access Management (IAM) and least privilege
3. How to enhance media quality

**Future Enhancements**
1. Using Terraform to enhance the Infrastructure as Code (IaC)
2. Increasing the amount of videos process and converted with AWS Media Convert
3. Change the date from static (specific point in time) to dyanmic (now, last 30 days from today's date,etc)
































