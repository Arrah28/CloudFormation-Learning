# Plan-Letting Hospitality Platform — CloudFormation + GitHub Actions

Infrastructure as Code for a hospitality booking platform, provisioned on AWS with **CloudFormation** and deployed through a **GitHub Actions** pipeline. Built for the *Software Defined Networks & Edge Services* (505AZ) module at Coventry University; the final design is multi-cloud, with video delivery on Azure.

> **Status: v1 trial build.** Networking foundation and a working Apache web server, deployed by pipeline. The private application tier, RDS, VPC peering and the Azure edge are on the roadmap below.

## What v1 deploys

[`V1/WebServer.yaml`](V1/WebServer.yaml) is a single stack that creates:

| Resource | Detail |
|---|---|
| VPC | `172.18.0.0/16`, DNS hostnames and resolution enabled |
| Subnets | Private `172.18.1.0/24` (AZ 1) and public `172.18.2.0/24` (AZ 2, auto-assign public IP) |
| Internet Gateway + route table | Default route `0.0.0.0/0` → IGW, associated with the public subnet |
| Security group | Inbound 80 and 443 from anywhere |
| EC2 web server | `t2.micro` Amazon Linux in the public subnet; user-data installs and starts Apache |
| Output | `InstancePublicIP` so the pipeline can print where the server is |

| Stack resources | Apache responding |
|---|---|
| ![](V1/PlanLetting-Apache%20Webserver.png) | ![](V1/Functional%20Apache.png) |

## CI/CD

[`.github/workflows/main.yml`](.github/workflows/main.yml) is a manually triggered (`workflow_dispatch`) pipeline that:

1. Checks out the repo.
2. Assumes AWS credentials from repository secrets (`aws-actions/configure-aws-credentials`).
3. Runs `aws cloudformation deploy` against `V1/WebServer.yaml` with `CAPABILITY_NAMED_IAM`.
4. Prints the stack outputs.

`cloudformation deploy` is idempotent, so re-running the workflow updates the stack in place rather than failing on an existing one.

## Deploying it yourself

```bash
aws cloudformation deploy \
  --stack-name PlanLetting-v1 \
  --template-file V1/WebServer.yaml \
  --parameter-overrides KeyName=<your-keypair> \
  --capabilities CAPABILITY_NAMED_IAM

aws cloudformation describe-stacks --stack-name PlanLetting-v1 \
  --query "Stacks[0].Outputs" --output table
```

Or, in GitHub: add `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` and `AWS_SESSION_TOKEN` as repository secrets and run **Deploy CloudFormation** from the Actions tab.

## Known limitations of v1

- **Monolithic stack.** Everything is in one template, so every change has the full blast radius. The final build splits networking, load balancing, application, database and management into separate stacks.
- **Web server is publicly exposed.** It sits in the public subnet with a public IP. Production traffic will go Internet → ALB → private-subnet Auto Scaling Group.
- **No data or management tier yet.**

## Roadmap to the final architecture

```
                 Users
                   │
      ┌────────────┴────────────┐
      ▼                         ▼
  AWS (application)        Azure (video)
      │                         │
  Internet-facing ALB      Azure Front Door
      │                         │
  Private subnets ── ASG   Blob Storage (HLS)
      │
  ┌───┴──────── VPC peering ────────┐
  ▼                                 ▼
VPC A: public services       VPC B: private services
  web tier                     RDS + read replica
                               management EC2 (SSH / ICMP)
```

| Phase | Work |
|---|---|
| Modularise | Split the template into networking, load balancing, application, database and management stacks |
| Application tier | Internet-facing ALB with health checks → Auto Scaling Group in private subnets |
| VPC peering | Second VPC for private services, peered with the public-services VPC |
| Data tier | RDS in VPC B, reachable only from the web tier's security group, plus a read replica |
| Management tier | Hardened EC2 in VPC B for SSH and ICMP diagnostics into private resources |
| Azure edge | Blob Storage for HLS video assets, delivered globally through Azure Front Door, configured with the Azure CLI |
| Pipeline | Extend the GitHub Actions workflow to deploy each stack in dependency order |

## What I've learned so far

- **Explicit `DependsOn` matters** when CloudFormation can't infer ordering — the default route has to wait for the IGW attachment or the stack fails midway.
- **`!GetAZs` + `!Select` keeps templates region-agnostic** instead of hard-coding availability zone names.
- **Pipelines make IaC honest.** Once the deploy runs from a clean runner with only the repo and secrets, anything that only worked on my laptop shows up immediately.
