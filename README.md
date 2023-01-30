# Create URL-shortner

1. To create a URL shortener using a Python Lambda function and trigger it with an API Gateway, follow these steps:
2. Create a Python function to shorten the URL. This function can use a hash function to map long URLs to short codes.
3. Create a new Lambda function in the AWS Management Console, and upload the code for your URL shortener.
4. In the Lambda function's configuration, create an API Gateway trigger, selecting "Create a new API" and choosing the REST API option.
5. In the API Gateway, create a resource and method (e.g. POST) to handle incoming URL shortening requests.
6. Map the incoming request payload to the appropriate parameters in the Lambda function.
7. Deploy the API Gateway and test the URL shortener by sending a POST request with a long URL.
8. Store the mapping of long URLs to short codes in a database (e.g. DynamoDB) for persistent storage.
9. Modify the Lambda function to look up the short code for a given long URL or return the original long URL for a given short code.
10. Update the API Gateway to handle GET requests for redirecting short URLs to their original long versions.
11. Deploy the updated API Gateway and test the complete URL shortener.

Here is a sample template.yaml file for deploying a Python-based Lambda function and API Gateway using the AWS Serverless Application Model (SAM):

```yaml:
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Resources:
  URLShortenerFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: url_shortener.lambda_handler
      Runtime: python3.8
      CodeUri: .
      Events:
        ShortenAPI:
          Type: Api
          Properties:
            Path: /shorten
            Method: post

  RedirectAPI:
    Type: AWS::Serverless::Api
    Properties:
      StageName: prod
      DefinitionBody:
        swagger: "2.0"
        info:
          title: URL Shortener API
        paths:
          /{shortCode}:
            get:
              responses:
                "302":
                  description: Redirect to original URL
                  headers:
                    Location:
                      type: string
              x-amazon-apigateway-integration:
                uri:
                  Fn::Join:
                  - ""
                  - - "arn:aws:apigateway:"
                    - Ref: AWS::Region
                    - ":lambda:path/2015-03-31/functions/"
                    - !GetAtt URLShortenerFunction.Arn
                    - "/invocations"
                httpMethod: POST
                type: aws_proxy
```
This YAML file defines two AWS resources: a Lambda function URLShortenerFunction and an API Gateway RedirectAPI. The Lambda function is triggered by a POST request to the /shorten endpoint, and the API Gateway handles GET requests to redirect short URLs to their original long versions. The code for the Lambda function should be located in the same directory as the template.yaml file.

You can deploy this SAM template using the AWS CLI command aws cloudformation deploy or using the AWS Management Console.

Here is an example of a simple Python-based Lambda function to shorten URLs:

```python:

import hashlib

def lambda_handler(event, context):
    # Extract the long URL from the incoming request
    long_url = event['long_url']

    # Generate a short code for the URL using a hash function
    short_code = hashlib.sha256(long_url.encode()).hexdigest()[:6]

    # Store the mapping of long URLs to short codes in a database
    # Example using AWS DynamoDB:
    #   dynamodb = boto3.resource('dynamodb')
    #   table = dynamodb.Table('url_shortener')
    #   table.put_item(Item={'long_url': long_url, 'short_code': short_code})

    # Return the short code for the URL
    return {
        'short_code': short_code
    }
```

This Lambda function takes in an incoming request with a JSON payload containing the key long_url, representing the original long URL that should be shortened. The function generates a short code for the URL using the SHA-256 hash function, and stores the mapping of long URLs to short codes in a database. In this example, the database used is AWS DynamoDB, but any database or data storage solution could be used. The short code is then returned in the response from the Lambda function.

Here is an example of a simple React-based front-end for the URL shortener:

```javascript:

    import React, { useState } from 'react';

    const ShortenForm = () => {
    const [longUrl, setLongUrl] = useState('');
    const [shortCode, setShortCode] = useState('');
    const [error, setError] = useState('');

    const handleSubmit = async (event) => {
        event.preventDefault();

        try {
        // Call the URL shortener API to generate a short code
        const response = await fetch('https://<api-gateway-url>/shorten', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ long_url: longUrl }),
        });
        const data = await response.json();
        setShortCode(data.short_code);
        } catch (err) {
        setError(err.message);
        }
    };

    return (
        <form onSubmit={handleSubmit}>
        <input
            type="text"
            value={longUrl}
            onChange={(event) => setLongUrl(event.target.value)}
            placeholder="Enter a long URL"
            required
        />
        <button type="submit">Shorten</button>
        {shortCode && (
            <div>
            Short URL: <a href={`https://<short-url-base>/${shortCode}`}>https://<short-url-base>/{shortCode}</a>
            </div>
        )}
        {error && <div>Error: {error}</div>}
        </form>
    );
    };

    export default ShortenForm;

```

This React component provides a form for entering a long URL and submitting it to the URL shortener API. When the form is submitted, the `fetch` function is used to call the API and retrieve the generated short code. The short URL is displayed in the form, and any errors from the API are displayed as well. The URL of the API Gateway should be replaced with the actual endpoint for the URL shortener API in the `fetch` call.
