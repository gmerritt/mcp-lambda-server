# MCP Lambda Server - `infra/` Directory

This directory contains the AWS Cloud Development Kit (CDK) infrastructure code to deploy the MCP server as a serverless application on AWS Lambda. It sets up the necessary S3 buckets, Cognito authentication, and API Gateway integration.

## Architecture Highlights

- **AWS Lambda**: Runs the Express server using the [AWS Lambda Web Adapter](https://github.com/awslabs/aws-lambda-web-adapter). This allows the standard Node.js/Express app in `server/` to run in a serverless environment.
- **Amazon S3**: A bucket for storing prompt templates (e.g., `greetings_prompt.txt`).
- **Amazon Cognito**: A User Pool and Client to secure the `/mcp` endpoint.
- **Amazon API Gateway (HTTP API)**: The front-facing interface for the MCP server, integrated with Lambda and secured by Cognito.

## Key Files

### bin/infra.ts
- The entry point for the CDK application. It initializes the `InfraStack`.

### lib/infra-stack.ts
- Defines the main infrastructure stack:
  - `PromptsBucket`: Private S3 bucket for prompt templates.
  - `httpHandler`: A Node.js Lambda function that bundles the code from `../server/`.
    - Includes a Lambda Layer for the Web Adapter.
    - Sets up environment variables like `BUCKET_NAME`.
    - Grants read access to the S3 bucket.
  - `mcpUserPool` & `mcpUserClient`: Authentication resources.
  - `mcpAPI`: HTTP API Gateway with a default Cognito authorizer.

### prompts/greetings_prompt.txt
- A sample prompt template file that is intended to be uploaded to the S3 bucket.

### authenticate.sh
- A helper script to test Cognito authentication using the `admin-initiate-auth` command.

## Useful Commands

* `npm run build`   Compile TypeScript to JavaScript.
* `npx cdk deploy`  Deploy this stack to your default AWS account/region.
* `npx cdk diff`    Compare deployed stack with current state.
* `npx cdk synth`   Emit the synthesized CloudFormation template.
* `npx cdk destroy` Remove the deployed infrastructure.

## Deployment Instructions

1. **Install Dependencies**:
   ```sh
   npm install
   ```
2. **Bootstrap CDK** (if first time in account/region):
   ```sh
   npx cdk bootstrap
   ```
3. **Deploy**:
   ```sh
   npx cdk deploy
   ```
4. **Post-Deployment**:
   - The deployment will output the `userPoolId` and `userPoolClientId`.
   - Upload the contents of the `prompts/` folder to the newly created S3 bucket.
   - Update `authenticate.sh` with the correct IDs to test authentication.

## Security
- The S3 bucket is configured with `BLOCK_ALL` public access.
- The HTTP API is protected by a Cognito User Pool Authorizer, requiring a JWT for all requests.
