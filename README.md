## Cloudformation templates for Cost-effectively gain insights into your EC2 fleets across your AWS Organization with AWS Systems Manager and AWS Lambda

This repository contains 3 cloudformation templates. Please use it as mentioned below

1. step1.yaml Cloudformaiton template creates an IAM role named LambdaSsmOsDetailRole and lambda function named LambdaSsmOsFunction in delegated account or management account for Lambda service.
Note : Delegated account can be any account of Organization which is not management account. 
If you are using Management account then ignore next step (Step 2). 
If you are using an account of Organization which is not Management/Payer account as Delegated account, then please make sure to complete Step 2 in next step

    Deploy the following CloudFormation template step1.yaml. For details regarding the process to deploy CloudFormation template, please see, [Creating a stack on the AWS CloudFormation console](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html)


2. step2.yaml is optional Cloudformaiton template. This template creates an IAM role named ManagementOrganizationRole in the management account of Organization. This is required in case report has to be collected from any other account of organization, i.e. when an account of Organization which is not Management/Payer account is referenced as delegated account.

    Deploy the following CloudFormation template step2.yaml in the management account. For details regarding the process to deploy
CloudFormation template, please see, [Creating a stack on the AWS CloudFormation console](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html)

3. step3.yaml creates an IAM role named ssmLambdaRole in each account of Organization for Lambda service to gather the required details via stack-set. This cloudformation template needs to be deployed in the management account as normal stack as stack-set do not target management account.

    Deploy the following CloudFormation template step3.yaml in the management account as normal stack. For details regarding the process to deploy CloudFormation template, please see, [Creating a stack on the AWS CloudFormation console](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html)

    Deploy the same CloudFormation template step3.yaml in the management account as StackSet. To create a stack set for your organization

	1. Sign in to the AWS  Management Console as the management account for your organization.
	2. Open the AWS  CloudFormation console at https://console.aws.amazon.com/cloudformation.
	3. If you haven't already,  in the Region selector, choose the same AWS Region that you  used in the previous procedure.
	4. In the navigation pane,  choose StackSets.
	5. Choose Create  StackSet.
	6. On the Choose a  template page, keep the default options for the following  options:
	    * For Permissions,  keep Service-managed permissions.
	    * For Prerequisite  - Prepare template, keep Template is ready.

	7. Under Specify  template, choose Upload a template file, and then  choose Choose file.
	8. Choose the file and  then choose Next.
	9. On the Specify  StackSet details page, enter a stack name such as LamdaSSMRole, enter a description, and then  choose Next.
	10. On the Configure  StackSet options page, keep the default options and then  choose Next.
	11. On the Set  deployment options page, for Add stacks to stack set,  keep the default Deploy new stacks option.
	12. For Deployment  targets, choose if you want to create the stack for the entire  organization or specific OUs. If you choose an OU, enter the OU ID.
	13. For Specify  regions, enter only one of region because IAM is a global service so deploying into multiple regions will result in a failure
	14. For Deployment  options, for Failure tolerance - optional, enter the  number of accounts where the stacks can fail before CloudFormation stops  the operation. We recommend that you enter the number of accounts that you  want to add, minus one. For example, if your specified OU has 10 member  accounts, enter 9. This means that even if CloudFormation fails the  operation 9 times, at least one account will succeed.
	15. Choose Next.
	16. On the Review page,  review your options, and then choose Submit. You can check the  status of your stack on the Stack instances tab.

 

* Change the title in this README
* Edit your repository description on GitHub

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.

