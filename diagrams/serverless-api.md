# Serverless API: API Gateway, Lambda, DynamoDB

Companion guide to `serverless-api.drawio`. Technical audience.

## Flow

1. **Client → API Gateway** (HTTPS). The client holds a JWT issued by the user
   pool; the request carries it in the `Authorization` header.
2. **Cognito → API Gateway** (verify JWT). The authorizer validates the token
   at the edge, so the function never runs for an unauthenticated request and
   you are not billed for it.
3. **API Gateway → Lambda** (invoke). One function per route, or one function
   with a router inside it — both are defensible; the split matters when cold
   start or IAM scope differ per route.
4. **Lambda → DynamoDB.** Direct SDK call under the function's execution role.
   No connection pool exists and none is needed, which is the reason DynamoDB
   pairs with Lambda where a relational database needs a proxy.
5. **API Gateway → CloudWatch** (logs, dashed). Access logs and metrics, out of
   band from the request path.
6. **Lambda → SQS** (failures, red dashed). The dead-letter queue for
   asynchronous invocations after retries are exhausted. Something has to drain
   it; an unmonitored DLQ is a silent data-loss channel.

## Services

| Service | Purpose | The decision it carries |
|---|---|---|
| **Cognito** | User pool, JWT issuance and verification | Authorizer at the gateway, not in the handler — unauthenticated traffic costs nothing |
| **API Gateway** | HTTPS entry point, routing, throttling | REST vs HTTP API is a cost and feature trade; throttle limits are your first backpressure control |
| **Lambda** | Request handler | Timeout and memory are architectural constraints; the handler must be idempotent because retries are the platform's default |
| **DynamoDB** | Primary store | Single-table design and access patterns must be settled before launch; on-demand until traffic is predictable |
| **CloudWatch** | Logs, metrics, alarms | Budget alarms are an availability control here, not finance hygiene |
| **SQS (DLQ)** | Async failure sink | Needs an owner and an alarm, or failures accumulate unseen |

## Key design decisions

- **Authorisation at the gateway.** Moving JWT verification into the handler
  means paying for every unauthenticated request and duplicating the check per
  function.
- **No VPC attachment.** DynamoDB is reached over the AWS network without
  placing the function in a VPC, which avoids the ENI cold-start penalty. Add a
  VPC only when something in the path genuinely requires it.
- **Idempotency is not optional.** Lambda retries on error and API Gateway
  clients retry on timeout, so a handler that is not idempotent will double-write
  under exactly the conditions you cannot reproduce.
- **The DLQ is part of the design, not an afterthought.** It is drawn on the
  diagram because it needs an alarm and a person, the same as any other queue.

## Rendering

- Source: `serverless-api.drawio` — open in draw.io desktop or app.diagrams.net
- Export: `serverless-api.drawio.png` — the double extension means the diagram
  XML is embedded, so the PNG itself reopens in draw.io and stays editable

Re-export after editing the source:

```bash
/Applications/draw.io.app/Contents/MacOS/draw.io -x -f png -e -b 10 \
  -o serverless-api.drawio.png serverless-api.drawio
```

## Layout notes

Two constraints drove the placement, both found by reviewing the exported PNG:

- **Main-lane labels sit above their icons.** Every main-lane node has an edge
  leaving its bottom, and a label below the icon is wider than the icon, so a
  bottom-exiting edge draws straight through the text.
- **All secondary services share one lower lane inside the cloud boundary.**
  Cognito, CloudWatch and the DLQ hang off the bottom of the lane above them,
  which keeps every vertical edge in its own corridor with no crossings.
