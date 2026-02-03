# MCP Lambda Server

Deploy a Model Context Protocol (MCP) server as a serverless application on AWS Lambda using HTTP API Gateway. This project leverages the `Streamable HTTP` transport from the MCP SDK to enable stateless, high-performance prompt handling with prompt templates stored in Amazon S3.

## Overview

This repository provides a complete implementation of an MCP server that acts as a **Prompt Library**. It uses:
- **TypeScript & Express**: To build the core MCP server.
- **AWS CDK**: For Infrastructure as Code (IaC) deployment.
- **AWS Lambda Web Adapter**: To run the standard Express application in a serverless Lambda environment.
- **Amazon S3**: For storing and dynamically loading prompt templates.
- **Amazon Cognito**: To secure the `/mcp` endpoint with JWT-based authorization.

## Architecture

The serverless architecture ensures that the MCP server remains cost-effective and scales automatically.

![Architecture Diagram](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvwdwdzm5jzdh26gsq4zj.png)

1.  **HTTP API Gateway**: Acts as the entry point, resolving requests and enforcing Cognito authorization.
2.  **AWS Lambda**: Executes the Express server. The **Lambda Web Adapter** layer translates HTTP requests into the format expected by the Node.js app.
3.  **Amazon S3**: Stores `.txt` prompt templates (e.g., `prompts/greetings_prompt.txt`) which are fetched at runtime.
4.  **Cognito User Pool**: Manages user authentication and provides JWT tokens for secure access.

## Repository Structure

- [infra/](infra/): AWS CDK infrastructure code.
  - [infra/lib/infra-stack.ts](infra/lib/infra-stack.ts): Defines S3, Cognito, Lambda, and API Gateway resources.
  - [infra/prompts/](infra/prompts/): Sample prompt templates to be uploaded to S3.
- [server/](server/): The Express application source code.
  - [server/server/server.ts](server/server/server.ts): MCP server initialization and prompt registration.
  - [server/services/s3Service.ts](server/services/s3Service.ts): Service for fetching templates from S3.
  - [server/main.ts](server/main.ts): Express entry point and MCP transport handling.

## Prerequisites

- [Node.js](https://nodejs.org/) (v20 or later)
- [AWS CLI](https://aws.amazon.com/cli/) configured with appropriate credentials.
- [AWS CDK CLI](https://docs.aws.amazon.com/cdk/v2/guide/getting_started.html) (`npm install -g aws-cdk`)
- An S3 bucket (created during deployment) to store prompts.

## Quick Start

### 1. Local Development

You can run the server locally for testing:

```bash
cd server
npm install
export BUCKET_NAME=your-local-test-bucket
npx ts-node main.ts
```

The server will be available at `http://localhost:8080/mcp`. You can test it using the [MCP Inspector](https://github.com/modelcontextprotocol/inspector).

### 2. Infrastructure Deployment

Deploy the stack to AWS:

```bash
cd infra
npm install
npx cdk bootstrap # Only if first time in account/region
npx cdk deploy
```

Take note of the `mcpAPIEndpoint`, `userPoolId`, and `userPoolClientId` from the CDK outputs.

### 3. Upload Prompt Templates

After deployment, upload your prompt templates to the created S3 bucket:

```bash
# Upload the sample greeting prompt
aws s3 cp infra/prompts/greetings_prompt.txt s3://<YOUR_BUCKET_NAME>/prompts/greetings_prompt.txt
```

## Security & Authentication

The `/mcp` endpoint is protected by a Cognito JWT Authorizer. To interact with the server, you must provide a valid `Authorization` header.

Use the provided helper script to authenticate and get a token:

1.  Update [infra/authenticate.sh](infra/authenticate.sh) with your `USER_POOL_ID` and `CLIENT_ID`.
2.  Run the script:
    ```bash
    ./infra/authenticate.sh
    ```
3.  Use the returned `IdToken` in your requests:
    ```http
    Authorization: Bearer <ID_TOKEN>
    ```

## Testing with MCP Inspector

To test the deployed server with the MCP Inspector:

1.  Open the [MCP Inspector](https://github.com/modelcontextprotocol/inspector).
2.  Choose the **HTTP Streamable** transport.
3.  Enter your API Gateway URL (e.g., `https://<api-id>.execute-api.<region>.amazonaws.com/mcp`).
4.  (Note: Ensure your authorizer allows the Inspector's requests or use a tool like [Bruno](https://usebruno.com) for authenticated testing).

## References

- **Blog Post**: [MCP Server with AWS Lambda and HTTP Api Gateway](https://dev.to/aws-builders/mcp-server-with-aws-lambda-and-http-api-gateway-1j49)
- **Model Context Protocol**: [Official Documentation](https://modelcontextprotocol.io/)
- **AWS Lambda Web Adapter**: [GitHub Repository](https://github.com/awslabs/aws-lambda-web-adapter)

---
Built with ❤️ using the Model Context Protocol.
