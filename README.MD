Project for AWS serverless sentiment analysis  
  
This project implements a serverless sentiment analysis pipeline using AWS services. It captures the mood of text files that have been uploaded, it keeps the records, and you may look at them whenever you need. The architecture follows the AWS free tier, and it will only spend money when it is being used.

What does sentiment analysis involve?  

Sentiment analysis refers to a sub-category of the text analytics field that determines the emotional mood of text. It manifests itself in practice:  
• Product review: see what customers write about products in online shops.  
• Crowdsource: monitor the social media buzz, whether it is politics or talk about the brand.  
• Customer service: tag users that are angry or in a stressful state on chat records or ticket history.  
• Market research: go through customer responses in surveys or the forums.  
• Board communicator: automatically detect the existence of harmful posts or excessively negative chatter.  
• Chatbots: respond differently to a user when he or she is either positive or critical.

2. What makes the term serverless and AWS Free Tier a perfect match?  

This project is an event-driven, cloud-native project which is inexpensive, scalable, secure, and extendable with new functionality.  
• Affordable: no servers, thus it does not go beyond the boundaries of Free Tier.  
• Scaleable: no needed capacity planning- scale up or scale down automatically.  
• Safe: access control is based on IAM.  
• Modular: are already divided into small and loosely coupled parts, which means that additional modules such as notifications, dashboards, or additional analytics will be easily integrated.


3. Big takeaways  

• In the case of sentiment analysis, it is still possible to read between the lines of the text and decipher the emotion behind the words.  
Serverless No matter how large or small the demands placed on a computing system, it is scalable, adjustable, and cost-friendly, and therefore you are free to concentrate on development rather than running equipment.  
• AWS Free Tier allows you to explore without afraid of huge payments.


Overview

1. To save the text input files, create an S3 bucket that is free.
2. Drop `.txt` files in (e.g., user reviews, forum posts) to that bucket.
3. A Lambda event publishes events whenever new files appear.
4. Lambda:

   1. Fetches the file contents of the S3.
   2. Sends them to Amazon Comprehend to have **sentiment analysis** done (Positive, Negative, Neutral, Mixed).
   3. Stores the output in a dynamodb table comprising of the filename and a timestamp.

The resulting data about the sentiment denoted by a sentiment label, timestamp, and filename is easy to access in DynamoDB after the job is finished. At this point, you can make queries to the table, do some number crunchin,g and tie the information to dashboards or other applications. All of it operates within the Amazon Web Services Free Tier, and there is no invoice until and unless you are using it.

Technologies

Amazon S3 - Raw text input comes under the format of user reviews, forum posts and so on, as it comes directly in raw file format and is stored in Amazon S3.  
AWS Lambda - No-Kill compute called on file upload and sentiment analysis code logic occurred.  
AWS Natural Language Processing (NLP), Amazon Comprehend, is a service that identifies sentiment.  
Amazon DynamoDB- NoSQL database to store the sentiment results, such as their filename, sentiment and time.  
Amazon CloudWatch - Amazon Cloudwatch keeps track of the Lambda logs and other metrics of execution.  
IAM Roles - Safety Gives Lambda-Onto S3, Comprehend, and DynamoDB.

Deployment and maintenance are very easy because all the components are fully managed.



Detailed Explanation of Lambda Function Code


import boto3
import json
import datetime

def lambda_handler(event, context):
    print("Lambda started...")
    try:
        # Initialize AWS clients for S3, Comprehend, and DynamoDB
        s3 = boto3.client('s3')  # Read uploaded files
        comprehend = boto3.client('comprehend')  # Sentiment analysis
        dynamodb = boto3.resource('dynamodb')  # Store results
        print("Clients initialized...")

        # Connect to DynamoDB table named 'SentimentResults'
        table = dynamodb.Table('SentimentResults')

        # Extract bucket name and file name from the S3 event trigger
        bucket = event['Records'][0]['s3']['bucket']['name']
        file_key = event['Records'][0]['s3']['object']['key']
        print(f"Bucket: {bucket}, Key: {file_key}")

        # Fetch the text file content from S3
        response = s3.get_object(Bucket=bucket, Key=file_key)
        text = response['Body'].read().decode('utf-8')  # Decode bytes to string
        print(f"Text extracted: {text[:100]}...")  # Show snippet for debugging

        # Use Amazon Comprehend to detect sentiment of the text
        sentiment_response = comprehend.detect_sentiment(Text=text, LanguageCode='en')
        sentiment = sentiment_response['Sentiment']  # Extract main sentiment label
        print(f"Sentiment result: {sentiment_response}")

        # Store sentiment result in DynamoDB with filename as partition key
        table.put_item(
            Item={
                'Filename': file_key,  # Must exactly match DynamoDB partition key name
                'Sentiment': sentiment,
                'Timestamp': datetime.datetime.utcnow().isoformat()  # Current UTC timestamp
            }
        )
        print(f"Stored in DynamoDB: {file_key}, {sentiment}")

        # Return HTTP success response (useful if called via API Gateway)
        return {
            'statusCode': 200,
            'body': json.dumps({'Sentiment': sentiment})
        }

    except Exception as e:
        # Log any errors to CloudWatch and raise to signal failure
        print("ERROR occurred:", str(e))
        raise





Code Breakdown

So when you start installing a project such as this you need to do one thing first: configure your AWS clients so they know how to communicate with the various services, you have to communicate with, which are: S3 (where you store your files), Comprehend (your sentiment analyzer), and DynamoDB (where you store the results). All that configuration is done in some lines of initialization code. 

Then you are crafting a Lambda that is listening to S3 events. Once a new file is uploaded, the Lambda stores the name of the bucket as well as the file name and proceeds. 

Third is to read the file in S3, convert it into in memory string, and give that string to the detect sentiment() API in the Comprehend. The API returns a classification: Positive, Negative, Neutral or Mixed, and you store this in a DynamoDB table together with the filename and some timestamp.

During the process, the script scans against misses and makes the error noted in CloudWatch in order to troubleshoot later. When a part breaks, something, the Lambda will report such failure; hence, no one has the responsibility to sit and supervise the entire system.

Conclusion

This combination demonstrates how, in practice, it is possible to launch and turn off serverless services and conduct automatic sentiment analysis without any concerns with server management or infrastructure. Since AWS takes the load of scaling, securing, and costing, it suits so well when one intends to do some social-media monitoring, customer-feedback analysis, and so on.



