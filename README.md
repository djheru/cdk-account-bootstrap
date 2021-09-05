# Bootstrapping your AWS Account

## Introduction

### Introduction to AWS Identity and Access Management (IAM)

Interacting with AWS requires both:

- **Authentication** - Ensuring that the user is who they claim to be
- **Authorization** - Ensuring that the user has permission to do what they're attempting to

AWS provides the IAM service to manage authentication and authorization across the platform

IAM enables authentication of **Principals**, entities such as IAM users, users from 3rd-party identity providers, IAM roles, AWS accounts and AWS services

IAM enables authorization using JSON documents called **Policies**

#### Helpful mnemonic: PARC

- **P**rincipal - The entity that is allowed or denied access
- **A**ction - The type of access that is allowed or denied
- **R**esource - The AWS resources the action will use
- **C**ondition - The conditions under which the access is valid

IAM principals use *credentials* to interact with AWS. Credentials are composed of an **Access Key ID** and a **Secret Access Key**

## Authentication

### Users

One of the most commonly used principals in AWS is the IAM user. An IAM user typically represents a human who has a username and password that they use to authenticate. You can have many IAM users in an AWS account. An IAM user could, for example, represent a member of your team.

#### Root User

When creating an AWS account, one usually signs up with an email address and password. This creates a special user associated with that email address and password that has complete access to all AWS services and resources in the account. This is the AWS account’s root user.

### Roles

An IAM role is identical in function to an IAM user, with the important distinction that it is not uniquely associated with one entity, but assumable by many entities. Typically, IAM roles correspond to a job function.

A loose analogy for IAM roles are that of professional uniforms: a surgeon’s scrubs, a firefighter’s helmet, or a startup CTO’s favorite unwashed hoodie. Many people can assume the role of a surgeon, firefighter, and startup CTO, which identifies them with a certain job function. It’d be pretty weird if you needed surgery and found out a firefighter would be performing it, thus we apply roles to the entities that are appropriate.

#### Service Roles

IAM roles can be associated with individual users, but they can also be associated with an AWS service. These roles are known as **service roles**

For example, you can assign a role directly to an EC2 instance or some other resource. Then you can associate specific IAM policies with the role, so the EC2 instance use those permissions to perform tasks on other AWS services

## Authorization

So far we’ve been discussing **IAM Principals**. These represent the authentication component of accessing AWS. We authorize principals by attaching JSON documents to them called **IAM policies**. Policies define elements you can remember by the pneumonic **PARC**.

### Principals

The entities that are allowed or denied access. Commonly these are IAM users and IAM roles

### Actions

Actions are the type of access that is allowed or denied. Actions are commonly AWS service API calls that represent create, read, describe, list, update, and delete semantics.

### Resources

Resources are the AWS resources the action will act upon. All AWS resources are identified by an [Amazon Resource Name (ARN)](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html). Because AWS is deployed all over the world, ARNs function like an addressing system to precisely locate which specific part of AWS we are referring to. ARNs have these hierarchical structures:

```
arn:partition:service:region:account-id:resource-id
arn:partition:service:region:account-id:resource-type/resource-id
arn:partition:service:region:account-id:resource-type:resource-id
```

#### Description of the segments

- the literal `arn` indicates that this string is an ARN
- **partition** - One of the three AWS partitions
  - AWS Regions
  - AWS China Regions
  - AWS GovCloud (US) Regions
- **service** - The specific AWS service like EC2 or S3
- **region** - The AWS region (e.g. us-east-1)
- **account-id** - The AWS account ID
- **resource-type/resource-id** - The unique identifier for the resource

### Conditions

*Conditions* are the specific rules for when the access is valid

### Other Elements

- All IAM policies have an *Effect* field which is either **Allow** or **Deny**
- The **Version** field defines which IAM service API version to use when evaluating the policy
- The **Statement** field consists of one or many JSON objects that contain the specific `Action`, `Effect`, `Resource`, and `Condition` fields above

### Example

