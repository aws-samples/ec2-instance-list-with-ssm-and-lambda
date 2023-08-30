<h2>Cloudformation templates for Automate insights for your EC2 fleets across AWS accounts and regions</h2>

<h3>Solution overview</h3>

The implementation of this automated solution has provided Delhivery with several key benefits, as outlined below.

1. It simplified the process of obtaining operating system details across their entire organization, enabling streamlined patch management and timely OS upgrades. This not only improved system security but also ensured compliance with end-of-life support for operating systems.
2. Identifying the number of instances that are not managed via Systems Manager, Helped Delhivery determine the required actions to make these instances managed via Systems Manager. Once managed, they can leverage the various capabilities of Systems Manager such as Patch Manager, Run Command, Session Manager, and more.
3. Delhivery created an organization-wide EC2 inventory, enabling them to upgrade old instances, manage spot instances, and utilize tag-based filters. The inventory improved operational efficiency and decision-making.
4. Map the End-of-Life (EOL) dates for different operating systems. This analysis provided delhivery with insights about the support timelines for each operating system, enabling them to plan and schedule timely upgrades accordingly.

You can use this solution to obtain a list of all your EC2 instance details within a few minutes, spread across multiple regions and accounts. This allows customers to conveniently find and verify the details of their instances.

This process involves a [AWS Lambda](https://aws.amazon.com/lambda/) function in the delegated account/ management account assuming a role named "**ssmLambdaRole**" in each account of an [AWS Organizations](https://aws.amazon.com/organizations/). The Lambda function executes the necessary AWS API calls, including **ec2:DescribeInstances**, **ec2:DescribeImages**, **ec2:DescribeRegions** and **ssm:DescribeInstanceInformation**, to gather the required details of the instances. Once the details are collected, the Lambda function uploads a CSV report to an S3 bucket. **Any member account can be designated as a delegated account in an organization**.
![](https://github.com/aws-samples/ec2-instance-list-with-ssm-and-lambda/blob/main/ssm-lambda.png)

<h3>This repository contains 3 cloudformation templates. Please use it as mentioned below</h3>

<h3>Step 1 : Create an AWS Identity and Access Management (IAM) role named LambdaSsmOsDetailRole and lambda function named LambdaSsmOsFunction in delegated account or management account for Lambda service.</h3>

Deploy the following CloudFormation template [Step1.yaml](https://github.com/aws-samples/ec2-instance-list-with-ssm-and-lambda/blob/main/step1.yaml). See [Creating a stack on the AWS CloudFormation console for more details](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html).

Download the [Step1.yaml](https://github.com/aws-samples/ec2-instance-list-with-ssm-and-lambda/blob/main/step1.yaml) CloudFormation template. This stack will create an IAM role “LambdaSsmOsDetailRole” and policy “LambdaSsmOsDetailPolicy" with the specified permissions to list AWS accounts in the organization, perform S3 PutObject operations, and assume the ssmLambdaRole role. Please make sure to specify the S3 bucket where you want to upload the final report.

If you have deployed the Step 1 template into the management account, then proceed to Step 3, otherwise continue to Step 2

<h3>Step 2: (Optional) Create IAM role for delegated account</h3>

Note: This step is optional and should be executed in case the report needs to be collected from an account of organization which is not management account.

In case you want to collect the report in management account of organization, then please proceed to Step 3.

Deploy the following CloudFormation template [Step2.yaml](https://github.com/aws-samples/ec2-instance-list-with-ssm-and-lambda/blob/main/step2.yaml). For details regarding the process to deploy CloudFormation template, please see, [Creating a stack on the AWS CloudFormation console](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html). Download the [Step2.yaml](https://github.com/aws-samples/ec2-instance-list-with-ssm-and-lambda/blob/main/step2.yaml) CloudFormation template.

The stack will create an IAM role “ManagementOrganizationRole” and policy “ManagementOrganizationPolicy" with the specified permissions to list AWS accounts in the organization. This role will be assumed by Lambda function from delegated account of next step that collects the data and generates report.

<h3>Step 3 : Create an IAM role named ssmLambdaRole in each account of organization for Lambda</h3>

Download the [Step3.yaml](https://github.com/aws-samples/ec2-instance-list-with-ssm-and-lambda/blob/main/step3.yaml) CloudFormation template. This CloudFormation template will create an IAM policy named "MyPolicy" and an IAM role named "MyRole" with the specified permissions and trust policy, respectively.  Please make sure to replace <Parent/Payer Account id> with the actual ID of the parent/payer account. Please note that the AWS::AccountId pseudo parameter is only available within the AWS CloudFormation service. If you're deploying the stack using an external tool or framework, you may need to find an equivalent method to retrieve and substitute the account ID.

Use the above cloud formation template for all the target account role creation to create a stack set with service-managed permissions. You can use stack sets to create the stack for your entire organization or specify the OUs that you want.

To create a stack set for your organization

1. Sign in to the AWS Management Console as the management account for your organization.
2. Open the AWS CloudFormation console.
3. If you haven't already, in the **Region selector**, choose the same AWS Region that you used in the previous procedure.
4. In the navigation pane, choose **StackSets**.
5. Choose **Create StackSet**.
6. On the **Choose a template** page, keep the default options for the following options:
	- For **Permissions**, keep **Service-managed permissions**.
	- For **Prerequisite - Prepare template**, keep **Template is ready**.
7. Under **Specify template**, choose **Upload a template file**, and then select **Choose file**.
8. Choose the file and then choose **Next**.
9. On the **Specify StackSet details** page,
	- Enter a stack name such as **LamdaSSMRole**,
	- Enter a description for parameter **SsmLambdaRoleName** specify the same name that you specified in [Step1.yaml](https://github.com/aws-samples/ec2-instance-list-with-ssm-and-lambda/blob/main/step1.yaml) for parameter **SsmLambdaRoleName**,
	- For parameter **DeploymentAccountID** specify the ID of the account where you deployed the [Step1.yaml](https://github.com/aws-samples/ec2-instance-list-with-ssm-and-lambda/blob/main/step1.yaml) and then choose **Next**.
10. On the **Configure StackSet options** page, keep the default options and then choose **Next**.
11. On the **Set deployment options** page, for **Add stacks to stack set**, keep the default **Deploy new stacks** option.
12. For **Deployment targets**, choose if you want to create the stack for the entire organization or specific OUs. If you choose an OU, enter the OU ID.
13. For **Specify regions**, enter only one of region because IAM is a global service so deploying into multiple regions will result in a failure
14. For **Deployment options**, for **Failure tolerance - optional**, enter the number of accounts where the stacks can fail before CloudFormation stops the operation. We recommend that you enter the number of accounts that you want to add, minus one. For example, if your specified OU has 10 member accounts, enter 9. This means that even if CloudFormation fails the operation 9 times, at least one account will succeed.
15. Choose **Next**.
16. On the **Review** page, review your options, and then choose **Submit**. You can check the status of your stack on the **Stack instances** tab.

After CloudFormation creates the stacks, each member account can sign in to the [Support Center Console](https://console.aws.amazon.com/) and find a role is created.

If you are using AWS Organizations to deploy stack-set, then you need will to deploy the same CloudFormation template as a simple stack in the same region to allow the creation of similar IAM role in the management account as well.

If you do not have an organization, you need to deploy the CloudFormation template in each account to gather details of EC2 instances from each account.

Since stack-set do not deploy the CloudFormation template in the management account, please deploy the step3.yaml CloudFormation template as normal stack. For details regarding the process to deploy CloudFormation template, please see [Creating a stack on the AWS CloudFormation console](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html).

<h3>Step 4: Configure test event for Lambda function</h3>

A Lambda function named **LambdaSsmOsFunction** is created in account where CloudFormation template from **Step 1** is deployed. Using this function, we will get the details of all the EC2 instances across different region and accounts, based on the test event details.

The format required for test event is
```
{
    "region": [],
    "accountids": [],
    "bucket": "bucket-name",
    "managementAccountId": ""
}
```
For **region**, you can mention the list of regions from you want to get the details. To get details from all region, please keep it as empty array.

For **accountids**, you can mention the list of accounts from you want to details.

For **bucket**, you need to mention the S3 bucket name where the report needs to be saved. The report will be saved as 'ssm/<YYYY-MM-DD>/instanceReport.csv' in S3 bucket. Please make sure to specify the same bucket, that is specified during the execution of CloudFormation template in **Step 1**.

Here are some examples which demonstrate how to get required details from specific account or region using Lambda function.

**Example 1** : To get details from all the account belonging to the organization from all regions, when executing the function from a sub-account, we need to pass blank value for accountids and region, and account id of management account under managementAccountId such as
```
{
  "region": ["ap-south-1","ap-southeast-1"], 
  "accountids": [],
  "bucket": "bucket-name",
  "managementAccountId": "9999999999"
}
```
**Example 2** : To get details from account 1111111111 and 2222222222 from Mumbai and Singapore region, when executing the function from management account, we can set following values in the event
```
{
  "region": ["ap-south-1","ap-southeast-1"], 
  "accountids": ["1111111111","2222222222"],
  "bucket": "bucket-name",
  "managementAccountId": ""
}
```
**Example 3** : To get details from account 1111111111 and 2222222222 from all the region, , when executing the function from management account, we can set following values in the event
```
{
  "region": [],  
  "accountids": ["1111111111","2222222222"],
  "bucket": "bucket-name"
  "managementAccountId": ""
}
```
**Example 4** : To get details from all the account belonging to the organization from Mumbai and Singapore region, when executing the function from management account, we need to pass blank value for accountids and managementAccountId such as
```
{
  "region": ["ap-south-1","ap-southeast-1"], 
  "accountids": [],
  "bucket": "bucket-name"
  "managementAccountId": ""
}
```
**Example 5** : To get details from all the account belonging to the organization from Mumbai and Singapore region , when executing the function from a sub account, we need to pass blank value for accountids and account id of management account under managementAccountId such as following
```
{
  "region": ["ap-south-1","ap-southeast-1"], 
  "accountids": [],
  "bucket": "bucket-name"
  "managementAccountId": "9999999999"
}
```
Save the test event as per requirement and execute the Lambda function. The repost will be stored in S3 bucket passed under test event as ssm/<YYYY-MM-DD>/instanceReport.csv

<h3>Step 5: (Optional) Set the run schedule for the Lambda function</h3>

You can configure this lambda function to run weekly or at your desired frequency using EventBridge and cron syntax. Refer to the below steps:

1. Open the AWS Management Console and go to the EventBridge service.
2. Click on "**Rules**" and select the rule that triggers your Lambda function or create a new rule.
3. Configure the rule with a name, description, and **state** (enabled).
4. Select "**Schedule**" as the rule type and use a cron expression for the desired interval.
5. Add your existing Lambda function as the target.
6. Configure the target with the necessary settings, including an empty JSON object as the input.
7. Assign the appropriate IAM role to the rule to ensure the Lambda function has the necessary permissions.
8. Save the rule, and your existing Lambda function will now run according to the specified cron expression.

<h3>Cleanup</h3>

In this blog, we created multiple resource via AWS CloudFormation. It is easy to clean up everything in the same way it was created.

In Step 3, AWS CloudFormation stack-set was created. To delete the resources belonging to CloudFormation stack set, please see [Delete a stack set using the AWS Management Console](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacksets-delete.html#stacksets-delete-set)

After the AWS CloudFormation stack set is cleaned up, you can proceed with the clean up of resources created as AWS CloudFormation stacks. Please delete the CloudFormation stacks in following sequence:
	1. Step 3
	2. Step 2
	3. And finally, Step 1.

For more details, please see [Deleting a stack on the AWS CloudFormation console](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-delete-stack.html).

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.

