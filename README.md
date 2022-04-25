# Voice Based Smart Photo Search Album

### A Voice based Photo Album using AWS Services | Cloud Computing and Big Data Systems

###### S3 BUCKET LINK FOR THE album : http://myawsbucket-frontend.s3-website-us-east-1.amazonaws.com/
---

## INSTRUCTIONS :

Implemented a photo album web application, that can be searched using natural language through both text and voice. We used AWS Lex, ElasticSearch, and Rekognition to create an intelligent search layer to query your photos for people, objects, actions, landmarks and more.

## DESCRIPTION:

This project had eight components:

**1. Launch an ElasticSearch instance:**

    a. Using AWS ElasticSearch service, create a new domain called “photos” .

    b. Make note of the Security Group ( SG1 ) you attach to the domain.

    c. Deploy the service inside a VPC.

      i. This prevents unauthorized internet access to your service.

**2. Upload & index photos**

     a. Create a S3 bucket ( B2 ) to store the photos.

     b. Create a Lambda function ( LF1 ) called “index-photos” .

       i. Launch the Lambda function inside the same VPC as ElasticSearch. This ensures that the function can reach the ElasticSearch instance.

      ii. Make sure the Lambda has the same Security Group ( SG1 ) as ElasticSearch.

    c. Set up a PUT event trigger on the photos S3 bucket ( B2 ), such that whenever a photo gets uploaded to the bucket, it triggers the Lambda function ( LF1 ) to index it.

        i. To test this functionality, upload a file to the photos S3 bucket ( B2 ) and check the logs of the indexing Lambda function ( LF1 ) to see if it got invoked. If it did, your setup is complete.
      -If the Lambda ( LF1 ) did not get invoked, check to see if you set up the correct permissions5 for S3 to invoke your Lambda function.

    d. Implement the indexing Lambda function ( LF1 ):
      i. Given a S3 PUT event ( E1 ) detect labels in the image, using Rekognition (“detectLabels” method).

      ii. Use the S3 SDK’s headObject method7 to retrieve the S3 metadata created at the object’s upload time. Retrieve the x-amz-meta-customLabels metadata field, if applicable, and create a JSON array ( A1 ) with the labels.

      iii. Store a JSON object in an ElasticSearch index (“photos”) that references the S3 object from the PUT event ( E1 ) and append string labels to the labels array ( A1 ), one for each label detected by Rekognition.

      Use the following schema for the JSON object:
      {
            “objectKey”: “my-photo.jpg”,
            “bucket”: “my-photo-bucket”,
            “createdTimestamp”: “2018-11-05T12:40:02”,
            “labels”: [
                        “person”,
                        “dog”,
                        “ball”,
                        “park”
                    ]
      }

**3. Search:**

    a. Create a Lambda function ( LF2 ) called “search-photos” .

        i. Launch the Lambda function inside the same VPC as ElasticSearch. This ensures that the function can reach the ElasticSearch instance.

        ii. Make sure the Lambda has the same Security Group ( SG1 ) as ElasticSearch.

    b. Create an Amazon Lex bot to handle search queries.

        i. Create one intent named “SearchIntent”.

        ii. Add training utterances to the intent, such that the bot can pick up both keyword searches (“trees”, “birds”), as well as sentence searches (“show me trees”, “show me photos with trees and birds in them”).

        ● You should be able to handle at least one or two keywords per query.

    c. Implement the Search Lambda function ( LF2 ):

        i. Given a search query “q”, disambiguate the query using the Amazon Lex bot.

        ii. If the Lex disambiguation request yields any keywords ( K 1 , …, K n ), search the “photos” ElasticSearch index for results, and return them accordingly (as per the API spec).

        ● You should look for ElasticSearch SDK libraries to perform the search.

        iii. Otherwise, return an empty array of results (as per the API spec).


**4. Build the API layer:**

    a. Build an API using API Gateway.

       i. The Swagger API documentation for the API can be found here: https://github.com/001000001/ai-photo-search-columbia-f2018/blob/master/swagger.yaml

    b. The API should have two methods:

        i. PUT /photos

            Set up the method as an Amazon S3 Proxy8. This will allow API Gateway to forward your PUT request directly to S3.

            ● Use a custom x-amz-meta-customLabels HTTP header to include any custom labels the user specifies at upload time.

        ii. GET /search?q={query text}

            Connect this method to the search Lambda function ( LF2 ).

    c. Setup an API key for your two API methods.

    d. Deploy the API.

    e. Generate a SDK for the API ( SDK1 ).

