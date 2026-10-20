# AWS Text-to-Speech Automation

### Project Overview:
This project is a serverless AWS-based Text-to-Speech application that converts text into an audio file using Amazon Polly.
The project uses Amazon S3 for storing input and output files, AWS Lambda for processing the text, and Amazon Polly for converting the text into speech.
The goal of this project was to understand how different AWS services can work together to build a simple, automated, and serverless cloud workflow.

### Architeecture Workflow
<img height="1080" alt="Amazon Polly" src="Assets/architecture.png" />

### Steps to be Performed:
1. Exploring Amazon Polly
2. Creating an IAM role: 
```sh
RoleName: PollyTranslationRole
```
### Policies:
```sh
AmazonPollyFullAccess
AmazonS3FullAccess
AWSLambdaBasicExecutionRole
```
3. Creating an Source and Destiation S3 Buckets:
```sh
Source S3 Bucket Name: polly-text-files
Destiation S3 Bucket Name: polly-translate-audio-files
```

4. Writing Lambda function code:
```python
import boto3
import time
import json

polly = boto3.client("polly")
s3 = boto3.client("s3")

def lambda_handler(event, context):  # This name is required
    try:
        # Get S3 file details from trigger event
        record = event["Records"][0]["s3"]
        source_bucket = record["bucket"]["name"]
        file_key = record["object"]["key"]

        # Read text file from twy-polly-text-files-storage-bucket
        obj = s3.get_object(Bucket="polly-text-files", Key=file_key)
        text = obj["Body"].read().decode("utf-8")

        # Convert text → speech
        polly_response = polly.synthesize_speech(
            Text=text,
            OutputFormat="mp3",
            VoiceId="Joanna"
        )

        if "AudioStream" not in polly_response:
            raise Exception("Polly did not return audio data")

        # Read audio stream
        audio_bytes = polly_response["AudioStream"].read()

        # Destination audio file name
        audio_key = f"speech-{int(time.time()*1000)}.mp3"

        # Upload audio to twy-polly-audio-files-storage-bucket
        s3.put_object(
            Bucket="polly-translate-audio-files",
            Key=audio_key,
            Body=audio_bytes,
            ContentType="audio/mpeg",
            ContentLength=len(audio_bytes)
        )

        return {
            "statusCode": 200,
            "body": json.dumps({
                "message": "Conversion successful",
                "source_file": file_key,
                "audio_file": audio_key
            })
        }

    except Exception as e:
        print("Error:", str(e))
        return {
            "statusCode": 500,
            "body": json.dumps({"error": str(e)})
        }

```

6. Checking the output of Amazon Polly

### Services Used: 
1. Amazon Polly: Converts text to life like speech with customizable features.
2. AWS Management Console: Manages accounts and configures Amazon Polly.
3. AWS IAM: Ensures secure access by managing user permissions.

### Estimated Time & Cost:
* This project is estimated to take about 20-30 minutes
* Cost: Free (When using the AWS Free Tier)

Congratulations! You have successfully completed the project of text to speech translation using Amazon Polly, Lambda and S3 bucket.