```JSON
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeInstances",
                "ec2:RunInstances"
            ],
            "Resource": "*",
            "Condition" :  {
                 "IpAddress" : {
                    "aws:SourceIp" : "12.34.56.78/32"
                }
            }
        }
    ]
}
```

- First notice there is no **Principal** element in the policy. This is because attaching a policy to a principal implicitly specifies the principal the policy applies to. Since policies can exist apart from principals, this allows us to reuse common policies amongst many users, roles, services, etc.
- The **Effect** is Allow, so this is a policy that explicitly grants access. This is the most common type of policy. You can write policies that explicitly Deny access as well.
- The **Action** is an array of multiple actions that permit using EC2 to run instances and describe instance metadata. Actions can be a single action, or multiple actions in an array like depicted here.
- There is no specific ARN in the **Resource** field, but instead *, which is a wildcard character that means this policy applies to all resources.
- Finally, we have a **Condition** set that applies this policy only if the caller’s IP address matches exactly 12.34.56.78. Conditions are optional, and do not need to be specified if the policy is to be applied unconditionally.

The /32 is part of CIDR notation, which allows us to specify ranges of IP addresses in a compact way. In this case, the /32 means this IP address is not a range, but that every number in the IP address is to be matched exactly.

So if, for example, an IAM user had this IAM policy attached and was attempting to run an EC2 instance and retrieve metadata about it, calling the EC2 API from their laptop that has an IP address of 12.34.56.78, they would be allowed to do so.

It can go a lot further than this. IAM policies can be combined each with varying degrees of sensitivity and specificity, the net effect of which allows for fine-grained access control to every resource in an AWS account.

### How Policies are Evaluated

- For IAM principals, all requests to AWS are ***denied*** by default
- If the principal hash an attached policy with *Allow*, the implict *Deny* is overridden
- **However**, an explicit *Deny* in any policy attached to the principal will override any *Allow*s

#### Root User Access

- ***Important*** Actions taken by the AWS root account user are always implicitly ***Allowed***
- This is the only AWS principal that functions with this type of access

## Credentials

- Credentials are required to interact with AWS
- Credentials are associated with IAM principals like IAM users and roles
- Credentials consist of at least two strings, 
  - Access Key ID
  - Secret Access Key
  - May optionally contain a session token

Nearly all AWS services present secure HTTPS based APIs that require authentication using HMAC signatures calculated with an algorithm called Signature Version 4. You should rarely if ever have to use Signature Version 4 directly, because the AWS command line interface (CLI) and AWS Software Development Kit (SDK) perform it for you. You need only provide credentials in your environment.

When we speak of credentials in the context of using AWS services, we’re referring to a pair of strings called an access key ID and secret access key. These credentials are associated with an IAM principal like an IAM user or IAM role.

Here are what credentials look like:

```bash
AWS_ACCESS_KEY_ID="AKIAIOSFODNN7EXAMPLE"
AWS_SECRET_ACCESS_KEY="wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
```

### Temporary Credentials

It is also possible to use credentials that permit temporary access. These credentials have a third value, a session token, that is included along with the access key ID and secret access key, which looks like this:

```bash
AWS_SESSION_TOKEN="000000000000011111111111112222222222233333333333444444444444444555555555556666666666666667777777888888888888899999999999999AAAAAAAABBBBBBCCCCCCCCCCCCDDDDDDDDDDEEEEEEEEEEEEEEFFFFFFFFFFF+++++++++++GGGGGGGGGHHHHHHHHHHHHIIIIIIIIIIIJJJJJJJJJJJJKKKKKKKKKKKKKLLLLLLLLLMMMMMMMMMMMMMNNNNNNNNNNNNNOOOOOOOOOOOOOOOOOPPPPPPPPPQQQQQQQQQQQQQQRRRRRRRRRRRRRSSSSSSSSSSSSSTTTTTTTUUUUUUUUUUUUUUUUUUUUUUUUVVVVVVVVVVVVVVWWWWWWWWWWWWWWWWWWWWXXXXXXXXXYYYYYYYYYYYYYZZZZZZZZZZaaaaaaaaaaaaaabbbbbbbbbbbbbbcccccccccccdddddddddddddeeeeeeeeeeeeeeffffffffffffggggggggggggggghhhhhiiiiiiiiiiiiijjjjjjj//////////////kkkkkkkkkkkklllllllllllllmmmmmmmmmmnnnnnnnnnnnnnnoooooooooooooooopppppppppppqqqqqqqqqqqrrrrrrrrrrrrrrssssssssstttttttttttttuuuuuuuuvvvvwwwwwwwwwwwwwwwxxxxxxxyyyyyyyyyyyyyyyzzzzzzzzz="
```

