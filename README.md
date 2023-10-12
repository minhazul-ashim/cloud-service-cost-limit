<!-- HEADER -->
<br />
<div align="center">
  <a href="https://revant.io">
    <img src="images/logo.png" alt="Logo" width="80" height="80">
  </a>

  <h1 align="center">Cost Limit for AWS</h1>

  <p align="center">
    A Collection of CDK Constructs to Deploy Cost-Aware Self-Limiting Resources
    <br />
    <br />
    <a href="https://revant.io">Website</a>
    ·
    <a href="">Docs</a>
    ·
    <a href="https://github.com/revant-io/cdk-cost-limit/issues/new">Request Feature</a>
  </p>
</div>

## Overview

### What?

This package lets you set spending limits on AWS. While existing AWS solutions merely alert, this library disables resources, using non-destructive operations, when budgets are hit.

### Why?

Every month, companies and individual contributors face surging cloud bills[^1] due to mistakes in their application code, misunderstandings of services pricing model, or even malicious activities. This library shields you from such events.

### How?

This library includes an [Aspect](https://docs.aws.amazon.com/cdk/v2/guide/aspects.html) and a collection of [AWS CDK Level-2 Constructs](https://docs.aws.amazon.com/cdk/v2/guide/constructs.html#constructs_lib). They deploy [additional resources](./docs/constructs.md#per-service-level-2-constructs) to compute real-time spending and halt resources when budgets are met (e.g. Lambda Functions reserved concurrency is set to 0)

### Tradeoffs

- **🚧 Availability** - Disabled resources impacts your application availability
- **💰 Costs** - Tracking spending and halting resources incur additional costs
- **🏎️ Performance** - Some implementations negatively impact performances

> [!IMPORTANT]
> 🧑‍💻 We're actively working on reducing cost and perf impacts of this library! We'll [keep you posted](./docs/tradeoffs.md) as we minimize and eventually remove complitely those tradeoffs

## Getting started

### Prerequisites

- CDK >= 2 - only applications deployed using AWS CDK can use this library. It is not a standalone application.
- Node.js >= 14.15.0 - even those working in languages other than TypeScript or JavaScript (see [AWS CDK Documentation](https://docs.aws.amazon.com/cdk/v2/guide/getting_started.html#getting_started_prerequisites))

### Installation

```sh
npm install -s @revant-io/cdk-cost-limit
```

### Usage guide

Using this library is as simple as importing the `CostLimit` Aspect and using it on any node to apply a budget on the node and all of its children.

```typescript
import { Aspects } from "aws-cdk-lib";
import { CostLimit } from "@revant-io/cdk-cost-limit";

Aspects.of(anyConstruct).add(new CostLimit({ budget: 1000 }))
```

Using an aspect allow setting a budget on a specific construct of the application, or even a whole stack. Multiple budgets can be set up at once.

```typescript
import { App, Aspects, Stack } from "aws-cdk-lib";
import { Instance } from "aws-cdk-lib/aws-ec2";
import { Function } from "aws-cdk-lib/aws-lambda";

import { CostLimit } from "@revant-io/cdk-cost-limit";

const app = new App();
const stack = new Stack(app, "MyStack");

const myEC2Instance = new Instance(stack, "MyEC2Instance");
const myFirstLambdaFunction = new Function(stack, "MyFirstLambda");
const mySecondLambdaFunction = new Function(stack, "MySecondLambda");

// Sets one global budget on all resources within MyStack of $10,00
Aspects.of(stack).add(new CostLimit({ budget: 1000 }));

// Sets another budget on MyFirstLambda of $2,00
Aspects.of(myFirstLambdaFunction).add(new CostLimit({ budget: 200 }));

// Sets a final budget on MySecondLambda of $5,00
Aspects.of(mySecondLambdaFunction).add(new CostLimit({ budget: 500 }));
```

Once a budget is hit, all resources within the construct on which this budget has been applied are disabled to prevent further cost increase. In the above example:

- once `MyFirstLambda` incurred costs reach $2,00, the lambda function is disabled
- once `MySecondLambda` incurred costs reach $5,00, the lambda function is disabled
- once `MyStack` incurred costs reach $10,00 - in this specific case, the addition of incurred costs from `MyEC2Instance`, `MyFirstLambda` and `MySecondLambda` - all resources within `MyStack` are disabled

### Supported services/resources

While `CostLimit` aspect can be applied on any node type, budget capabilities are only enabled on a subset of resources listed below. Cost generated by resources other than the one specified are not taken into account with regards to the budget limit. Only resources listed below are disabled once budget is reached.

- [✅ `AWS::Lambda::Function`](./docs/lambda.md)
- [🚧 `AWS::EC2::Instance`](./docs/ec2.md)

### Lifecycle

- **🧑‍💻 Develop** - Use this library to set your budget limit. Look at the [catalog documentation](./docs/constructs.md) to learn how to implement a budget. You can update budget amount and location anytime.

- **🤖 Limit** - Budgeted resources will disable automatically when budget is reached. Have a look at [each service documentation](./docs/constructs.md#per-service-level-2-constructs) to understand how resource are disabled.

- **🔄 Restore**

  - **📆 Automatically** - At the end of each month, resources that have been disabled are automatically re-enabled

  - **⚒️ Manually** - You might want to increase budget and keep on using disabled resources in the current month. In order to do so, you need to follow [each service documentation](./docs/constructs.md#per-service-level-2-constructs) procedure to identify and manually re-enable affected resources

### Design principles

See the [design principles](./docs/design-principles.md) section to learn more about the architecture of the library.

[^1]: 📖 Have a look at those [AWS billing horror stories aggregated by Victor Grenu](https://unusd.cloud/blog/post-5/)!
