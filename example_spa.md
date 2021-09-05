# Example Single Page Application (SPA)

It is time to develop a web app that leverages your new SDLC environment. Here we will create a simple contact form, referred to here as a ‘Landing Page’.

Outcomes

By the end of this section, you’ll have the following:

- A structured git repository hosting your Landing Page’s Front and Backend
- A CDK app
- A React Application hosted using S3 & CloudFront
- A backend API Gateway & Lambda Function to handle form submissions
- A DynamoDB table to store the inbound contacts
- A mechanism to link your react Application and the Backend API endpoint dynamically
- A DNS setup following the hierarchy described in Software Development Life Cycle section

## Dev Environment Setup

### Configure AWS CLI for SSO

In order to interact with your different environments using SSO with the AWS CLI, you will need to configure your credentials. You will have to configure a specific profile for it by running the command below for each environment that you want to access.

Here you will setup your first SSO profile, dev, that will be used to interact with the Dev account of your SDLC environment.

1. Run `aws configure sso --profile dev` to create the `dev` profile and configure it for SSO login
1. You will be prompted for the SSO start URL. Enter the URL you customized previously in `initialize_adlc_env.md`
1. It will open a browser window asking for permissions to allow login from this device, hit "Ok" and close the browser window/tab
1. It will show you the accounts available to you in the terminal. Select "Dev"
1. It will display a message with the roles available to you (currently only "AdministratorAccess")
1. It will ask you to confirm the default region for the client
1. It will ask you the default output format (e.g. JSON)
1. Test that the configuration was successful by running `aws --profile=dev sts get-caller-identity`
    ```
    aws --profile=dev  sts get-caller-identity
      
    {
      "UserId": "A1B2C3D4E5F6G7EXAMPLE:admin",
      "Account": "222222222222",
      "Arn": "arn:aws:sts::222222222222:assumed-role/AWSReservedSSO_AdministratorAccess_1234a12345a12aa1/admin"
    }
    ```