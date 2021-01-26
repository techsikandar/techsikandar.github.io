---
layout: post
title:  "Creating and automating a serverless profile application with AWS SAM, AWS CI/CD & Angular"
date:   2019-05-29 21:13:27 -0500
categories: aws serverless
---

<h1>{{ "Introduction" }}</h1>

![sam](/assets/aws/serverless/sam.png)

In this blog, i am creating a serverless application with AWS Lambda & API Gateway using SAM. The API will return the personal profile data as JSON which will be consumed by an angular frontend to develop a single page application and host that on S3.

<h1>{{ "Intended Audience" }}</h1>

<ul>
<li>You know AWS Lambda, API Gateway, AWS SAM and now want to know how to develop an end-to-end application.</li>
<li>You are interested in automating the AWS SAM build & deployment.</li>
<li>You are interested in developing your personal profile website.</li>
</ul>

<h1>{{ "What is AWS SAM?" }}</h1>

The AWS Serverless Application Model (SAM) is an open-source framework for building serverless applications. It’s a shorthand way to write lambda functions, APIs, databases, and event source mappings. AWS SAM is an extension of CloudFormation. So we can use the full suite of resources, intrinsic functions, and other template features that are available in AWS CloudFormation.

<h1>{{ "Initializing & setting up the SAM project" }}</h1>

Initializing the SAM project is straightforward. We just have to choose the project runtime and it’s name. For this blog, I am using Python and project name is profile-serverless. Create the project workspace. For example, `<directory>/profile-serverless/`.  Change to `<directory>/profile-serverless/` and then execute this command:

{% highlight ruby %}
<directory>/profile-serverless> sam init --runtime python3.8 --name profile-serverless
{% endhighlight %}

Initializes a serverless application with an AWS SAM template. The template provides a folder structure for your Lambda functions, and is connected to an event source such as APIs, S3 buckets, or DynamoDB tables. This application includes everything we need to get started and to eventually extend it into a production-scale application. You can open this project in any editor of your choice. I used VS Code.

<h1>{{ "Create AWS serverless profile project" }}</h1>

<b>Configure your profile data</b><br>
Configure the `profile.json` based upon your own personal profile. Refer the simple profile below:

{% highlight ruby %}
{
  "statistics": {
    "Years": "15+",
    "Projects": "20",
    "Certifications": "10+"
  },
  "personal": {
    "name": "Khan, Malik Sikandar",
    "title": "Experienced Software Consultant"
  },
  "technicalSkills": {
    "Programming Languages": [
      "Java",
      "Python",
      "Go"
    ]
  },
  "certifications": [
    {
      "certification": "AWS Certified Developer Associate"
    }
  ],
  "education": [
    {
      "university": "National University of Singapore"
    }
  ],
  "projects": [
    {
      "projectName": "Some Project"
    }
  ]
}
{% endhighlight %}

Store this `profile.json` in `<directory>/profile-serverless/profile_loader`.

<b>Profile loader Lambda function</b>

Create `profileloader.py` in `<directory>/profile-serverless/profile_loader`. This simple python program will basically do two simple things. One, read the profile, second, return it as the response.

{% highlight ruby %}
import json

with open('profile.json') as f:
  data = json.load(f)

def lambda_handler(event, context):
    return {
        'statusCode': 200,
        'headers': {
          "Access-Control-Allow-Origin": "",
           "Access-Control-Allow-Credentials": "true"
        },
        'body': json.dumps(
          {
            'data': data
          }
        )
    }
{% endhighlight %}

You may also want to add `requirements.txt` in the same directory and include `requests` & modules.

<b>Profile loader API</b>

Look for `template.yaml` in project directory. This file is created by AWS SAM init command. It comes with Hello World function & API. We will remove or rename all occurrences of these and replace them with “ProfileLoaderFunction” & “ProfileLoaderApi”. `template.yaml` will look something like this after that.

{% highlight ruby %}
Resources:
  ProfileLoaderFunction:
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: ProfileLoaderFunction
      CodeUri: profile_loader
      Handler: profileloader.lambda_handler
      Runtime: python3.7
      Events:
        ProfileLoaderApi:
          Type: Api
          Properties:
            Path: /getProfile
            Method: get
{% endhighlight %}

