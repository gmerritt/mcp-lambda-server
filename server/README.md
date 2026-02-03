# MCP Lambda Server - `server/` Directory

This directory contains the source code for the MCP Lambda Server, which provides an HTTP interface for Model Context Protocol (MCP) prompt handling, with prompt templates stored in S3. The server is built with Express and TypeScript, and leverages the `@modelcontextprotocol/sdk` and AWS SDK for S3 integration.

## Directory Structure

```
server/
├── .gitignore
├── main.ts
├── package.json
├── run.sh
├── tsconfig.json
├── server/
│   └── server.ts
└── services/
    └── s3Service.ts
```

## File Overview

### .gitignore
- Ignores `node_modules` to prevent dependencies from being committed.

### main.ts
- Entry point for the server.
- Sets up the Express app and JSON middleware.
- Initializes the S3 client and `S3Service`.
- Handles `/mcp` POST requests by connecting the MCP server and streaming responses.
- Handles unsupported HTTP methods (GET, DELETE) on `/mcp` with 405 errors.
- Listens on port 8080.

### package.json
- Lists project dependencies:
  - `@aws-sdk/client-s3` for S3 access
  - `@modelcontextprotocol/sdk` for MCP server
  - `express` for HTTP server
  - `zod` for schema validation
  - `@types/express` for TypeScript types
- Contains basic project metadata and scripts.

### run.sh
- Simple shell script to run the server using Node.js.
- Note: The script runs `node index.js`, but the main entry point is `main.ts` (ensure build step or entry point alignment).

### tsconfig.json
- TypeScript configuration for compiling the server code.
- Targets ES2016 and uses CommonJS modules.

### server/server.ts
- Exports `initializeServer`, which creates and configures an MCP server instance.
- Registers a `greeting-template` prompt that loads a template from S3 and injects a name.
- Uses Zod for input validation.

### services/s3Service.ts
- Defines the `S3Service` class for interacting with AWS S3.
- Provides a `getFile(filePath: string): Promise<string>` method to fetch file contents from S3.
- Throws an error if the file is not found.

## Environment Variables
- `BUCKET_NAME`: The name of the S3 bucket containing prompt templates. If not set, defaults to "bucket name missing" (which will cause errors).

## Usage

1. **Install dependencies:**
   ```sh
   npm install
   ```
2. **Set environment variables:**
   - Export `BUCKET_NAME` with your S3 bucket name.
   - Ensure AWS credentials are available (via environment, profile, or IAM role).
3. **Run the server:**
   ```sh
   npx ts-node main.ts
   # or build and run with node
   # tsc && node dist/main.js
   ```
   Or use the provided `run.sh` (ensure the entry point matches your build output).

## Endpoints

- `POST /mcp`: Handles MCP requests. Expects a JSON body. Streams responses using the MCP protocol.
- `GET /mcp`, `DELETE /mcp`: Return 405 Method Not Allowed.

## Prompt Templates
- Prompt templates are stored in S3 under the `prompts/` prefix (e.g., `prompts/greetings_prompt.txt`).
- The `greeting-template` prompt replaces `{{name}}` in the template with the provided name.

## Extending the Server
- Add new prompts by registering them in `server/server.ts`.
- Add new S3 interactions or services in `services/s3Service.ts`.

## Development Notes
- Written in TypeScript for type safety.
- Uses Zod for runtime schema validation.
- Designed for easy extension with new prompts and S3-backed resources.

---

For more information, see the root `README.md` or the documentation for the [Model Context Protocol SDK](https://www.npmjs.com/package/@modelcontextprotocol/sdk) and [AWS SDK for JavaScript v3](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/).
