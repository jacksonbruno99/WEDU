# WEDU (WEடு): Home, handled.

WEDU is a mobile-friendly home services demo for Puducherry. Browse sample local providers, filter by service, choose a time slot, and save a booking request. The interface is available in English and Tamil.

**Live application:** [https://lewflit6u7.execute-api.ap-south-1.amazonaws.com/](https://lewflit6u7.execute-api.ap-south-1.amazonaws.com/)

## Demo behavior

- Providers and prices are sample data across electrician, deep cleaning, plumbing, AC repair, painting, and salon at home.
- Booking requests are saved in DynamoDB, but providers are not contacted yet.
- The recent requests feed is anonymised and does not show names, phone numbers, or notes.
- Do not enter real personal details. For a demo booking, use any name and the number `9876543210`.

## Architecture

Browser -> Amazon API Gateway (HTTP API) -> AWS Lambda (Node.js 22) -> Amazon DynamoDB

The Lambda serves both the WEDU page and JSON API. AWS SAM defines and deploys the application infrastructure.

## Deploy

Prerequisites: AWS CLI authenticated to your account, AWS SAM CLI, and Node.js 22.

```bash
sam build
sam deploy --stack-name pondy-services --region ap-south-1 --capabilities CAPABILITY_IAM --resolve-s3 --no-confirm-changeset --no-fail-on-empty-changeset
```

The deployment output includes the `LiveUrl` for the API Gateway stage.

## API smoke test

```bash
URL="https://lewflit6u7.execute-api.ap-south-1.amazonaws.com"
curl "$URL/api/providers"
curl -X POST "$URL/api/bookings" -H "content-type: application/json" \
  -d '{"providerId":"p1","name":"Demo User","phone":"9876543210","slot":"Morning (9-12)"}'
curl "$URL/api/bookings"
```

The POST request saves demo data only; no provider notification is sent.

## AWS Builder submission

This project is submitted in the **Commercial Potential** category and **Startups** lane. The Builder post, including the workflow, screenshots, architecture, and product roadmap, is in [`BUILDER_POST.md`](BUILDER_POST.md).

## Troubleshooting

- `sam: command not found`: install the AWS SAM CLI.
- Deployment permission errors: check access for CloudFormation, Lambda, API Gateway, DynamoDB, S3, and IAM role creation.
- HTTP 500 responses: inspect the Lambda logs with `sam logs -n AppFunction --stack-name pondy-services --tail`.

## Cleanup

To remove the stack after the demo:

```bash
sam delete --stack-name pondy-services --region ap-south-1
```