<h1>{{ "Test Profile loader function & API locally" }}</h1>

Build the project

{% highlight ruby %}
sam build --use-container
{% endhighlight %}

Test the profile loader function locally

{% highlight ruby %}
sam local invoke ProfileLoaderFunction --no-event
{% endhighlight %}

Test the profile loader Api locally

{% highlight ruby %}
profile-serverless$ sam local start-api profile-serverless$ curl http://localhost:3000/
{% endhighlight %}

Note: You need docker installed locally to test this.

<h1>{{ "Packaging & Deployment" }}</h1>

{% highlight ruby %}
sam package --template-file template.yaml --output-template-file output-template.yaml --s3-bucket <S3_BUCKET>
{% endhighlight %}

Mention your S3 bucket name above.

{% highlight ruby %}
sam deploy --template-file output-template.yaml --stack-name ProfileLoader --capabilities CAPABILITY_IAM
{% endhighlight %}

Continue to read if you want to automate this application with AWS CI/CD.

![cicd](/assets/aws/serverless/cicd.png)

Before we proceed, copy the `profile-serverless` project to GitHub or CodeCommit. I am using GitHub.

<h1>{{ "Setup CI/CD for serverless profile application with AWS Pipeline" }}</h1>

<b>Create AWS Pipeline</b>

Create a new folder `pipeline` inside `<directory>/profile-serverless` and run this command:

{% highlight ruby %}
<directory>/profile-serverless/pipeline> sam init --location gh:aws-samples/cookiecutter-aws-sam-pipeline
{% endhighlight %}

Choose AWS Pipeline project name: profile-serverless-pipeline
Choose your GitHub / CodeCommit repository

It will then create a set of necessary artifacts required for this automation. Now move the `buildspec.yaml` from pipeline folder to the root of the project. It should look like this:

{% highlight ruby %}
<directory>/profile-serverless/buildspec.yaml
{% endhighlight %}

<b>Configuration</b>

AWS Pipeline (actually AWS CodeBuild) needs access to the GitHub repository so that it can build the project. It needs to know the username, repository and the access token. We are going to configure these properties in AWS Systems Manager Parameter Store. For more information, Refer to https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html

{% highlight ruby %}
aws ssm put-parameter --name "/service/profile-serverless-pipeline/github/user" \
    --description "Github Username for Cloudformation Stack profile-serverless-pipeline" \
    --type "String" --value "<YOUR_USER_NAME>"
{% endhighlight %}

{% highlight ruby %}
aws ssm put-parameter --name "/service/profile-serverless-pipeline/github/repo" \
    --description "Github Repository name for Cloudformation Stack profile-serverless-pipeline" \
    --type "String" --value "profile-serverless"
{% endhighlight %}

{% highlight ruby %}
aws ssm put-parameter --name "/service/profile-serverless-pipeline/github/token" \
    --description "Github Token for Cloudformation Stack profile-serverless-pipeline-pipeline" \
    --type "String" --value "<GITHuB_TOKEN>"
{% endhighlight %}

Refer to https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line to find out how to get the developer access token.

<h1>{{ "Create CloudFormation stack" }}</h1>

{% highlight ruby %}

aws cloudformation create-stack --stack-name profile-serverless-pipeline \
    --template-body file://pipeline/pipeline.yaml \
    --capabilities CAPABILITY_NAMED_IAM

{% endhighlight %}

`pipeline.yaml` contains the necessary services and roles required to create the stack. It’s a one time job. This also has a manual approval phase. So, if your build process is taking too long, it’s probably waiting for the approval. In this case, go to Code Pipeline console and approve the step. Plus, you may want to delete the unnecessary environments. You can always refer to the working source code here at GitHub.

<h1>{{ "Test the Pipeline" }}</h1>

Make changes to the `profile-serverless` project and do git commit & git push. This will trigger the build process. Verify your changes by hitting the API Gateway endpoint.

Continue to read if you want to host your personal profile on S3 bucket with Angular front-end.