# Build-a-password-protected-website-using-AWS-Cloudfront-Lambda-and-S3
Here we are enhancing the security of your Amazon S3 website with CloudFront and Lambda@Edge.
we'll discuss how to add CloudFront and a Lambda function to your Amazon S3 website to enhance its security.

Creating an S3 Bucket
Open the AWS Console and ensure you are in the Northern Virginia region, as it supports Lambda@Edge. Additionally, ensure all AWS services are running in the same region.

First, create an S3 bucket. Here is a view of the AWS S3 page:





Give this bucket a name then and keep the bucket key enabled -





Here is the created S3 bucket. Now we will upload all of your files to an S3 bucket.



Leave all default settings as-is, i.e., “block all public access.” There’s no need for a bucket policy; you can leave that blank.



.



Creating a CloudFront Distribution
Next you all need to create your CloudFront distribution-

Amazon CloudFront is a web service that speeds up distribution of your static and dynamic web content, such as . html, . css, . js, and image files, to your users. CloudFront delivers your content through a worldwide network of data centers called edge locations.

In the AWS Console, search for CloudFront and select “Create Distribution”:-



Then you will see a screen like this -





Remember to select "Yes" in the origin access identity bucket policy and save changes.



It will take a couple of minutes for your distribution to be deployed; the status will switch from “Deploying” to “Deployed.”

Creating a Lambda Function
In the AWS Console, switch to Lambda. Here, we’ll build a function with username and password authorization and attach it to our CloudFront distribution.





Select the arrow by “Choose or create an execution role” and then “Create a new role from AWS policy templates.” Give the role a name. From Policy Templates, select “Basic Lambda@Edge permissions (for CloudFront trigger)” and then click “Create function”:



Once your Lambda is created, paste the following code into the index.js file of the Function Code section. Update the username and password by changing the authUser and authPass variables, then click “Save”:

To add code to your blog, use the following Markdown syntax:


COPY

COPY
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



In the upper menu, select Actions -> Deploy to Lambda@Edge as follows. Note that this will currently only work if you are in the Northern Virginia region.

Select the CloudFront distribution you created earlier from the select menu. Leave the Cache Behavior as *, and for the CloudFront Event select “Viewer Request”. Toggle “Include Body” and toggle “Confirm deploy to Lambda@Edge”. Finally, click “Deploy”.



It takes about 10-20 minutes to replicate your Lambda@Edge across all regions and edge locations.

Finalizing
Switch back to the CloudFront console. By this time, your distribution should be done deploying. Click on it, and you’ll see a set of descriptions under the “General” tab. Look for “Domain Name”: this is the URL of your new website.



You should now be prompted to enter the username and password defined earlier as “user” and “pass.”



Congratulations — you’ve successfully built a password-protected website on AWS using CloudFront and Lambda@Edge!