## Introduction to AWS Cloud Development Kit

The AWS Cloud Development Kit (CDK) is a new software development framework from AWS. It intends to make cloud deployments fun and easy by providing the capability to define stacks in the programming language of your choice. The compiled definitions are then deployed using AWS CloudFormation.

AWS CDK apps are composed of building blocks known as constructs. Constructs are composed together to form stacks and apps.

### Constructs

Constructs are the basic building blocks of AWS CDK apps. A construct represents a “cloud component” and describes everything that AWS CloudFormation needs to create the component.

A construct can represent a single resource, such as an Amazon Simple Storage Service (Amazon S3) bucket, or it can represent a higher-level component consisting of multiple AWS resources. Examples of such components include a worker queue with its associated compute capacity, a cron job with monitoring resources and a dashboard, or even an entire app spanning multiple AWS accounts and regions.

We call your CDK application an app. Apps are represented by the AWS CDK class [App](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.App.html).


### Stacks

The unit of deployment in the AWS CDK is called a stack. All AWS resources defined within the scope of a stack, either directly or indirectly, are provisioned as a single unit.

Because AWS CDK stacks are implemented through AWS CloudFormation stacks, they have the same limitations as in AWS CloudFormation.

### Environments

Each Stack instance in your AWS CDK app is explicitly or implicitly associated with an environment (env). An environment is the target AWS account and region into which the stack is intended to be deployed

### Stages of CDK Deploy Operations

#### 1. Construction (or Initialization)

Your code instantiates all of the defined constructs and then links them together. In this stage, all of the constructs (app, stacks, and their child constructs) are instantiated and the constructor chain is executed. Most of your app code is executed in this stage.

#### 2. Preparation

All constructs that have implemented the prepare method participate in a final round of modifications, to set up their final state. The preparation phase happens automatically. As a user, you don’t see any feedback from this phase. It’s rare to need to use the “prepare” hook, and generally not recommended. You should be very careful when mutating the construct tree during this phase, because the order of operations could impact behavior.

#### 3. Validation

All constructs that have implemented the validate method can validate themselves to ensure that they’re in a state that will correctly deploy. You will get notified of any validation failures that happen during this phase. Generally, we recommend that you perform validation as soon as possible (usually as soon as you get some input) and throw exceptions as early as possible. Performing validation early improves diagnosability as stack traces will be more accurate, and ensures that your code can continue to execute safely.

#### 4. Synthesis

This is the final stage of the execution of your AWS CDK app. It’s triggered by a call to app.synth(), and it traverses the construct tree and invokes the synthesize method on all constructs. Constructs that implement synthesize can participate in synthesis and emit deployment artifacts to the resulting cloud assembly. These artifacts include AWS CloudFormation templates, AWS Lambda application bundles, file and Docker image assets, and other deployment artifacts. Cloud assemblies describes the output of this phase. In most cases, you won’t need to implement the synthesize method

#### 5. Deployment

In this phase the AWS CDK CLI takes the deployment artifacts that cloud assembly produced in the synthesis phase and deploys it to an AWS environment. It uploads assets to Amazon S3 and Amazon ECR, or wherever they need to go, and then starts an AWS CloudFormation deployment to deploy the application and create the resources.

By the time the AWS CloudFormation deployment phase (step 5) starts, your AWS CDK app has already finished and exited. This has the following implications:

The AWS CDK app can’t respond to events that happen during deployment, such as a resource being created or the whole deployment finishing. To run code during the deployment phase, you have to inject it into the AWS CloudFormation template as a custom resource. For more information about adding a custom resource to your app, see the AWS CloudFormation module, or the custom-resource example.

The AWS CDK app might have to work with values that can’t be known at the time it runs. For example, if the AWS CDK app defines an Amazon S3 bucket with an automatically generated name, and you retrieve the bucket.bucketName attribute, that value is not the name of the deployed bucket. Instead, you get a Token value. To determine whether a particular value is available, call cdk.isToken(value).

### Bootstrapping

For all but the simplest deployments, you will need to bootstrap each environment you will deploy into. Deployment requires certain AWS resources to be available, and these resources are provisioned by bootstrapping.

## Introduction to AWS Organizations

AWS Organizations is an account management service that lets you consolidate multiple AWS accounts into an organization that you create and centrally manage. With AWS Organizations, you can create member accounts and invite existing accounts to join your organization. You can organize those accounts and manage them as a group.

### Organization

An organization is an entity that you create to consolidate your AWS accounts so that you can administer them as a single unit. You can use the AWS Organizations console to centrally view and manage all of your accounts within your organization. An organization has one management account along with zero or more member accounts. You can organize the accounts in a hierarchical, tree-like structure with a root at the top and organizational units nested under the root. Each account can be directly in the root, or placed in one of the OUs in the hierarchy. An organization has the functionality that is determined by the feature set that you enable.

### Root

The parent container for all of the accounts in your organization. If you apply a policy to the root, it applies to all organizational units and accounts in the organization

### Organizational Unit

A container for accounts within a root. An OU also can contain other OUs, enabling you to create a hierarchy that resembles an upside-down tree, with a root at the top and branches of OUs that reach down, ending in accounts that are the leaves of the tree. When you attach a policy to one of the nodes in the hierarchy, it flows down and affects all the branches (OUs) and leaves (accounts) beneath it. An OU can have exactly one parent, and currently each account can be a member of exactly one OU.

### Account

A standard AWS account that contains your AWS resources. You can attach a policy to an account to apply controls to only that one account.

There are two types of accounts in an organization: a single account that is designated as the **management account**, and **member accounts**.

The **management account** is the account that you use to create the organization. From the organization’s management account, you can do the following:

- Create accounts in the organization
- Invite other existing accounts to the organization
- Remove accounts from the organization
- Manage invitations
- Apply policies to entities (roots, OUs, or accounts) within the organization

The management account has the responsibilities of a payer account and is responsible for paying all charges that are accrued by the member accounts. You can’t change an organization’s management account.

The rest of the accounts that belong to an organization are called **member accounts**. An account can be a member of only one organization at a time.

See the [Documentation](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_introduction.html) for more details

# Setup Your Environments

A Software Development Lifecycle (SDLC) environment is a structured environment that is used to develop and operate your software throughout its lifecycle. On AWS, your SDLC environment will cater for the various roles in your IT team. Much of the environment can be described in Infrastructure as Code artifacts.

## Operators

In this context, an operator is someone in your organization that will manage your IT environment

In a complete SDLC environment, operators will have:

- An isolated set of secure environments. The environment isolation enables them to ensure the reliability of systems by limiting blast radius of security issues, bad code changes, and human errors.
- A web portal enabling them to login to each of the user specific environments; dev, staging, prod, CI/CD.
- A central billing system enabling you to control your spend accross your environments
- A central activity view that provides visibility over what is done in each of your environments
- A central users and permissions management service enabling you to control what your team members can and cannot do in your different environments
An isolated DNS management zone to securely control your main DNS domains

## Developers

Similarly, developers are members of your organization that develop software.

Developers will have:

- An environment with wide permissions to experiment, test and develop
- A web portal enabling them to login to each of the user specific environments; dev, staging, prod, CI/CD.
- A simple way to create a multi-stage CI/CD pipeline to securely deploy your code to production
- A simple way to add public DNS records to expose your app or service

## [Initialize your SDLC Environment](./initialize_sdlc_env.md)

