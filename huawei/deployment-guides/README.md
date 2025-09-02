# 🚢 Deployment Guides: Step-by-Step Suffering

[← Back to Huawei Guides](../) | [🏠 Home](../../)

## ⚠️ IMPORTANT: Follow These In Order!

These guides are numbered for a reason. Skip steps at your own peril.

## The Deployment Journey

| Step | Guide | What You'll Learn | Time to Complete | Pain Level |
|------|-------|-------------------|------------------|------------|
| 1 | [SWR Docker Push](./01-swr-docker-push.md) | How to authenticate with Huawei's registry | 2 hours | ⭐⭐⭐⭐⭐ |
| 2 | [ECS Deployment](./02-ecs-deployment.md) | Deploy containers that actually run | 3 hours | ⭐⭐⭐⭐ |
| 3 | [Function Graph](./03-function-graph-deploy.md) 🚧 | Serverless deployment | TBD | ⭐⭐⭐ |
| 4 | [Production Setup](./04-production-setup.md) 🚧 | Making it not break at 3 AM | 5 hours | ⭐⭐⭐⭐⭐ |

## Before You Start

### Prerequisites Checklist
- [ ] Huawei Cloud account with credits
- [ ] Docker installed locally
- [ ] Basic understanding of containers
- [ ] Coffee (lots of it)
- [ ] Backup plan for when this doesn't work

### Required Access
- [ ] SWR (Software Repository for Container)
- [ ] ECS (Elastic Cloud Server)
- [ ] VPC configured
- [ ] Security groups that aren't completely open (please)

## Common Issues Quick Links

- [SWR Authentication Failed](./01-swr-docker-push.md#authentication-nightmare)
- [Docker Push Timeout](./01-swr-docker-push.md#the-push-where-dreams-go-to-die-slowly)
- [Container Won't Start](./02-ecs-deployment.md#container-wont-start-surprise)
- [Permission Denied Errors](./02-ecs-deployment.md#the-permission-wars)

## Emergency Contacts

When these guides fail (not if, when):
- Huawei Support: [Good luck with that]
- Justin: [Check We Link, probably crying]
- Stack Overflow: [Your real support team]