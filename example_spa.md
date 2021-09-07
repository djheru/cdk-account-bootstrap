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
1. Install CDK SSO Sync
    - `npm i -g cdk-sso-sync`
1. Propagate your credentials to the CDK
    - `cdk-sso-sync dev`
1. If your login token expires, refresh it by logging in again
    - `aws sso login --profile dev`

## Create Git Repository

We will store the application code in a git repository. We won’t use the one forked in the previous section, in order to avoid complex security settings that would be necessary to keep your Software Development Life Cycle Organization setup and application code separate. Hence, we will create a new one.

1. Create a new repository on GitHub 
    - e.g. `aws-bootstrap-kit-examples-landing-page`
1. Copy the files from ./source/4-landing-page-with-feedback-api to the new repo
    ```bash
    cp -r ./aws-bootstrap-kit-examples/source/4-landing-page-with-feedback-api/. ./aws-bootstrap-kit-examples-landing-page/ 
    ```

## Run the frontend locally

1. Go to the `ui` folder, install dependencies and run the app
    `cd ./ui && npm i && npm start`

## Deploy the App Infrastructure

1. If you have DNS set up in the SDLC section, update the `domain_name` attribute in `infrastructure/cdk.json`
1. Build the CDK app
    - `cd infrastructure && npm install && npm run build:all`
1. Log in as `dev` and initiate deployment of the stack
    ```bash
    aws sso loging --profile dev
    npx cdk-sso-sync dev
    npm run cdk deploy LandingPageStack-dev --profile dev
    ```
1. When prompted, accept the changes to be deployed
1. Retrieve the `FrontendUrl` and `configJSON` from the "Outputs" displayed in the terminal
    ```
    Outputs:
    LandingPageStack-dev.FrontendUrl = https://mylandingpage.dev.damraventures.com
    LandingPageStack-dev.configJSON = {"API_URL":"https://n26qxjouh8.execute-api.us-east-1.amazonaws.com/prod/"}
    ```
1. Copy the `LandingPageStack-dev.configJSON` to the `./ui/public/config.json` file to connect the local UI to the REST API
    ```
    export API_URL=https://n26qxjouh8.execute-api.us-east-1.amazonaws.com/prod/
    echo "{\"API_URL\": \"$API_URL\"}" > ../ui/public/config.json
    cd ../ui && npm start
    ```
1. Send a sample message in the UI
1. Navigate to the DynamoDB console and view the saved message
1. Navigate to the hosted frontend URL displayed in the `LandingPageStack-dev.FrontendUrl` CloudFormation output (e.g. https://mylandingpage.dev.damraventures.com) and repeat the previous 2 steps

## Local Development Setup

1. Install [AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html)
1. Install [Docker](https://docs.docker.com/get-docker/)
1. Generate the CloudFormation templates
    - `npm run cdk -- synth LandingPageStack-dev --no-staging`
    - This will output a generated CloudFormation template with a name like `./cdk.out/LandingPageStackdevSurveyServiceStackxxxxxxxx.nested.template.json`
1. Run the API locally using the generated template
    ```
    sam local start-api \
      -t cdk.out/LandingPageStackdevSurveyServiceStack659623EC.nested.template.json \
      --port 3001 \
      --profile dev
    ```
1. Update the `API_URL` in the `./ui/public/config.json` file with `http://127.0.0.1:3001/
1. Submit a message from the UI that should be running locally
  - This will take a long time the first time you submit, because SAM is spinning up a docker image in the background.
1. Check that the message was saved in DynamoDB
1. If desired, edit `./infrastructure/lib/surveyService/lambda and re-submit to see local changes

## Add Your CI/CD Pipeline

In this section we will take our AWS-deployed website and put it on a fully-fledge CI/CD pipeline that deploys your code to both the Staging and Prod environments.

This process will require interactions from both your operator and developer.

The Operator will grant DevOps permission to your Developer to let him/her access the CI/CD environment.

The Developer will:

- Configure the cicd AWS CLI profile
- Add a pipeline to your landing page app
- Track the deployment and access the website for testing

### Granting DevOps Permissions to Developers

At this stage, the Developer user has no access to the CI/CD account; this is because the Developer user only belongs to the Developers group. Let’s add it to the DevOpsEngineers group to grant it access to the CI/CD account with the appropriate permissions.

1. Sign into the SSO portal as an Administrator user
1. Click on the main account and navigate to the Management Console
1. Go to the "AWS SSO" console
1. Click on "Users" on the side menu
1. Click on the Developer user
1. Click on the "Groups" tab under that user
1. Click "Add to Group"
1. Select the DevOpsEngineers group and submit

### Set up the CI/CD CLI profile

Here you will setup your profile that will be used to interact with your CICD account (--profile cicd). We will do this in a similar manner to how we created our dev CLI profile.

1. Execute `aws configure sso --profile cicd` 
1. Enter the start URL
1. Enter the region
1. For the account, select "CICD"
1. Log into the CICD account using the CLI
    - `aws sso login --profile cicd`
1. Test the credentials by running `aws s3 ls --profile cicd`
    - Also `aws --profile=cicd  sts get-caller-identity`
1. Propagate your credentials to the CDK by running the following command
    - `cdk-sso-sync cicd`

### Generate GitHub Token for CI/CD

In this section you will give permissions to your AWS CI/CD pipeline so that it can access your github account and subscribe to code changes.

As in the SDLC environment setup, we will leverage AWS Secrets Manager to store the encrypted personal access token encrypted on AWS.

1. Create a personal GitHub access token with the following scopes: `admin:repo_hook` and `repo` full control
1. Use your `cicd` profile to put the access token in AWS Secrets Manager

    ```bash
    aws secretsmanager create-secret \
      --profile cicd  \
      --name GITHUB_TOKEN \
      --secret-string ghp_GnE55voNzDi30iCcyz1ewltOEXAMPLE
    ```

### Build, Push, Deploy

1. Navigate to the infrastructure directory
1. Ensure that you have configured credentials for `cicd`, and log in
    ```bash
    aws sso login --profile cicd && cdk-sso-sync cicd
    ```
1. Build the CDK app
    - `npm run build`
1. If everything builds successfully, commit and push your changes
    ```bash
    git add .
    git commit -m "Add CICD pipeline"
    git push
    ```
1. Now, deploy the pipeline
    - `npm run cdk deploy LandingPagePipelineStack -- --profile cicd`
    - Acccept the changes and confirm