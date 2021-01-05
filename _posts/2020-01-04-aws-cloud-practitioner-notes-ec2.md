---
layout: post
title: AWS cloud practitioner notes - EC2
date: 2021-01-04 01:06:05.000000000 -03:00
image: 
type: article
published: true
status: published
categories:
- aws
- ccp
tags:
- aws,
- ec2,
- instance,
- resources,
- modules,
- service,
- messages,
- cloud,
- on-demand,
- ec2 instance,
- modules ec2,
- high performance,
- aws cloud,
- cloud computing,
- instance types
---

This post aims to expose the path I took to take the AWS cloud practitioner
certification. This post was inspired by {% cite preparing_for_ccp --file 2020-01-04-aws-cloud-practitioner-notes-ec2 %}
and some of the resources I am going to share here were taken from his post as
well my own notes from the free official training {% cite aws_free_training --file 2020-01-04-aws-cloud-practitioner-notes-ec2 %}.

The CCP exam is the first certification in the AWS exam list, as such, if you
have some experience with AWS it should be no problem to complete it. Therefore, the material presented is valuable in the details and conventions used.

## Course overview

- Lessons includes videos and additional resources such as links
- For each module there is a quiz to check on the material knownledge

In the end there will be a 30 question assesment of the exam.

## Module 1 - Introduction AWS

The first module goes on the basics of cloud computing, what it is, and AWS
pay as you go model, remember:

1. Cloud computing is: On-demand delivery of IT resources and applications through the internet with pay-as-you-go pricing
2. Pay on-demand: The aggregated cloud usage from a large number of customers results in lower pay-as-you-go prices.

## Module 2 - Introduction EC2 (CaaS)

EC2 takes care of the part of setting up physical servers. Such as buying them,
setting them up in a datacenter (or even rent a hack to put it in) and the
security part as well. You can spin up a new server or take it down as you will.

Multitenancy: AWS EC2 uses the server as a host which can have multiple
servers (EC2 instances) on it.

Provision a EC2 instance with the following OS's:
 
- Linux
- Windows

Besides that you are in the control of network aspect of the EC2 as well.

In short, EC2 is:

- Flexible
- Reliable
- Scalable

Using EC2 you are responsable for:

1. Set up and manage your instances
2. Responsible for patching instances
3. Setting up scale
4. Architecting in a highly available setup

### Modules 2 - EC2 Instance types

Each EC2 instance type is grouped under an instance family. Instance types offer
different combinations of storage, memory, processing power.

For the following list, consider the term **resources** as the group of: compute, memory and networking.

- General purpose (Balanced resources: Web servers/code repository)
- Computer optimized (Compute intensive tasks: Gaming servers, high performance computing - HPC, Scientific modeling)
- Memory optimized (Memory intensive tasks)
- Accelerated computing (Floating point number calculations, Graphics processing, Data pattern matching, hardware accelerators)
- Storage optimized (High performance for locally stored data)

### Modules 2 - EC2 pricing

- On-demand
- Savings plans (low prices, in exchange for commitments in one or three year term, leading up to 72% of saving compared to on-demand)
- Reserved instances (predictable usage up to 75% of savings compared to on-demand)
- Spot instances (Ideal for flexible workloads - therefore AWS can terminate the instance at any time)
- Dedicated  (No shared tenancy of the host)

### Modules 2 - EC2 scaling

1. Vertical scaling: EC2 are resizable as well, you can upgrade or downgrade hardware
aspects of the instance such as: processing power, memory and HD.
2. Horizontally - creating new intances and adding them to the pool

- Dynamic scaling: responds to changing demand- Auto scaling.
- Predictive scaling: schedule the right number of EC2 based on predicted demand.

### Modules 2 - Elastic load balancing (ELB)

Load balancing distributes incoming application traffic accross
multiple resources and its built on regional construct, it runs on regional level.

- High performance
- Cost-efficient
- Highly available
- Automatically scalable

### Modules 2 - Message and queueing

Applications that communicates directly are defined as: Tighlty coupled architecture. If a single compnent fails or changes, it causes issues
to the entire system.

```
                    talks to
Application A --------------------> Application B
```

Loosely coupled architectures are defined as single failure won't cause cascading failures.

```
              sends to                  process
                      _________________
Application A ------> | message queue | <-------- Application B
                      |_______________|
```

For decoupled solutions, AWS provides two services:

1. AWS SQS (simple queue service)

SQS queues are places where messages are storaged until they are processed.

- send messages (data within a message: payload. It is protected until delivery)
- store messages
- receive messages
- at any volume (without loosing messages)

2. AWS SNS (simple notification service)

SNS is a channel for messages to be delivered, pub-sub pattern.Subscribers can be web servers,email, lambda functions, etc.

### Modules 2 - Additional compute services

#### AWS lambda

AWS lambda is a service that allows you to upload your code
to be executed. The code is executed in a managed environment.

- lambda is designed to run code in 15 minutes
- automatically scale

#### AWS ECS (Elastic container service) or EKS (Elastic Kubernetes service)

ECS is designed to orchestrated your containers at scale, without
the hassel to manage your own cluster. ECS can run on top of EC2, so then
you can manage details under the hood.
Also AWS offers Fargate to deploy containers, which is a managed service
that removes the access to low level components such as the operational system. In short:

1. ECS on EC2 you manage
2. ECS on Fargate AWS manages

## References

{% bibliography --cited_in_order --file 2020-01-04-aws-cloud-practitioner-notes-ec2 %}