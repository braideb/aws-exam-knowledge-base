---
title: "Multi-user secrets rotation for Amazon RDS"
source: "https://aws.amazon.com/blogs/database/multi-user-secrets-rotation-for-amazon-rds/"
author:
  - "[[Hammad Ausaf\"]]"
  - "[[\"Gowrish Natarajan]]"
published: 2022-04-22
created: 2026-10-02
description: "Most database deployments have multiple database users who have varying degree of privileges on the data stored in the database, database structure, and administrative operations. In multi-user database environments, it’s important to grant and limit the privileges of different users based on their roles and needs. It’s also a best practice to limit the lifespan […]"
tags:
  - "clippings"
---
Most database deployments have multiple database users who have varying degree of privileges on the data stored in the database, database structure, and administrative operations. In multi-user database environments, it’s important to grant and limit the privileges of different users based on their roles and needs. It’s also a best practice to limit the lifespan of a user’s credentials, passwords in particular, to enhance your security posture. In this post, we show you two ways of using the [AWS Secrets Manager](https://aws.amazon.com/secrets-manager/) rotation function [templates](https://docs.aws.amazon.com/secretsmanager/latest/userguide/reference_available-rotation-templates.html) to create a multi-user rotation strategy on an [Amazon Relational Database Service (Amazon RDS)](https://aws.amazon.com/rds) without needing to write custom code.

First, we show how to set up multi-user password rotation using AWS Secrets Manager and an AWS provided [AWS Lambda](https://aws.amazon.com/lambda/) function. Then, we show how you can achieve multi-user password rotation using an AWS provided rotation function with [AWS AppConfig](https://docs.aws.amazon.com/appconfig/latest/userguide/what-is-appconfig.html) pointing to the active credentials in Secrets Manager. This architecture works with any Amazon RDS engine that can work with AWS provided password rotation functions.

## Problem context

Secrets Manager helps you protect user credentials, API keys, and other secrets that are needed to access your applications, services, and IT resources. The service enables you to easily [rotate, manage, and retrieve secrets](https://aws.amazon.com/blogs/security/rotate-amazon-rds-database-credentials-automatically-with-aws-secrets-manager/) throughout their lifecycle. Secrets Manager also offers deployable Lambda functions that can perform [secrets rotation for database users](https://aws.amazon.com/blogs/security/how-to-securely-provide-database-credentials-to-lambda-functions-by-using-aws-secrets-manager/). These templates are provided in the [AWS CloudFormation User Guide](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-secretsmanager-rotationschedule-hostedrotationlambda.html) and use [AWS CloudFormation](https://aws.amazon.com/cloudformation/) to create a stack of preconfigured resources. For example, you can download these Amazon RDS for MySQL [single user rotation](https://serverlessrepo.aws.amazon.com/applications/us-east-1/297356227824/SecretsManagerRDSMySQLRotationSingleUser) and [multi-user rotation](https://serverlessrepo.aws.amazon.com/applications/us-east-1/297356227824/SecretsManagerRDSMySQLRotationMultiUser) templates, and you can search for any Amazon RDS single user and multi-user template by searching for “rotation” on the AWS Serverless Application Repository home page and selecting the check box to show apps that create [AWS Identity and Access Management (IAM)](https://aws.amazon.com/iam) roles and resource policies. Resources created by the CloudFormation stack include a Lambda function and an IAM role that Secrets Manager can assume to invoke the function for password rotation.

With single user secret rotation, there is potential for application downtime when applications try to access the database during the password rotation process (between the [setSecret and finishSecret steps](https://docs.aws.amazon.com/secretsmanager/latest/userguide/rotate-secrets_how.html)). For this reason, some customers prefer a multi-user or [alternating user rotation strategy](https://docs.aws.amazon.com/secretsmanager/latest/userguide/rotating-secrets_strategies.html#rotating-secrets-two-users). This strategy involves applications connecting to the database by alternating between two sets of users to prevent application downtime while the secret is being rotated.

Secrets Manager provides single user and multi-user rotation Lambda functions. When the multi-user rotation function first runs, it creates a second user with a prefix of “\_clone” with the same database permissions as the first user.

## Using Secrets Manager provided password rotation functions

In this section, we look at how to set up single and multi-user (alternate user) rotation with Secrets Manager.

### Prerequisites

- AWS account
- Amazon RDS instance setup with multiple users provisioned
- Access to AWS IAM
- Access to AWS Lambda, Amazon RDS
- Access to AWS AppConfig
- Access to [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/)
- [AWS CLI](https://docs.aws.amazon.com/cli/index.html) access

### Single user AWS provided password rotation template

Once your Amazon RDS instance has been created, you can [create a secret](https://docs.aws.amazon.com/secretsmanager/latest/userguide/manage_create-basic-secret.html) and associate it with that RDS instance. When you create a new secret, you have the option to enable password rotation, as shown in Figure 1.

[![Figure 1 – Secrets Manager provided single user password rotation Lambda function](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2022/04/18/DBBLOG-1848-image001.png)](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2022/04/18/DBBLOG-1848-image001.png)

Figure 1 – Secrets Manager provided single user password rotation Lambda function

Selecting **No** for the option **Use separate credentials to rotate this secret** results in the creation of a single user password rotation template. The secrets rotation Lambda function logs in to the database using the username and password provided in the secret string. Once logged in, the function changes the password of that user. Following is the sample format for the secret value string for your secret.

```bash
{
  "engine": "mssql",
  "host": "<required: instance host name/resolvable DNS name>",
  "username": "<required: username>",
  "password": "<required: password>",
  "dbname": "<optional: database name>",
  "port": "<optional: TCP port number>"
}
```

### Multi-user AWS provided password rotation template

If you select the option **Yes** under **Use separate credentials to rotate the secret**, a multi-user password rotation function is created. This function uses the selected admin secret, shown in Figure 2, to rotate the password for the specified user.

Figure 2 – Secrets Manager provided Multi-user password Rotation Lambda function

Following is the sample format for the secret string when you enable Secrets Manager’s provided multi-user password rotation. This Lambda function on the first run uses the “masterarn” secret to generate a “\_clone” user and grant that user the same permissions as the original user.

```bash
{ 
   "engine": "mssql", 
   "host": "<required: instance host name/resolvable DNS name>",
   "username": "<required: username>",
   "password": "<required: password>", 
   "dbname": "<optional: database name>", 
   "port": "<optional: TCP port number>", 
   "masterarn": "<required: the ARN of the master secret used to create 2nd user and change passwords>"
}
```

## Password rotation functions

You can also write your [own](https://docs.aws.amazon.com/secretsmanager/latest/userguide/rotate-secrets_how.html) Lambda function to rotate the password. If a secret was created without password rotation, you can enable password rotation by using either the Secrets Manager provided password rotation function or by bringing in your custom rotation Lambda function.

## Multi-user password rotation with AWS Secrets Manager and AWS AppConfig

The next approach uses two existing user credentials stored as two separate secrets. Both these secrets use Secrets Manager’s provided single user Lambda function to rotate the passwords on independent schedules to provide an additional safety net during password rotation.

[AWS AppConfig](https://docs.aws.amazon.com/appconfig/latest/userguide/what-is-appconfig.html) is a feature of [AWS Systems Manager](https://aws.amazon.com/ec2/systems-manager) that can create, manage, and deploy application configurations. AWS AppConfig is used with applications hosted on [Amazon Elastic Compute Cloud (Amazon EC2)](https://aws.amazon.com/ec2) instances, AWS Lambda, containers, mobile applications, or IoT devices. AWS AppConfig simplifies the administration of applications at scale by deploying configuration changes from a central location.

In the architecture illustrated in Figure 3, we show a workflow where applications alternate between the two database user credentials every 60 days. Each secret is set on its independent schedule (the second secret is set up for rotation 30 days after the first secret is rotated) while ensuring that they are each rotated every 60 days. For example, if the first secret is set up for rotation on day T0, with a rotation interval of 60 days, the second secret would be enabled for rotation on T0+30 <sup>th</sup> day also with a rotation interval of 60 days. The recently rotated secret is the “active secret” and is updated in AWS AppConfig. The “active” secret is pulled by the application using AWS AppConfig API calls. This mechanism enables setting up secrets’ rotation every 30 days, alternating between the two independent secrets.

Figure 3 – Multi-user password rotation using two single secrets via AWS AppConfig

Figure 3 provides the event workflow of how multi-user secret rotation works after you’ve configured it using the following steps:

1. Configure appuser1 (first database user credentials) in the database. In Secrets Manager, set up an appuser1 secret with the same credentials as provided in the database.
2. In Secrets Manager, enable rotation for the appuser1 secret and select the **Create a new Lambda function to perform rotation** option to set up the single user secret rotation and set a rotation schedule of 60 days (as shown in Figure 3). Selecting this option initiates the creation of a single user secret rotation Lambda function through a CloudFormation stack deployment. Note that when changing the rotation configuration, Secrets Manager can immediately rotate the credentials in the secret once to validate the new settings. Ensure that all of your applications that use these credentials are updated to retrieve the credentials from this secret using Secrets Manager. At this point, the appuser1 secret can be actively used by applications and is designated as the “active” secret.
3. Follow step 1 for appuser2 (the second database user credentials being the “inactive” secret). Password rotation for this second database user is set up by a custom Lambda function that is initiated via an [Amazon EventBridge](https://aws.amazon.com/eventbridge/) rule, 30 days after setting up the first user. This step is shown in Figure 4.
	Figure 4 – An EventBridge rule to initiate a custom Lambda function after 30 days, to enable rotation for a second user
4. Create a new custom Lambda function by using the following code snippet. This is the target for the EventBridge rule you created in the previous step. The Lambda function runs for 30 days after you set it up. It enables rotation on the second secret using the AWS provided single user Lambda rotation function (that was created by the CloudFormation stack in step 2). You can disable this EventBridge rule after its first run. Alternately, you can also enable the password rotation of the second secret (appuser2) manually 30 days after the first database user rotation is set up. To do this, you use [schedule expressions in Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/rotate-secrets_schedule.html) by setting a specific value for day-of-month, month, and year, skipping the automation step below as shown in [How to configure rotation windows for secrets stored in AWS Secrets Manager](https://aws.amazon.com/blogs/security/how-to-configure-rotation-windows-for-secrets-stored-in-aws-secrets-manager/).
	```python
	import json
	import boto3
	import os
	from botocore.exceptions import ClientError
	def lambda_handler(event, context):
	    session = boto3.session.Session()
	    region_name = os.getenv('REGION_NAME')
	    secret_name = os.getenv('SECRET_NAME')
	    rotation_lambda_arn = os.getenv('ROTATION_LAMBDA_FN_ARN')
	    client = session.client(
	        service_name='secretsmanager',
	        region_name=region_name,
	    )
	    try:
	        response = client.describe_secret(
	            SecretId=secret_name
	        )
	        if response['RotationEnabled'] == True:
	            return {
	                'statusCode': 200,
	                'body': json.dumps('Second user rotation already set up')
	            }
	        if response['RotationEnabled'] == False:
	            response = client.rotate_secret(
	                SecretId=secret_name,
	                RotationLambdaARN=rotation_lambda_arn,
	                RotationRules={
	                    “ScheduleExpression”: “rate(60
	                    days)”
	                }
	            )
	        return {
	            'statusCode': 200,
	            'body': json.dumps('Second user rotated after 30 days')
	        }
	    except ClientError as e:
	        if e.response['Error']['Code'] == 'ResourceNotFoundException':
	            print("The requested secret " + secret_name + " was not found")
	        elif e.response['Error']['Code'] == 'InvalidRequestException':
	            print("The request was invalid due to:", e)
	        elif e.response['Error']['Code'] == 'InvalidParameterException':
	            print("The request had invalid params:", e)
	        elif e.response['Error']['Code'] == 'DecryptionFailure':
	            print("The requested secret can't be decrypted using the provided KMS key:", e)
	        elif e.response['Error']['Code'] == 'InternalServiceError':
	            print("An error occurred on the service side:", e)
	        return {
	            'statusCode': 500,
	            'body': json.dumps('Unexpected Errors')
	        }
	```
5. 30 days after the first user rotation, the custom Lambda function (created in step 4) initiates and sets up rotation for the second user (with a rotation schedule of 60 days). This also rotates the inactive secret (appuser2’s password). When this rotation is complete, an [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/) event with the event name of “RotationSucceeded” initiates a Lambda function (created in step 4).

The Lambda function updates AWS AppConfig, with the newly rotated appuser2 secret being set as the new “active” secret. A sample event is shown in the following code snippet, and a sample Lambda function that processes this event is shown in the code snippet in step 7.

You can configure an application to use AWS AppConfig with one or more environments, as applicable. [This workshop](https://mng.workshop.aws/appconfig/create-application.html) provides a detailed overview of the steps involved in setting up an application, environment, configuration profile, and hosted configuration on AppConfig. Set up the ARN for the active secret (now appuser1) in the AWS AppConfig configuration profile before invoking the Lambda that was created in step 4.

```python
{
    "version": "0",
    "id": "dd5fd657-7361-f5d2-0c0f-0fe27390611b",
    "detail-type": "AWS Service Event via CloudTrail",
    "source": "aws.secretsmanager",
    "account": "accountid",
    "time": "2021-08-13T19:17:16Z",
    "region": "us-west-2",
    "resources": [],
    "detail": {
        "eventVersion": "1.08",
        "userIdentity": {
            "accountId": "accountid",
            "invokedBy": "secretsmanager.amazonaws.com"
        },
        "eventTime": "2021-08-13T19:17:16Z",
        "eventSource": "secretsmanager.amazonaws.com",
        "eventName": "RotationSucceeded",
        "awsRegion": "us-west-2",
        "sourceIPAddress": "secretsmanager.amazonaws.com",
        "userAgent": "secretsmanager.amazonaws.com",
        "requestParameters": null,
        "responseElements": null,
        "additionalEventData": {
            "SecretId": "arn:aws:secretsmanager:us-west-2:accountid:secret:Appuser-pyodbc-WoDpmU"
        },
        "requestID": "Rotation-arn:aws:secretsmanager:us-west-2:accountid:secret:Appuser-pyodbc-WoDpmU-7ac47c8c-f328-423b-9698-6926dd1f293e",
        "eventID": "638c6d2d-e098-4100-a3e0-a4b7b87e5b92",
        "readOnly": false,
        "eventType": "AwsServiceEvent",
        "managementEvent": true,
        "recipientAccountId": "accountid",
        "eventCategory": "Management"
    }
}
```

6. Applications use the configuration stored in the AWS AppConfig configuration profile to fetch the active database user from Secrets Manager. A sample AWS AppConfig configuration profile is shown following.
	```bash
	{  SECRETS_MANAGER_ACTIVE_USER=arn:aws:secretsmanager:us-west-2:account-id:secret:AppUser2_Secret-OZ7qCX
	}
	```
7. During the rotation cycle, appuser1’s secret (the current inactive secret) is rotated. The rotation completion event triggers a CloudWatch event (“rotationSucceeded”) which causes the Lambda function to update AWS AppConfig with appuser1’s secret ARN set as the new “active” secret. The secrets rotation cycle repeats every 30 days, alternating between the two independent secrets.

Following is a sample Lambda function that updates an AWS AppConfig configuration profile after a successful rotation event from Secrets Manager. Each application can have multiple deployment environments that point to different configuration profile versions, which in turn point to the active Secrets Manager secret. If you want to control the rollout of your passwords across the clients, refer to the AWS AppConfig [deployment strategy](https://docs.aws.amazon.com/appconfig/latest/userguide/appconfig-creating-deployment-strategy.html).

```python
import json
import boto3
import os

def lambda_handler(event, context):
    # Enviroment variables
    # REGION_NAME = Region where your application configuration is deployed
    # APPUSER1/2 = Database usernames that will be the AWS Secrets Manager secret value
    # APPLICATION_NAME = AWS AppConfig application name
    # APPCONFIG_PROFILE_NAME = AWS AppConfig configuration profile name that stores the AWS Secrets Manager ARN
    # APPCONFIG_STRING_HEADER = SECRETS_MANAGER_ACTIVE_USER JSON key for the AWS secret ARN

    region_name = os.getenv('REGION_NAME')
    appuser1 = os.getenv('APPUSER1')
    appuser2 = os.getenv('APPUSER2')
    appName = os.getenv('APPLICATION_NAME')
    configProfileName = os.getenv('APPCONFIG_PROFILE_NAME')
    appConfigStringHeader = os.getenv('APPCONFIG_STRING_HEADER')
    content_type = "json"

    # Fetch secret ID from the event
    secretId = event['detail']['additionalEventData']['SecretId']

    # Create a Secrets Manager client
    session = boto3.session.Session()
    secret_client = session.client('secretsmanager', region_name)
    appconfig_client = session.client('appconfig')

    # get secret value
    try:
        secretVal = secret_client.get_secret_value(
            SecretId=secretId
        )
        secretString = secretVal['SecretString']
    except:
        string = str(e)
        print("Unexpected error while executing secretmanager::get_secret_value function!", string)
        return {
          'statusCode': 500,
          'body': json.dumps('Unexpected Errors')
        }

    if (appuser1 in secretString or appuser2 in secretString):
        content = appConfigStringHeader + secretId
    else:
        return {
            'statusCode': 200,
            'body': json.dumps('Rotation Succeeded Lambda fired for some other secret!')
        }

    # List all applications in AppConfig and get the appId associated with our #configuration in AppConfig
    applist_response = appconfig_client.list_applications()
    appList = applist_response['Items']
    app = next(item for item in appList if item["Name"] == appName)
    appId = app['Id']

    # List all config profiles in AppConfig and get the configProfileId #associated with our configuration in AppConfig
    confProfilelist_response = appconfig_client.list_configuration_profiles(
        ApplicationId=appId
    )
    confProfileList = confProfilelist_response['Items']
    confProfile = next(item for item in confProfileList if item["Name"] == configProfileName)
    configProfileId = confProfile['Id']

    ## The below API call will return a response like this.
    # {'ApplicationId': 'string','ConfigurationProfileId': #'string','VersionNumber': 123,'Description': 'string','Content': #StreamingBody(),'ContentType': 'string'}
    try:
        appconfig_client.create_hosted_configuration_version(
            ApplicationId=appId,
            ConfigurationProfileId=configProfileId,
            Content=content,
            ContentType=content_type
        )
        return {
            'statusCode': 200,
            'body': json.dumps('Rotation Succeeded Lambda fired with appuser1 or appuser2!')
        }
    except:
        string = str(e)
        print("Unexpected error while executing appconfig::create_hosted_configuration_version function :", string)
        return {
          'statusCode': 500,
          'body': json.dumps('Unexpected Errors')
        }
```

If your application is running in Lambda, then you can use the [AWS AppConfig Lambda extension](https://docs.aws.amazon.com/appconfig/latest/userguide/appconfig-integration-lambda-extensions.html) to automatically pull the latest application configuration, depending on your environment variable settings for `AWS_APPCONFIG_EXTENSION_POLL_INTERVAL_SECONDS`.

## Comparing the two multi-user secrets rotation strategies

Now that we have looked at the two multi-user password rotation strategies, let’s look at the pros and cons of each.

### AWS provided multi-user secrets rotation function, which creates a second “\_clone” database user

**Pros**:

1. Deploys Secrets Manager’s provided multi-user rotation Lambda function, reducing the complexity of multi-user secrets rotation; no hassle of actively maintaining custom code.
2. The provided multi-user password rotation function syncs up permissions between the two database users (the original and the “\_cloned” user).

**Cons**:

1. In some cases, you might have already created two database users that you want to rotate and alternate between. This is not possible when using the Secrets Manager provided multi-user Lambda function.
2. There is no convenient way to proactively notify applications of rotated user credentials. If applications are caching secrets, they require custom retry logic to fetch the current version after a secret has been rotated.

### Multi-user secrets rotation using two existing database users and the single user secrets rotation function on an independent schedule

**Pros**:

1. Works with existing database users instead of letting the password rotation Lambda function create a new “\_cloned” user.
2. There is a clear separation of control between managing secrets and fetching secrets. With AWS AppConfig managing the active user secret through config version updates, applications can pull a single configuration item pointing to the “active” secret.
3. Even though applications query AWS AppConfig for newly rotated active user secrets after rotation, applications can continue to use the “passive” secret to connect to the database, because that password is not rotated for another 30 days.
4. Because both secrets are maintained independently, it’s possible to look at the previous (AWSPREVIOUS version) passwords of both the database users.

**Cons**:

1. Applications need to poll AWS AppConfig for config updates; as of this writing there is no notification or push mechanism for applications to be notified of config updates. Applications should poll for the active Secrets Manager ARN from AWS AppConfig if they get an “invalid username/password” exception.
2. Permissions between the two database users have to be synchronized outside of the Secrets Manager provided password rotation function.

## Cleanup

Secrets created in AWS secrets Manager can be deleted either via the AWS console or AWS CLI following the steps outlined [here](https://docs.aws.amazon.com/secretsmanager/latest/userguide/manage_delete-secret.html). Lambda functions can also be deleted using the [AWS CLI](https://docs.aws.amazon.com/cli/latest/reference/lambda/delete-function.html) or [AWS console](https://docs.aws.amazon.com/lambda/latest/dg/getting-started-create-function.html). Resources created as part of AWS AppConfig can be deleted by following the instructions [here](https://mng.workshop.aws/appconfig/cleanup.html).

## Conclusion

In this post, we showed you two ways of using Secrets Manager’s provided password rotation functions for a multi-user rotation. Both these approaches make it unnecessary for you to write a custom password rotation Lambda function. Using AWS AppConfig as a front end to Secrets Manager gives you the ability to toggle the two independent secrets working on their own rotation cycle. The onus is on applications to determine how often they need to poll AWS AppConfig to retrieve the latest configuration version based on their needs.

---

### About the Authors

 **[![](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2022/04/18/Hammad-Ausaf.jpg)](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2022/04/18/Hammad-Ausaf.jpg) Hammad Ausaf** is a Senior Solutions Architect in the Media & Entertainment (M&E) space. He is a passionate builder and strives to provide the best solutions to AWS customers.

 **[![](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2022/04/18/Gowrish-Natarajan.jpg)](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2022/04/18/Gowrish-Natarajan.jpg) Gowrish Natarajan** is a Senior Solutions Architect at AWS in the Media & Entertainment(M&E) space. He specializes in building fault tolerant, low latency, highly available streaming solutions for Media customers globally.