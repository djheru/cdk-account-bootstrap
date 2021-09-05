# Initialize your SDLC environment

This includes the following steps:

- Create an AWS Account
- Create an Administrator IAM user
- Create a Github account
- Create a Github repo for your future AWS Organizations Infrastructure as Code
- Create a Github personal token

## 1. Create an AWS Account

You can get started by [following these steps](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account) or [clicking this direct link](https://portal.aws.amazon.com/billing/signup#/start)

Next, log into the [AWS Console](https://signin.aws.amazon.com/console) as a root user using your email address

## 2. AWS Account Administration

- There is only one root user and it has administrator access to the AWS account it is associated with.
- The root user is identified with the email address and password used during AWS account sign up.
- Don’t use the root user for general AWS administration. Instead, create users with IAM, each with varying permissions.

### The Root User

When first creating an AWS account, it is typical to sign up with an email address and password. This process creates a special user that is associated with that email address and password. This user is known as the root user, and it has complete access to all AWS services and resources in the account.

### IAM Administrative Users

It is recommended that IAM Administrator Users are created that are separate from the root user. This concept derives from a [security best practice](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) of the principle of least privilege. Administrator users will have full administrative access to account, but will not require the use of the email address and password to sign in.

## 3. Enable Billing for IAM Users

At this stage you should be signed in as the root user in your new AWS account. Let’s now enable access to Billing data for IAM users, so that administrators are able to see and analyze the AWS usage and cost without having to login as the root user.

1. In the AWS Management Console’s navigation bar, choose your account name and then choose My Account.
1. Scroll down to "**IAM User and Role Access to Billing Information**", choose Edit.
1. Select the "**Activate IAM Access**" check box.
1. Click the "**Update**" button.

Now that this setting is enabled, any IAM users set up with appropriate privileges will have access to the Billing and Cost Management console. This is especially useful for administrators or operations staff in order to monitor cost. We will grant these privileges to an Administrator IAM User.

## 4. Create an IAM Admin User

Let’s now create a new user using IAM by navigating to the [Add user page](https://console.aws.amazon.com/iam/home?#/users$new).

### Define Credentials

For the new `administrator` users, fill out the form as follows:

- **User name**: `Administrator`
- **Access type**: `AWS Management Console Access`
- **Console password**: Custom Password - `wh@t3v3rY0Uw@nt`
- **Require password reset**: Deselect this option

### Grant Administrator Permissions

IAM allows us to dynamically attach JSON policies that define permissions. IAM comes provides various AWS managed policies that encapsulate common access levels, and also allows us to create our own policies. We can use the AdministratorAccess AWS managed policy to grant your new IAM user with administrator permissions.

To assign the administrator permissions to the user follow these steps:

- **Set permissions**: Select **"Attach existing policies directly"**
- Select **"AdministratorAccess"** from the list of policies that are displayed

### Set Tags

This is optional, you can skip...

### Review Selections

Should look like this: 

- **User name**:  *Administrator*
- **AWS access type**:  *AWS Management Console access - with a password*
- **Console password type**:  *Custom*
- **Require password reset**:  *No*
- **Permissions boundary**:  *Permissions boundary is not set*

### Grant Programmatic Access

Now that we have created an administrator IAM user with console access, we want to grant the ability for it to access AWS programmatically (via the provided APIs and CLI). To access AWS programmatically, the user must have an access key and a secret key.

1. Click the Administrator user in the user list
1. Click the "*Security credentials*" tab
3. Click the "*Create access key*" button
4. ***IMPORTANT** You must copy your **Secret Access Key** here at this point in the process because it will not be shown again
5. Alternatively, you can download the .csv credentials file and save it somewhere safe

## 5. Sign in as an IAM User

Now that we have created our administrative user, we can follow best practices and avoid using the root user whenever possible

To sign in as the IAM user, first (before you sign out as root), look for the dropdown menu in the header with your name. Click it and copy your account number from the dropdown, then click "Sign out"

Navigate back to the sign in page and sign in as an IAM user. Paste your account ID in the first box, then enter "*Administrator*" as the username and whatever you chose as the password

___ 

# Local Environment Setup

## Install the AWS CLI

To interact with AWS from your terminal you will need to install the AWS Command Line Interface (CLI).

As stated in [the official documentation](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) it’s as simple as running:

### MacOS

```bash
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /
aws --version
```

### Windows

Download execcutable here: [https://awscli.amazonaws.com/AWSCLIV2.msi](https://awscli.amazonaws.com/AWSCLIV2.msi)

```bash
C:\> aws --version
aws-cli/2.0.47 Python/3.7.4 Windows/10 botocore/2.0.0
```

### Linux

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" \
  -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

## Configure Credentials

To authenticate requests made using the CLI, you need to provide the programmatic access credentials that were generated when you created your IAM User. You also need to decide which AWS region you would like to refer to when running commands from the CLI.

1. Run `aws configure --profile main-admin` to set up credentials for a new CLI profile called `main-admin`

    ```
    aws configure --profile main-admin
    AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
    AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
    Default region name [None]: eu-west-1
    Default output format [None]: json
    ```
1. Check that the setup was successful by running `aws --profile=main-admin sts get-caller-identity`
    - The `Account` field should show the root account ID
    - The `Arn` field should show that you're using the `Administrator` account

    ```
    aws --profile=main-admin  sts get-caller-identity

    {
        "UserId": "A1B2C3D4E5F6G7EXAMPLE",
        "Account": "111122223333",
        "Arn": "arn:aws:iam::111122223333:user/Administrator"
    }
    ```

To learn more, consult [the official CLI documentation](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-config)

## GitHub Setup

1. [Create a GitHub account](https://github.com/join) if you don't already have one
1. [Register an SSH key](https://docs.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent) If you don't have one already
1. Create a [personal github access token](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token) with the following scopes: 
    - `repo`: **full control**
    - `admin:repo_hook`: **full control**
    - It will be used by the CI/CD pipeline to react to activity in the repo
1. **Copy the token to a safe place**

## Repository Setup

### Fork and Clone

In order to work from our examples, but with them configured for your own environment, fork and the clone our examples repository.

1. Fork the repository on your GitHub account by [clicking here](https://github.com/aws-samples/aws-bootstrap-kit-examples/fork)
1. Clone the forked repository to your local machine

### Link to AWS

The code for your SDLC environment that is stored in GitHub will be used to deploy infrastructure to AWS. To do this, the repository needs to be linked to and accessible from AWS.

1. Store your GitHub token as a secret in AWS Secrets Manager
    ```bash
    aws --profile main-admin \
      secretsmanager create-secret \
      --name GITHUB_TOKEN \
      --secret-string <YOUR_GITHUB_PERSONAL_ACCESS_TOKEN>

    # Response
    # {
    #   "ARN": "arn:aws:secretsmanager:us-east-1:822550785339:secret:GITHUB_TOKEN-IBWMWt",
    #   "Name": "GITHUB_TOKEN",
    #   "VersionId": "bc83c608-02a0-420d-b1c6-34cecea908a6"
    # }
    ## INFO ##
    # This secret stored in Secrets Manager under GITHUB_TOKEN
    # will be reused by the CI/CD Pipeline using the exact same secret name : GITHUB_TOKEN . 
    # So don't change it, or if you do make sure to change as well the referenced value [https://github.com/aws-samples/aws-bootstrap-kit-examples/blob/main/source/1-SDLC-organization/lib/cicd-stack.ts#L70
    ```
1. Update source/1-SDLC-organization/cdk.json to hold the following key/value pairs
    - `email` - The administrator email address that will be used to create additional accounts (must not have a "+")
    - `github_alias` - The github username
    - `github_repo_name` - The name of the forked and cloned repository
    - `github_repo_branch` - The main branch of your repo (e.g. "main")
    - `pipeline_deployable_regions` - The list of AWS Regions you plan to deploy your future applications to
    - (optional) `domain_name` - A DNS domain name to expose your services publicly

    The result should look something like this:

    ```JSON
    {
      "app": "npx ts-node bin/sdlc-organization.ts",
      "context": {
        "@aws-cdk/core:enableStackNameDuplicates": "true",
        "aws-cdk:enableDiffNoFail": "true",
        "@aws-cdk/core:stackRelativeExports": "true",
        "@aws-cdk/core:newStyleStackSynthesis": true,
        "github_alias": "djheru",
        "github_repo_name": "aws-bootstrap-kit-examples",
        "github_repo_branch": "main",
        "email": "pdamra@gmail.com",
        "domain_name": "example.com",
        "force_email_verification": true,
        "pipeline_deployable_regions": [
          "us-east-1"
        ]
      }
    }
    ```

# Deploy your SDLC Environment

By completing the following tasks, you will create an AWS CodePipeline CI/CD pipeline which will:
  - Connect to your github repo hosting the code of the SDLC Landing Zone
  - Create an AWS Organization in your main account with the following structure:
    - Shared Services Organization Unit
      - CI/CD account
    - SDLC Organization Unit (Software Development Life Cycle organization)
      - Dev account
      - Staging account
      - Prod account
  - Secure account access by
    - Enforcing Multifactor Authentication (MFA) usage on the root user access of your main account
    - Enforcing deletion of access and secret keys of the root user of your main account (if you have one)
    - Activating and centralize AWS Cloudtrail logs on all your organization account
  - Create a DNS structure for future public exposition of your services
  - Update itself based on changes you might do on this source code repository

## Install Dependencies

  1. Navigate to the SDLC organization folder
      - `cd source/1-SDLC-Organization`
  1. Install project dependencies with a clean install
      - `npm ci`
  1. Bootstrap the CDK deployment using the *cdk bootstrap* command. This is a required step to proceed
      - `npm run bootstrap`

## Deploy Pipeline

1. Build and deploy the CDK application
    - `npm run build`
    - `npm run deploy`
    - Press **Y** to confirm you wish to make the changes
    - It will take *a while...*
    ```
    # Example Output
    ✅  AWSBootstrapKit-LandingZone-PipelineStack

    Outputs:
    AWSBootstrapKit-LandingZone-PipelineStack.PipelineConsoleUrl = https://us-east-1.console.aws.amazon.com/codesuite/codepipeline/pipelines/AWSBootstrapKit-LandingZone/view?region=us-east-1

    Stack ARN:
    arn:aws:cloudformation:us-east-1:810550787339:stack/AWSBootstrapKit-LandingZone-PipelineStack/aa6d28c0-0e17-11ec-ab2a-0ad0300346b1
    ```
1. Click the link in the output to view the pipeline
1. Click the "Review" button to continue the deployment
1. Look for the subscription verification email in the root account email
1. Look for the email verification email in the root account email
1. Look for the AWS Organizations email address verification email
1. Look for the "Welcome to AWS" emails incoming for your PROD and DEV environments
1. *Please continue to hold...*
1. When the pipeline is finished, [Review the Created Accounts](https://console.aws.amazon.com/organizations/v2/home/accounts) in the AWS organizations console

## Finalize DNS Setup

If you want to use a custom domain name and have added a value in the domain_name field of cdk.json, you have one last step to take to delegate your registered domain to the new hosted zone created in the previous deployment.

If you do not want a custom domain name you can skip this section.

### Register the Nameservers

1. Go to the [Route53 Hosted Zone page](https://console.aws.amazon.com/route53/v2/hostedzones#)
1. Click on the domain name that has 5 records associated with it
1. Copy the NS records for the root domain (not subdomains)
    ```
    ns-1470.awsdns-55.org.
    ns-525.awsdns-01.net.
    ns-386.awsdns-48.com.
    ns-1828.awsdns-36.co.uk.
    ```
1. Copy the NS values to the domain registrar

## Configure SSO

### Enable SSO

To allow users to sign in to your AWS environment with Single Sign-On (SSO), you need to activate AWS SSO.

The basic procedure is outlined here, but you can check the [official documentation](https://docs.aws.amazon.com/singlesignon/latest/userguide/step1.html) for more details.

1. Go to the [AWS SSO Home Page](https://console.aws.amazon.com/singlesignon/home)
1. Click the "Enable AWS SSO" button

### Create Permission Sets

A permission set is a collection of administrator-defined policies that AWS SSO uses to determine a user’s effective permissions to access a given AWS account. Our environment requires the creation of 5 permission sets.

- ***AdministratorAccess***: this permission set grants administrator access to an AWS account. A user with this permission set is able to create, update or delete any resources in an AWS account including IAM users, roles and groups. It relies on the AdministratorAccess AWS managed job function policy.
- ***DeveloperAccess***: this permission set allows developers to create, update or delete AWS resources from an account, but prevents them from creating, updating and deleting IAM users and groups. A developer with this permission set is also able to create, delete and update IAM roles, and create, update, delete and attach role policies to resources. It allows a developer to directly deploy a CDK app into an account with the cdk deploy command. It relies on the PowerUserAccess AWS managed job function policy along with a set of extra IAM actions.
- ***DevOpsAccess***: this permission set allows DevOps engineers to deploy and manage CI/CD pipelines through CDK. It relies on 5 AWS managed policies: AWSCloudFormationFullAcces, AWSCodeBuildAdminAccess, AWSCodePipelineFullAccess, AmazonS3FullAccess, AmazonEC2ContainerRegistryFullAccess, SecretsManagerReadWrite. It relies on a set of IAM, KMS and Organizations actions.
- ***ApproverAccess***: this permission set allows users to view and approve manual changes for all pipelines. It relies on the AWSCodePipelineApproverAccess AWS managed policy.
- ***ViewOnlyAccess***: this permission set allows users to view resources and basic metadata across all AWS services. It relies on the ViewOnlyAccess AWS managed policy.

#### Administrator Access

1. Click the *AWS Accounts* section
1. Go to the *Permission sets* tab and click *Create permission set*
1. Select "Use existing job function policy" and click "Next: Details"
1. Select the ***AdministratorAccess*** Job function policy and click "Next: Tags", then click "Review"
1. Click "Create"

#### Developer Acccess

1. Click "Create Permission Set"
1. "Create a custom permission set"
1. ***DeveloperAccess***
1. Click the "Attach AWS Managed policies" and "Create a custom permissions policy" boxes
1. Select the ***PowerUserAccess*** managed policy
1. Enter the following policy into the custom policy input
    ```JSON
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Action": [
                    "iam:CreateRole",
                    "iam:DeleteRole",
                    "iam:GetRole",
                    "iam:PassRole",
                    "iam:UpdateRole",
                    "iam:AttachRolePolicy",
                    "iam:DetachRolePolicy",
                    "iam:PutRolePolicy",
                    "iam:DeleteRolePolicy"
                ],
                "Effect": "Allow",
                "Resource": "*"
            }
        ]
    }
    ```
1. Click through "Tags" and "Review"
1. "Create"

#### DevOps Access

1. "Create Permission Set"
1. "Create Custom Permission Set"
1. ***DevOpsAccess***
1. Check the Attach AWS managed policies and Create a custom permissions policy boxes. Select the following managed policies:
    - AWSCloudFormationFullAccess
    - AWSCodeBuildAdminAccess
    - AWSCodePipeline_FullAccess
    - AmazonS3FullAccess
    - AmazonEC2ContainerRegistryFullAccess
    - SecretsManagerReadWrite
1. Enter the following policy:
    ```JSON
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Action": [
                    "iam:CreateRole",
                    "iam:DeleteRole",
                    "iam:GetRole",
                    "iam:PassRole",
                    "iam:AttachRolePolicy",
                    "iam:DetachRolePolicy",
                    "iam:PutRolePolicy",
                    "iam:GetRolePolicy",
                    "iam:DeleteRolepolicy",
                    "kms:CreateKey",
                    "kms:PutKeyPolicy",
                    "kms:DescribeKey",
                    "kms:CreateAlias",
                    "kms:DeleteAlias",
                    "kms:ScheduleKeyDeletion",
                    "organizations:ListAccounts",
                    "organizations:ListTagsForResource"
                ],
                "Effect": "Allow",
                "Resource": "*"
            },
            {
                "Action": [
                    "sts:AssumeRole"
                ],
                "Effect": "Allow",
                "Resource": "arn:aws:iam::*:role/cdk*"
            }
        ]
    }
    ```
1. "Next: Tags"
1. "Review"
1. "Create"

#### ApproverAccess

1. "Create Permission Set"
1. "Create a custom permission set", "Next: Details"
1. ***ApproverAccess***
1. "Attach AWS managed policies" & "AWSCodePipelineApproverAccess"
1. "Next: Tags" 
1. "Next: Review"
1. "Create"

#### ViewOnlyAccess

1. "Create Permission Set"
1. "Use an existing job function policy"
1. "ViewOnlyAccess"
1. "Tags" -> "Review" -> "Create"
1. Check that all 5 permission sets appear in your AWS accounts page of the AWS SSO console

### Create Groups

An AWS SSO group is a logical combination of users that you define. Each group is assigned different permissions, and these permissions transfer to the group members. Here we will create 4 groups:

- **Administrators**: this user group is for operators that administer all your AWS accounts.
- **Developers**: this group is for developers that need to deploy their application in the Dev account for testing purposes and who need read-only accesses to the Staging and Production accounts for troubleshooting purposes.
- **DevOpsEngineers**: this group is for DevOps engineers who create and maintain the CI/CD pipelines in the CICD account.
- **Approvers**: this group is for users that need access to the AWS CodePipeline console in the CICD account to approve manual approval tasks.

1. Click the *Groups* tab
1. Click "Create group"
1. **Administrators**
1. Repeat 1-3 for **Developers**, **DevOpsEngineers** and **Approvers**

### Assign the Administrators Group

1. Go to the "AWS Accounts" section
1. Select all accounts and click "Assign Users"
1. Go to the "Groups" tab, select "Administrators" group
1. "Next: Permissions sets"
1. Select "AdministratorAccess" permissions and click "Finish"
1. After it finishes, click "Proceed to AWS accounts"

### Create the Administrator User

1. Click on the "Users" section
1. Click "Add User"
1. Fill in the form with the user's personal information
    - *Username* will be used by the user to log in via SSO
    - *Email Address* will be used to invite the SSO user
1. Click "Next: Groups"
1. Select *Administrators* group created previously, then "Add User"
1. Check email and *Accept invitation*
    - Note the *"Your User portal URL:"* URL
    - e.g. https://d-90676fe861.awsapps.com/start/
1. Set password and click "Update user"
1. Click "Continue" and then "AWS Account" card
1. Select your main account from the card and click "Management Console"
1. You are now connected as your new SSO Administrator user

### Assign Other Groups

Now you are going to assign:

- the Developers group to the Dev account with the DeveloperAccess permission set, and to the Staging and Prod accounts with the ViewOnlyAccess permission set.
- the DevOpsEngineers group to the CICD account with the DevOpsAccess permission set.
- the Approvers group to the CICD account with the ApproverAccess permission set.

1. Search for SSO on the console home page and go to the service
1. Click on "AWS Accounts"
1. Select the ***Dev*** account then click "Assign Users"
1. Select the ***Groups*** tab and select "Developers", then "Next: Permission sets"
1. Select "DeveloperAccess" permissions then click "Finish"
1. Click "Proceed to AWS Accounts"
1. Select "Prod" and "Staging" and click "Assign Users"
1. Select the "Groups" tab and select "Developers" group
1. Select "Next: Permission Sets"
1. Select "ViewOnlyAccess" and then "Finish" then "Proceed to AWS Accounts"
1. Repeat steps 3-7 with the CICD account, DevOpsEngineers group and DevOpsAccess
1. Repeat setps 3-7 with the CICD account, Approvers group and ApproverAccess permissions

### Create Developer User

1. Click the "Users" section
1. "Add User"
1. Enter new user details
1. "Next: Groups"
1. Select "Developers" group, click "Add User"
1. Accept invitation from email (Incognito window?), create password
1. Examine your available accounts as a developer
    - Only `Dev`, `Prod`, `Staging` environments are visible
    - Only `Dev` has "DeveloperAccess", the others have "ViewOnlyAccess"

### Customize your SSO Endpoint

Now that SSO has been set up for your environment, you and your colleagues will be able to login through the AWS SSO portal. By default, the SSO portal is accessible through a URL of the form d-xxxxxxxxxx.awsapps.com/start. This will not be very easy to remember. Fortunately, it is possible to customize the endpoint to match your company’s domain.

1. Go to the Management console page for SSO
1. At the bottom of the page, click "Customize"
1. Enter a custom subdomain and submit. 
1. See the new portal URL is your custom subdomain
    - e.g. [https://pdamra.awsapps.com/start](https://pdamra.awsapps.com/start)
1. Try it out: signout of your existing session, and navigate to your SSO url
1. Sign in as the developer, check it out

You have *DeveloperAccess* perms
