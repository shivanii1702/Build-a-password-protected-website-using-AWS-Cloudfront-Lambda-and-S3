# Build-a-password-protected-website-using-AWS-Cloudfront-Lambda-and-S3
Here we are enhancing the security of your Amazon S3 website with CloudFront and Lambda@Edge.
we'll discuss how to add CloudFront and a Lambda function to your Amazon S3 website to enhance its security.

Creating an S3 Bucket
Open the AWS Console and ensure you are in the Northern Virginia region, as it supports Lambda@Edge. Additionally, ensure all AWS services are running in the same region.

First, create an S3 bucket. Here is a view of the AWS S3 page:
![image](https://github.com/user-attachments/assets/cc8fa018-5f88-4eae-a13d-d2415222ef0b)

![image](https://github.com/user-attachments/assets/b7de6811-8ec3-40ec-b88c-c0660f38c117)


Give this bucket a name then and keep the bucket key enabled -
![image](https://github.com/user-attachments/assets/a013449f-4308-4d26-805a-de63db546705)

![image](https://github.com/user-attachments/assets/29c5e99b-a54d-47e5-b55e-4e5afc8b4915)


Here is the created S3 bucket. Now we will upload all of your files to an S3 bucket.

![image](https://github.com/user-attachments/assets/f7d5bd6a-2033-46b4-9f57-9f35361d6401)

Leave all default settings as-is, i.e., “block all public access.” There’s no need for a bucket policy; you can leave that blank.

![image](https://github.com/user-attachments/assets/e9c0fd89-b030-4867-87ca-2ec788f4fc87)

![image](https://github.com/user-attachments/assets/8b604943-a5df-4a05-af32-df901b2af3ad)

Creating a CloudFront Distribution
Next you all need to create your CloudFront distribution-

Amazon CloudFront is a web service that speeds up distribution of your static and dynamic web content, such as . html, . css, . js, and image files, to your users. CloudFront delivers your content through a worldwide network of data centers called edge locations.

In the AWS Console, search for CloudFront and select “Create Distribution”:-

![image](https://github.com/user-attachments/assets/3b9fb196-8349-4b2f-9d9a-84e445ed7302)

Then you will see a screen like this -

![image](https://github.com/user-attachments/assets/da16c478-27cc-46f7-8d3d-d871329b8ce2)

![image](https://github.com/user-attachments/assets/577f457a-9d99-416e-b2f4-8eaa5ecd7cc5)


Remember to select "Yes" in the origin access identity bucket policy and save changes.

![image](https://github.com/user-attachments/assets/2956cc11-8f53-430a-9345-f56ded1f0e0a)

It will take a couple of minutes for your distribution to be deployed; the status will switch from “Deploying” to “Deployed.”

Creating a Lambda Function
In the AWS Console, switch to Lambda. Here, we’ll build a function with username and password authorization and attach it to our CloudFront distribution.
![image](https://github.com/user-attachments/assets/f54c0e5f-2bb1-4af1-8340-9c9eb9c1ff09)

![image](https://github.com/user-attachments/assets/dbaecd60-d926-4643-af9b-d803d8e391dc)

Select the arrow by “Choose or create an execution role” and then “Create a new role from AWS policy templates.” Give the role a name. From Policy Templates, select “Basic Lambda@Edge permissions (for CloudFront trigger)” and then click “Create function”:

![image](https://github.com/user-attachments/assets/ffa613dd-d715-4df4-bb9b-a3686922c6a5)


Once your Lambda is created, paste the following code into the index.js file of the Function Code section. Update the username and password by changing the authUser and authPass variables, then click “Save”:

To add code to your blog, use the following Markdown syntax:

'use strict';  
exports.handler = (event, context, callback) => {  

  // Get request and request headers
  const request = event.Records[0].cf.request;  
  const headers = request.headers;  

  // Configure authentication
  const authUser = 'user';  
  const authPass = 'pass';  

  // Construct the Basic Auth string
  const authString = 'Basic ' + new Buffer(authUser + ':' + authPass).toString('base64');  

  // Require Basic authentication
  if (typeof headers.authorization == 'undefined' || headers.authorization[0].value != authString) {  
    const body = 'Unauthorized';  
    const response = {  
      status: '401',  
      statusDescription: 'Unauthorized',  
      body: body,  
      headers: {  
        'www-authenticate': [{key: 'WWW-Authenticate', value:'Basic'}]  
      },  
    };  
    callback(null, response);  
  } 
   // Continue request processing if authentication passed 
  callback(null, request);  
};
This is what that looks like in the Lambda console:

![image](https://github.com/user-attachments/assets/aa5ad835-f7da-4db9-97b0-4cac6091bb84)


In the upper menu, select Actions -> Deploy to Lambda@Edge as follows. Note that this will currently only work if you are in the Northern Virginia region.

Select the CloudFront distribution you created earlier from the select menu. Leave the Cache Behavior as *, and for the CloudFront Event select “Viewer Request”. Toggle “Include Body” and toggle “Confirm deploy to Lambda@Edge”. Finally, click “Deploy”.

![image](https://github.com/user-attachments/assets/2430f6ec-ada5-4a47-a28e-898668c47a5d)


It takes about 10-20 minutes to replicate your Lambda@Edge across all regions and edge locations.

Finalizing
Switch back to the CloudFront console. By this time, your distribution should be done deploying. Click on it, and you’ll see a set of descriptions under the “General” tab. Look for “Domain Name”: this is the URL of your new website.

![image](https://github.com/user-attachments/assets/63c56b59-d007-48cc-9357-dfb848dcd614)


You should now be prompted to enter the username and password defined earlier as “user” and “pass.”

![image](https://github.com/user-attachments/assets/3771ea09-5375-4b83-9bb6-5bd7c61757df)


Congratulations — you’ve successfully built a password-protected website on AWS using CloudFront and Lambda@Edge!
