# TravelWebsite
<br>

## Summary
<br>

# Objective
This code deploys an s3 bucket and a cloudfront distribution to AWS. The objective of this project is to host a static website on amazon S3 and access the website using amazon's Content Delivery Network (CDN), Cloudfront. 
<br>

# Technology Stack
* Amazon S3
* AWS Cloudfront
* HTML, JS, CSS
<br>

# Table of Content
* [**Insight**] (#ins)
* [**Instructions**] (#instr)
* [**Via-Cloudformation**] (#cfn)
* [**Resources**] (#res)
<br>

# Insight <a id='ins'></a>
* **Amazon S3:** Amazon Simple Storage Service (Amazon S3) is an object storage service offering industry-leading scalability, data availability, security, and performance. Customers of all sizes and industries can store and protect any amount of data for virtually any use case, such as data lakes, cloud-native applications, and mobile apps.
* **AWS Cloudfront:** Amazon CloudFront is a fast content delivery network (CDN) service that securely delivers data, videos, applications, and APIs to customers globally with low latency, high transfer speeds, all within a developer-friendly environment. CloudFront works seamlessly with services, including Amazon Shield Standard for DDoS mitigation, and Amazon S3, Elastic Load Balancing, or Amazon EC2 as origins for your applications.
<br>

# Instructions <a id='instr'></a>
- Create a bucket `<bucket-name>` and make it publicly accessible
- Enable `static website hosting` on the Bucket
- Upload the web files to s3
- Create a `Cloud Front Distribution`
- Make the bucket a `Cloud Front Origin Access Identity` for that Distribution
- Create a `bucket Policy` that `grants public read access` to the `CLoud Front Distriubtion only`
- Access the website via the cloud front Distrubution
<br>

# Via Cloudformation <a id='cfn'></a>
<details>
<summary> Expand to see Details </summary>

- Make sure you are in the root directory of this repo `travelwebsite`
- Run `aws configure` to set up your CLI
- Deploy this [CloudFormation template](./cfn/template.yaml) to AWS and save the outputs asn env variables

  - [`create-stack`](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/cloudformation/create-stack.html)

  ```
  export STACK_NAME=travel-website
  ```

  ```
  aws cloudformation create-stack \
  --stack-name $STACK_NAME \
  --template-body file://cloudformation/template.yaml
  ```

- Describe the stack to get the outputs (Bucket name and url, CDN ID and domain name )

  - [`describe-stacks`](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/cloudformation/describe-stacks.html)

  ```
  aws cloudformation describe-stacks --stack-name $STACK_NAME
  ```

  ```
  aws cloudformation describe-stacks --stack-name $STACK_NAME --query "Stacks[].Outputs"
  ```

  ```
  export BUCKET_NAME=$(aws cloudformation describe-stacks --stack-name $STACK_NAME --query "Stacks[*].Outputs[0].OutputValue" --output text)
  ```

  ```
  export CDN_ID=$(aws cloudformation describe-stacks --stack-name $STACK_NAME --query "Stacks[*].Outputs[1].OutputValue" --output text)
  ```

  ```
  export BUCKET_URL=$(aws cloudformation describe-stacks --stack-name $STACK_NAME --query "Stacks[*].Outputs[2].OutputValue" --output text)
  ```

  ```
  export CDN_DOMAIN=$(aws cloudformation describe-stacks --stack-name $STACK_NAME --query "Stacks[*].Outputs[3].OutputValue" --output text)
  ```

- Upload Files to the bucket

  - [`cp`](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/s3/cp.html)

  ```
  aws s3 cp ./web-files s3://$BUCKET_NAME/ --recursive
  ```

- Access the site via the CloudFront Domain Name.

  ```
  curl $CDN_DOMAIN
  ```

  ```bash
  #on mac
  open "http://$CDN_DOMAIN"
  #on linux
  xdg-open "http://$CDN_DOMAIN"
  ```

- Clean Up & Delete All Resources

  - [Empty S3 Bucket](https://docs.aws.amazon.com/AmazonS3/latest/userguide/empty-bucket.html)
  - [`delete-stack`](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/cloudformation/delete-stack.html)

  ```bash
  #empty s3 bucket
  aws s3 rm s3://$BUCKET_NAME --recursive
  ```

  ```bash
  # delete-stack
  aws cloudformation delete-stack --stack-name $STACK_NAME
  ```

</details>
<br>

# Recources <a id='res'></a>
* [Amazon Cloudfront] (https://www.amazonaws.cn/en/cloudfront/)
* [Amazon S3] (https://aws.amazon.com/s3/)
* [Amazon CloudFront resource type reference] (https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/AWS_CloudFront.html)
* [Amazon Simple Storage Service resource type reference] (https://docs.amazonaws.cn/en_us/AWSCloudFormation/latest/UserGuide/AWS_S3.html)
* [Using the AWS CloudFormation console] (https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-using-console.html)