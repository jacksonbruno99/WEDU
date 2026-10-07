# WEDU: Home, handled. Home services for Puducherry

*WEDU (written WEடு, say "we do") plays on வீடு (veedu), Tamil for "home".*

*AWS Zero to Shipped entry.*

- **App category:** Commercial Potential
- **Lane:** Startups
- **Tags:** #commercial-potential #startups
- **Live application:** [https://lewflit6u7.execute-api.ap-south-1.amazonaws.com/](https://lewflit6u7.execute-api.ap-south-1.amazonaws.com/)

## The problem

Finding a reliable electrician, plumber or cleaner in Puducherry still runs on word of mouth and phone calls. Big platforms focus on large metros, so smaller cities are underserved, for customers and for the local providers who could serve them.

## What I shipped

WEDU is a working booking flow, live on AWS at the link above:

- Browse local providers and filter by service
- Open a booking form, choose a time slot and send a request
- Server-side validation: Indian mobile number format, allowed time slots, size limits
- A public "recent requests" feed that is anonymised: it never shows names, phone numbers or notes
- English and Tamil interface (English by default)
- A clear demo notice: providers are sample data and are **not** contacted yet. Requests are saved so the flow can be tested end to end.

Tip for testing: enter any name and the demo number 9876543210.

## How I used my coding agent (and proof of connection to AWS)

I am new to this agent-plus-AWS workflow, so I set it up step by step and kept proof at each stage.

1. **Signed in the AWS CLI** with `aws login` (region `ap-south-1`, Mumbai).
2. **Set up the AWS Agent Toolkit.** The setup detected Codex on my machine and installed 24 AWS skills for it (deployment, serverless, CloudFormation, IAM and more).
3. **Connected Codex to the AWS MCP Server** through the MCP Proxy for AWS, using the SigV4 credentials from my CLI session. `codex mcp list` shows `aws-mcp` as enabled.
4. **Verified the connection.** From inside Codex I ran `aws sts get-caller-identity`, and Codex reported the account and the region (`ap-south-1`) it was working against. *(Screenshot: Codex session with the account number blurred.)*

**How the work was split:**
- I used an AI assistant (Claude) in a chat to draft the first version of the app and the AWS SAM template.
- I used **Codex, connected to my AWS account**, to build and deploy the stack, diagnose failures, and apply the later page redesign and wording fixes.
- I approved every command Codex wanted to run and checked its inputs before approving.

## Architecture

Browser -> Amazon API Gateway (HTTP API, throttled) -> AWS Lambda (Node.js 22) -> Amazon DynamoDB (on-demand)

- **One Lambda** serves both the single-page UI and the JSON API, so there is one deployable unit.
- **DynamoDB on-demand** means no capacity planning and near-zero cost when idle. *(Screenshot: a booking saved in the DynamoDB table, name and phone blurred.)*
- **API throttling** (50 requests per second, burst 100) protects the public demo from abuse.
- **Least-privilege access:** the function can only read and write its own table.
- **Infrastructure as code** with AWS SAM: the whole stack deploys, or is deleted, with one command.
- **Privacy by design:** the public feed shows only service, provider, area and slot. The page loads no third-party scripts or trackers.

## Deployment journey

Getting from zero to a live URL took several real problems, and each one taught me something:

1. **Codex could not reach AWS.** Its first commands failed with a proxy error (`Failed to connect to proxy URL 127.0.0.1:9`). The cause was Codex's sandbox, which blocks network access by default. I restarted it with network access enabled for the workspace sandbox, and the same commands then worked and returned my account and region.
2. **SAM could not write its settings.** `sam deploy --guided` failed because the sandbox blocked SAM's config folder. Codex noticed, explained it, and asked my approval to rerun the command outside the sandbox.
3. **Interactive prompts were fragile.** Answering SAM's guided questions one at a time through an agent was easy to get wrong, and I accidentally interrupted one run. I switched to a single non-interactive deploy command with explicit flags (stack name, region, IAM capability, managed artifact bucket).
4. **It shipped.** That deploy created the stack in `ap-south-1` in about two and a half minutes, and printed the public URL.
5. **Tested for real.** I opened the URL in a private window, made a booking, and confirmed the item in the DynamoDB console.
6. **Honesty fix.** On review I noticed the first page text implied providers are notified right away, which is not true yet. I rewrote that wording, added a demo-only notice to the booking form, and redeployed.

**Lessons:** check what your agent's sandbox can reach, prefer one explicit command over many interactive steps, and make the product's wording match what it actually does.

## Why this slice, and what comes next

This is the first vertical slice of a plan I am building as a real business: a set of microservices (users, providers, bookings, payments, notifications) on Amazon ECS Fargate with RDS PostgreSQL, and SQS and SNS for booking and notification events. For this hackathon I chose the smallest serverless path that proves the core flow end to end and ships publicly.

## Where it's headed (Startups lane)

**Who it is for:** homeowners and renters in Puducherry, and the independent local providers who today depend on word of mouth.

**Plan:**
1. **Start narrow.** Launch with a few services (electrician, plumber, cleaning, AC repair) in a few neighbourhoods, and onboard the first providers personally.
2. **Make requests real.** Add provider sign-up and login, and send booking alerts by SMS or WhatsApp so providers actually receive requests.
3. **Earn from bookings.** Charge a small commission on completed bookings, with payments handled in the app.
4. **Grow from proof.** Expand services and nearby towns only after repeat bookings show the model works.

**Where it stands today:** a working demo with sample providers. No real bookings have been fulfilled yet, and I would rather say that plainly than overstate it.

## Try it

**[https://lewflit6u7.execute-api.ap-south-1.amazonaws.com/](https://lewflit6u7.execute-api.ap-south-1.amazonaws.com/)**. Use the demo number 9876543210. Providers and prices are sample data.
