### Lab Objective 
Using AWS Polly, Translate and Lambda, create a complete and step by step lab guide that will allow a customer support department to communicate to customers when a significant event occurs. For example, if a system has emergency downtime or an urgent requirement, the customers should be notified using either email or voicemail.  It will produce a voicemail in the customer’s preferred language using a Lambda function that will invoke Translate to convert text to other languages, and invoke Polly to convert the text to voice and generate an MP3 file in S3. Also an example of triggering the Lambda function. Lastly, generate a pre-signed URL to the MP3 file and email it to the customer

### Prerequisites
        An AWS account
        Basic knowledge of Python
        Basic knowledge of Lambda, Polly, IAM role with Permission, SES, S3 bucket and Translate services

### Step- 1 S3 Bucket
        Create an S3 Bucket:
        Log in to your AWS Management Console.
        In the AWS Management Console, search for “S3” in the services search bar.
        Click on “S3” from the search results.
        Click the “Create bucket” button.
        The “Create bucket” page will open.
        Configure the Bucket:
        For the Bucket name, enter mytests3bucket777.
        The bucket name must be unique within your AWS account.
        It should be between 3 and 63 characters long.
        Use only lowercase letters, numbers, dots (.), and hyphens (-).
        Begin and end with a letter or number.
        Choose the AWS Region where you want the bucket to reside.
        Select a region close to you to minimize latency and costs.
        Objects stored in a region never leave that region unless explicitly transferred.
        Click the “Create bucket” button. Leave other options as default.
        Bucket Created: Your S3 bucket named mytests3bucket777 is now created.


### Step- 2 Amazon SES: Configure source and target email in Amazon SES to send an email containing the pre-signed URL to the customer.
        Add email accounts to send email notification using Amazon SES 
        Navigate to Amazon SES:
        In the AWS Management Console, search for “SES” in the services search bar.
        Click on “SES” from the search results.
        Verify Your Domain or Email Address:
        Before sending emails, you need to verify your domain or email address.
        Go to the “Email Addresses” section and click “Verify a New Email Address.”
        Follow the instructions to verify your email address

### Step - 3 IAM role and associated permission 
        Create an IAM role with permissions to access S3, Polly, and Translate.
        Navigate to IAM (Identity and Access Management):
        In the AWS Management Console, search for “IAM” in the services search bar.
        Click on “IAM” from the search results.
        Create a New Role:
        In the left navigation pane, click on “Roles.”
        Click the “Create role” button.
        Select Lambda as the Trusted Entity:
        Choose “AWS service” as the trusted entity.
        Select “Lambda” as the use case.
        Attach Policies:
        Search for and attach the following managed policies:
        AmazonPollyFullAccess
        TranslateFullAccess
        AmazonS3FullAccess
        AmazonSESFullAccess
        These policies grant the necessary permissions for Polly, Translate, S3, and SES.
        Review and Name the Role:
        Give your role a name (e.g., LambdaPollyTranslateRole).
        Optionally, add a description for the role.
        Click the “Create role” button.
        Role Created:
        Your IAM role named LambdaPollyTranslateRole is now created.
        Note down the ARN (Amazon Resource Name) of this role.
        Attach the Role to Your Lambda Function:
        When creating or updating your Lambda function, specify this role as the execution role.

### Step - 4 Create and Write the Lambda Function


        In the AWS Management Console, search for “Lambda” in the services search bar.
        Click on “Lambda” from the search results.
        Create a New Lambda Function:
        Click the “Create function” button.
        Choose “Author from scratch.”
        Fill in the following details:
        Function name: Give your Lambda function a unique name.
        Runtime: Choose the runtime environment ( use Python for this scenario).
        Existing role: Select the existing role LambdaPollyTranslateRole.
        Click the “Create function” button.
        Configure Your Lambda Function:
        In the “Function code” section, copy and paste the following code

## Python code for Lambda
        import boto3
        import os
        import time

        # Initialize clients
        # boto3 is the AWS SDK for python
        
        s3 = boto3.client('s3')
        polly = boto3.client('polly')
        translate = boto3.client('translate')
        ses = boto3.client('ses')

            # Get the parameters from the event. This function is the entry point for an AWS Lambda function. It receives an event (containing input data) and a context (runtime information). Extracts relevant parameters from the event: text, target_language, and email
            
        def lambda_handler(event, context):
            text = event['text']
            target_language = event['targetLanguage']
            email = event['email']

            # Translate the text
            result = translate.translate_text(Text=text, SourceLanguageCode='en', TargetLanguageCode=target_language)
            translated_text = result['TranslatedText']

            # Convert the translated text to speech, reads the audio streat into audio_stream
            response = polly.synthesize_speech(VoiceId='Joanna', OutputFormat='mp3', Text=translated_text)
            audio_stream = response['AudioStream'].read()

            # Upload the speech file to S3 which generate a unique key (filename) based on current timestamp and upload the s3 bucket with generated key
            key = f"{int(time.time())}.mp3"
            
            # s3.put_object(Bucket='mytests3bucket777', Key=key, Body=audio_stream, ACL='public-read')
            s3.put_object(Bucket='mytests3bucket777', Key=key, Body=audio_stream)

            # Generate a pre-signed URL for accessing audio file and url expires in 1 hour
            url = s3.generate_presigned_url('get_object', Params={'Bucket': 'mytests3bucket777', 'Key': key}, ExpiresIn=3600)

            # Send an email with the URL with status code 200 indicating email was sent with voicemail URL and body f-string (formatted string) to embed expression inside strings. f string contains placeholders for variables
            ses.send_email(
                Source='atmzht@gmail.com',
                Destination={'ToAddresses': [email]},
                Message={
                    'Subject': {'Data': 'Voicemail Notification'},
                    'Body': {'Text': {'Data': f"Here is your voicemail: {url}"}}
                }
            )

            return {
                'statusCode': 200,
                'body': f"Email sent to {email} with voicemail URL."
            }

### Step -5 Test and Trigger the Lambda Function
            ## Sample test
  
   {
            "text": "Hello, this is a test message.",
            "targetLanguage": "es",
            "email": "abu_tareq@live.co.uk"
            }

 ### Final step - Test and Verify 
            Notice new MP3 upload in the S3 bucket and email is sent to the customer with presigned URL of the MP3 stored in the S3 bucket
