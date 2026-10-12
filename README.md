# Serverless TODO App (AWS)

A serverless TODO application where each signed in user manages their own tasks and can attach an image to any task. The backend runs entirely on AWS managed services, with no servers to provision, and the React frontend authenticates users through Auth0.

Built in 2020 during Udacity's Cloud Developer nanodegree, starting from course starter code.

## Tech
* AWS Lambda (Node.js, TypeScript)
* Amazon API Gateway with a custom Lambda authorizer and request validation
* Amazon DynamoDB with a global secondary index on user id
* Amazon S3 with presigned upload URLs for attachments
* AWS X-Ray distributed tracing across API Gateway, Lambda and AWS SDK calls
* Auth0 with RS256 signed JWTs, verified against the Auth0 JWKS endpoint
* Serverless Framework, with least privilege IAM roles per function
* React frontend (TypeScript)

## Architecture

```
React client ──▶ API Gateway ──▶ Lambda authorizer (Auth0 JWT, RS256)
                     │
                     ▼
              Lambda functions ──▶ DynamoDB (todos, UserIdIndex)
                     │
                     └──────────▶ S3 (presigned URL for attachments)

                 X-Ray traces every request end to end
```

The backend follows a layered structure:

| Layer | Folder | Purpose |
|---|---|---|
| Handlers | `backend/src/lambda/` | Thin Lambda entry points for HTTP and auth |
| Business logic | `backend/src/businessLogic/` | TODO rules, independent of AWS |
| Data access | `backend/src/dataLayer/` | DynamoDB and S3 access |
| Auth | `backend/src/auth/` | JWT parsing and verification |
| Models | `backend/src/models/`, `backend/src/requests/` | Typed items and request shapes |

## API

All endpoints require a valid `Authorization: Bearer <token>` header, and every user only sees their own items.

| Method | Path | Description |
|---|---|---|
| GET | /todos | List the current user's TODOs |
| POST | /todos | Create a TODO |
| PATCH | /todos/{todoId} | Update name, due date or done status |
| DELETE | /todos/{todoId} | Delete a TODO |
| POST | /todos/{todoId}/attachment | Get a presigned S3 URL to upload an image |

## Running it

**Backend**
```
cd backend
npm install
npx serverless deploy --stage dev --region us-east-1
```

**Frontend**

Set the API id and Auth0 values in `client/src/config.ts`, then:
```
cd client
npm install
npm start
```

A Postman collection (`Final Project.postman_collection.json`) is included for testing the API.

## Note

Uses Node.js 12 and 2020 era package versions, so dependencies would need updating before deploying today.
