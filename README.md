# AWS Lex-Lambda-Bedrock use-case Demo
Arjun
Demonstration of an interactive Amazon Lex chatbot that generates images with Amazon Bedrock using Lambda codehooks.

![Diagram](diagrams/archie_diagram.png)

### What resources will be created?
This CDK code will create the following:
   - 1 Lex bot
   - 1 S3 bucket to store the generated image
   - 1 Lambda (to invoke the Bedrock API and subsequently the Foundation Model provided by Stability Diffusion to generate the image)
   - 2 IAM roles (one for the lex bot to call Lambda, one for the Lambda to call Bedrock and S3)

## Requirements

### Development Environment
**Cloud 9**

This demonstration is executed in AWS Cloud9 environment. The EC2 instance used is m5.large (8 GiB RAM + 2 vCPU).

### AWS setup
**Region**

If you have not yet run `aws configure` and set a default region, you must do so, or you can also run `export AWS_DEFAULT_REGION=<your-region>`. The region used in the demonstration is us-east-1 as Bedrock service is available only in limited regions.

**Authorization**

You must use a role that has sufficient permissions to create Iam roles, as well as cloudformation resources

#### Python >=3.8
Make sure you have [python3](https://www.python.org/downloads/) installed at a version >=3.8.x in the CDK environment. The demonstration has used python version 3.8 and a layer has been attached.
The layer used in this demonstration has Boto3>=1.28.57 (for Bedrock service), Pillow==9.5.0 (for image conversion), and pyshorteners (for generating tiny urls).

#### AWS CDK
Make sure you have the [AWS CDK](https://docs.aws.amazon.com/cdk/v2/guide/getting_started.html#getting_started_install) installed in the Cloud9 environment.


## Setup

### Set up environment and gather packages

```
cd lex-lambda-genai-imagegen-chatbot
```

Install the required dependencies (aws-cdk-lib and constructs) into your Python environment 
```
pip install -r requirements.txt
```

### Gather and deploy resources with the CDK

First synthesize, which executes the application, defines which resources will be created, and translates this into a cloudformation template
```
cdk synth
```
Now bootstrap, which provisions the resources you'll use when deploying the application
```
cdk bootstrap
```
and deploy with
```
cdk deploy LexGenAIBotStack
```

The deployment will create a Lex bot, a Lambda and a S3 bucket.

## Usage
Once all the resources are created after `cdk deploy` finishes, go to AWS Management Console and search for Amazon Lex service. 
If the deployment is successful, a Lex bot named `LexGenAIBot` should be seen in the Bots home page.

![Diagram](diagrams/LexBotHomePage.png)

### Things to make sure in Lex console

- Click on Bot `LexGenAIBot` and verify that three intents `Welcome Intent`, `GenerateImageIntent`, and `FallbackIntent` are present as per the below screenshot. 
![Diagram](diagrams/LexIntentsPage.png)


- Click on `WelcomeIntent` and scroll down to `Sample utterances` to ensure sample utterances are created. 
![Diagram](diagrams/WelcomeIntentSampleUtterances.png)


- Scroll down to find the `Closing response` section and expand the `Message group` dropdown. Ensure that a closing message is present. 
![Diagram](diagrams/WelcomeIntentClosingResponse.png)


- If all the above steps are in place, click on `GenerateImageIntent` available on the list of Intents (seen left-hand side).


- Once you are in the `GenerateImageIntent` page, scroll down to `Sample utterances` to ensure the utterances are created.
![Diagram](diagrams/GenerateImageIntentSampleUtterances.png)


- Scroll down to find the `Fulfilment` section and click on the `Advanced options` button. Verify that `Fulfilment Lambda code hook option` is checked.
![Diagram](diagrams/GenerateImageIntentLambdaHook.png)


- Once the above steps are verified, go back ot the `Intents` page and under the `Deployment section`, click on `Aliases`.
![Diagram](diagrams/Aliases.png)


- Click on `TestBotAlias` and scroll down to `Languages` section to find the language that used in the deployment, i.e., `English (US)`. Click on `English (US)`.


- The opened page shows the Lambda function for the Bot. Ensure the source Lambda and version or alias is properly set, as per the resource created from the CDK deployment.
![Diagram](diagrams/AliasLambdaFunction.png)


- Once all the above steps are verified, go back to the `LexGenAIBot` `All languages` section. Click on `Build` and you are ready to `Test` the bot post successful build.
![Diagram](diagrams/Build.png)


- Bot build in progress
![Diagram](diagrams/Build.png)
![Diagram](diagrams/Build-in-progress-I.png)
![Diagram](diagrams/Build-in-progress-II.png)


- Once the Bot is built successfully, we're ready to test the `LexGenAIBot` bot. Click on the `Test` button
![Diagram](diagrams/BuildComplete.png)


- The Test console opens up.
![Diagram](diagrams/TestConsole.png)


- Enter some sample queries and get the image generated.
![Diagram](diagrams/SampleTestCase.png)


- The image would be generated and available in S3 bucket. Additionally, a S3 pre-signed url has been generated and shortened for easy consumption. 
- Copy & paste the tiny url in a browser and the image is downloaded. 

### Tips for best results

**Keep your lambda perpetually warm by provisioning an instance for the runtime lambda**

Go to Lambda console > select the function LexGenAIDemoFilesStack-LexGenAIBotLambda*

Versions > Publish new version

Under this version 
   - Provisioned Concurrency > set value to 1
   - Permissions > Resource based policy statements > Add Permissions > AWS Service > Other, your-policy-name, lexv2.amazonaws.com, your-lex-bot-arn, lamdba:InvokeFunction

Go to your Lex Bot (LexGenAIBot)

Aliases > your-alias > your-language > change lambda function version or alias > change to your-version

This will keep an instance running at all times and keep your lambda ready so that you won't have cold start latency. This will cost a bit extra (https://aws.amazon.com/lambda/pricing/) so use thoughtfully. 

## Clean Up

To clean up the resources created as part of this demonstration, run the command `cdk destroy` in the directory `lex-lambda-genai-imagegen-chatbot`.
