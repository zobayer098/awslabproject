# awslabproject
sample lab using AWS Lambda, Polly, Translate, SES, S3 Bucket

# Scenario
using AWS Polly, Translate and Lambda, create a complete and step by step lab guide that will allow a customer support department to communicate to customers when a significant event occurs. For example, if a system has emergency downtime or an urgent requirement, the customers should be notified using either email or voicemail.  It will produce a voicemail in the customer’s preferred language using a Lambda function that will invoke Translate to convert text to other languages, and invoke Polly to convert the text to voice and generate an MP3 file in S3. Also an example of triggering the Lambda function. Lastly, generate a pre-signed URL to the MP3 file and email it to the customer

