---
title: From Kubernetes to a Cloudflare + k3s Hybrid Solution
published: 2025-02-27
description: 'A journey on how I abstracted most of my clusters logic using Cloudflare services.'
image: ''
tags: [kubernetes, learning, devops, cloudflare]
category: 'documentation'
draft: true 
lang: 'en'
---

From Kubernetes to a Cloudflare Hybrid Solution
# Introduction
Today, I gave myself the challenge to deploy full Saas on a low-cost high-available cluster.<br />
<img src="/image/sideeye.jpg" width="350" alt="side eye meme" />
Yes, this seems quite challenging...<br />
Plus, I wanted to make this infrastructure easily maintainable and provisionnable with an IaC.

# Productive tools I used during this lab
## Claude.ai
On my way to design the cluster, I decided to test this new AI called claud.ai.<br />
To be honest, it blew my mind. It's accuracy and skills to draw graphs and gather accurate information was by far superior to what I was provided with paid copilot, chat gpt or gemini plans.
## plane.so
Amazing alternative to jira, it has everything you need without the jira headache.
## mermaid.js.org
Help me create flowcharts diagrams.

# Comparing options
I had no idea where to begin so I started to compare different cloud providers.
- gcp
- aws
- aks
- openshift
- and traditional PaaS (infomaniak, vercel, linode, etc..)
- vps + coolify

:::caution[]
GCP, AWS and AKS were a no go since I wanted to scale rapidly without having to bankrupt and easily manage my cluster alone.

<img src="/image/awsbill.jpg" width="250" alt="aws bill" />
:::

:::warning[]
<abbr title="Switzerland's largest web-hosting company">Infomaniak</abbr> looked nice, but the price pool was too high for the offered service.<br />
Domains + vps + Infomaniak services came around 700$ a year for basic offer with 2 major drawbacks.
- Servers are only present in switzerland
- All in one packages always gives me a lot of uncertainties about scaling and can get pricy when wanting to expand my business in different fields.
:::

:::warning[]
I have low trust in Vercel and other traditional Paas because of their history with high-billing scandals due to abrupt scaling (we all remembered that 100k vercel bill 👀).
:::

:::note[]
Redhat OpenShift Container Platform looked interesting to manage my vps cluster but not yet needed at this point of time.
:::

:::tip[]
I felt kinda left on my own, so I decided to go with a traditional vps managed with coolify.<br />
Coolify is "An open-source & self-hostable Heroku / Netlify / Vercel alternative". It would help me manage tools and software integration easily.<br />
This is one of the ways of deploying an app that I haven't tested yet.<br />
As I have already tested all of the above during my career.
:::

## Pricing
From claud.ai with 5k budget max.:
- aws: 4.8k
- gcp: 4.75k
- aks: 4.9k
- hybrid Plan (Infomaniak + Cloudflare + AWS/GCP): 4.6k
- multi-Cloud Plan (Budget Maximized): 4.95k
- openshift: 4.8k
- okd: 4.6k

# Designing the architecture
## Kubernetes
1. First I decided to design the whole kubernetes architecture, beacuse I needed to list my needs and to think about how I want my <abbr title="high available">ha</abbr> infrastructure to run properly.<br/>Initially, I designed a traditional multi-region Kubernetes architecture:
- Separate K8s clusters in EU and US
- High-availability control plane with etcd clustering
- Self-managed databases with replication (I was thinking CockroachDB)
- Redis for caching and Kafka for messaging
- Multiple load balancers for redundancy
<img src="/image/k8s-ha.png" alt="k8s ha" />Of course a high available inrastructure needs to manage it's rollouts and deployments strategies properly.<img src="/image/k8s-deploy-pattern.png" alt="k8s deploy pattern" />


2. Next step was to design how data was going to be handled inside my cluster (with caching, message queues)<img src="/image/k8s-data.png" alt="k8s data" />

3. Last step was to get a broader view of these components and interactions.<img src="/image/k8s-integrated-arch.png" alt="k8s integrated architecture" />

:::important[Challenges]
The diagrams looked amazing - multiple availability zones, leader election, ....<br />
But managing all this infrastructure would be too much for our small team, and costs would quickly approach our budget limit.
* Required expertise in Kubernetes, networking, and database management
* Difficult to scale globally within budget constraints
* High maintenance burden for a small team
* Limited geographic reach
:::

## Cloud flare
Managing a whole cluster on my own is a burden, and useless.<br />
I remembered Joshua (a friends of mine) has a history of using cloudflare services ([blog](https://macawls.dev/blog)). I explored replacing our entire infrastructure with:

- Cloudflare Workers for computation
- Workers KV for data storage and caching
- R2 for object storage
- Cloudflare global network for distribution

This approach would cost around $1,000-1,400/year - a fraction of our budget! Plus, we'd instantly get global distribution across 275+ data centers, whith **FREE** plans and **clear scaling prices**.<br />
This will help me leverage a lot of difficulties, but I was hesitant to give up control of our core infrastructure.<br />
Some of our workloads didn't seem suitable for the pure serverless model, especially stateful components.
Anyways, I did the same diagrams as previously using only cloudflare services:
<img src="/image/cf-integrated-arch.png" alt="cloudflare integrated architecture" />

:::tip[Benefits]

Global presence out of the box

Significantly lower cost

No infrastructure to manage

Excellent scaling capabilities
:::

:::warning[Limitations]

Less control over the infrastructure

Some workloads may not fit the serverless model

Stateful applications could be challenging
:::

## Hybrid
After weighing the options, I realized we could combine the best of both worlds using  cloudflare + k3s (lightweight kubernetes):

1. Keep core applications on Kubernetes
- Maintain two small K8s clusters (EU and US)
- Run our stateful services and main application logic here
- Use service mesh for internal communication

2. Leverage Cloudflare at the edge
- Cloudflare as our global CDN and edge network
- Workers for authentication, rate limiting, and edge logic
- KV for global caching (huge performance boost!)
- R2 for cost-effective object storage

3. Connect them securely
- Cloudflare Tunnels to securely expose our clusters

<img src="/image/hybrid-integrated-arch.png" alt="k8s integrated architecture" />

# Conclusion
This hybrid approach gives us the control we want over our core infrastructure while leveraging Cloudflare's global network for performance, security, and scale. We get enterprise-level capabilities without the operational burden.

The beauty of this setup is that we can start small and grow incrementally. Our initial implementation will focus on our core application in two regions, with Cloudflare handling the global distribution. As we grow, we can add regions or shift more logic to the edge as needed.

For a startup with limited resources but global ambitions, this architecture provides the perfect balance of control, cost-efficiency, and scale.