**5. Frontend:**

    a. Build a simple frontend application that allows users to:

        i. Make search requests to the GET /search endpoint

        ii. Display the results (photos) resulting from the query

        iii. Upload new photos using the PUT /photos

        ● In the upload form, allow the user to specify one or more custom labels, that will be appended to the list of labels detected automatically by Rekognition (see 2.d.iii above). These custom labels should be converted to a comma-separated list and uploaded as part of the S3 object’s metadata9 using a x-amz-meta-customLabels metadata HTTP header. For instance, if you specify two custom labels at upload time, “Sam” and “Sally”, the metadata HTTP header should look like: ‘x-amz-meta-customLabels’: ‘Sam, Sally’

    b. Create a S3 bucket for your frontend ( B1 ).

    c. Set up the bucket for static website hosting (same as HW1).

    d. Upload the frontend files to the bucket ( B2 ).

    e. Integrate the API Gateway-generated SDK ( SDK1 ) into the frontend, to connect your API.

**6. Implement Voice accessibility in the frontend:**

    a. Give the frontend user the choice to use voice rather than text to perform the search.

    b. Use Amazon Transcribe10 on the frontend to transcribe speech to text (STT) in real time11, then use the transcribed text to perform the search, using the same API like in the previous steps.

    c. Note: You can use a Google-like UI (see below) for implementing the search: 1. input field for text searches and 2. microphone icon for voice interactions.

**7. Deploy your code using AWS CodePipeline:**

    a. Define a pipeline (P1) in AWS CodePipeline that builds and deploys the code for/to all your Lambda functions

    b. Define a pipeline (P2) in AWS CodePipeline that builds and deploys your frontend code to its corresponding S3 bucket

**8. Create a AWS CloudFormation13 template for the stack:**

    a. Create a CloudFormation template (T1) to represent all the infrastructure resources (ex. Lambdas, ElasticSearch, API

---

## WORKFLOW DIAGRAM

![alt text](https://github.com/rajat10cube/Voice-Based-Smart-Photo-Album/blob/main/Architecture/diagram.png)

## Sample Use Case

---

## SERVICES USED

### AWS SERVICES

#### ▫️ AWS S3 BUCKET:

Amazon Simple Storage Service (Amazon S3) is an object storage service that offers industry-leading scalability, data availability, security, and performance. Object storage service that offers industry-leading scalability, data availability, security, and performance.

#### ▫️ AWS LEX:

Amazon Lex is a fully managed artificial intelligence (AI) service with advanced natural language models for building conversational interfaces into applications such as _Build virtual agents and voice assistants_ ,_Automate informational responses_ ,and _Improve productivity with application bots_.

#### ▫️ AWS LAMBDA :

AWS Lambda is a serverless compute service that lets you run code without provisioning or managing servers, creating workload-aware cluster scaling logic, maintaining event integrations, or managing runtimes. With Lambda, we ran code for the application with zero administration.

#### ▫️ AWS API GATEWAY:

Amazon API Gateway is a fully managed service that makes it easy for developers to create, publish, maintain, monitor, and secure APIs at any scale. APIs act as the "front door" for applications to access data, business logic, or functionality from your backend services. Using API Gateway, we were able to create RESTful APIs for the chatbot which communicate with the lambda functions.

#### ▫️ AWS Elasticsearch :

AWS Elasticsearch is a distributed search and analytics engine built on Apache Lucene. Since its release in 2010, Elasticsearch has quickly become the most popular search engine and is commonly used for log analytics, full-text search, security intelligence, business analytics, and operational intelligence use cases.

#### ▫️ AWS Transcribe :

Amazon Transcribe makes it easy for developers to add speech to text capabilities to their applications. Audio data is virtually impossible for computers to search and analyze. Therefore, recorded speech needs to be converted to text before it can be used in applications.

#### ▫️ AWS Code Pipeline :

AWS CodePipeline is a fully managed continuous delivery service that helps you automate your release pipelines for fast and reliable application and infrastructure updates. CodePipeline automates the build, test, and deploy phases of your release process every time there is a code change, based on the release model you define.

#### ▫️ AWS Code Pipeline :

Speed up cloud provisioning with infrastructure as code. AWS CloudFormation lets you model, provision, and manage AWS and third-party resources by treating infrastructure as code.

---
