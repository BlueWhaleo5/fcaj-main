---
title: "Blog 1 - The Journey of Amazon SQS"
date: 2026-07-30
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
includeInReport: true
---

# The Journey of Amazon SQS: The First Service That Shaped the AWS Cloud Empire

When people think of AWS, the "household names" that usually come to mind are EC2 for virtual servers, S3 for storage, or RDS for databases. But do you know which AWS service actually launched first, globally?

It wasn't a compute service, and it wasn't a storage service. It was **Amazon SQS (Simple Queue Service)** — a service that sounds almost too modest for the role: a message queue.

Today, let's look at what made SQS the "first brick" of a trillion-dollar cloud empire, and why it still matters so much in modern software architecture.

## 1. The 2004 Problem: When Systems "Doze Off" Together

In the early 2000s, web applications were typically built as monoliths, or with components calling each other directly (synchronously).

Picture a simple scenario on an e-commerce site like Amazon:

1. The customer clicks "Place Order."
2. The system processes the credit card payment.
3. The system sends a confirmation email.
4. The system updates inventory in the warehouse.

If all four steps run one after another, synchronously, a single slow step — say, the email service hits network congestion, or the payment gateway takes 5 extra seconds to respond — freezes the entire site. Frustrated customers cancel their orders, and the server buckles under the backlog.

This created a real need: a "relay station" that lets components talk to each other asynchronously. Team A sends a message and its job is done; Team B picks the message up and processes it whenever it's ready — neither has to sit and wait for the other.

That exact need is what gave birth to Amazon SQS in 2004 — even before the AWS brand itself officially launched in 2006.

## 2. How Does SQS Work?

The easiest mental model for SQS is the order rail at a coffee shop:

```
[ Customer / Producer ]
          |
          v
[ Order Slip on the Rail / SQS Queue ]
          |
          v
[ Barista / Consumer ]
```

* **Producer (sender):** The cashier takes the order and prints an order slip (sends a message to the queue).
* **SQS (the queue):** The order rail. It holds the slips in order, safely, with no risk of one blowing away.
* **Consumer (receiver):** The barista. They pull slips down one at a time to work on. The cashier never has to stand around waiting for one drink to finish before taking the next customer.

## 3. SQS's Two Main "Weapons"

SQS offers two queue types for two different problems:

### Standard Queues

* **Throughput:** Effectively unlimited messages per second.
* **Behavior:** Guarantees at-least-once delivery. Message order can occasionally shuffle slightly.
* **Best for:** High-volume, high-speed workloads where strict ordering doesn't matter much (e.g. image processing, notification emails).

### FIFO Queues (First-In-First-Out)

* **Throughput:** Lower than Standard queues.
* **Behavior:** True to its name — "first in, first out" — guaranteeing messages are processed in exact order, exactly once.
* **Best for:** Financial transactions, banking, flight booking — anywhere the order of steps determines whether the data is even correct.

## 4. Why Is SQS a Developer's "Lifesaver"?

* **Auto-scaling:** Whether your app receives 10 messages a day or 10 billion, SQS absorbs it without you ever pressing an "upgrade RAM" button or worrying about the infrastructure falling over.
* **Decoupling:** Lets microservices operate independently. If the email service goes down, messages stay safely in SQS. When the service recovers, it simply picks up where it left off — no customer data lost.
* **Dead Letter Queue (DLQ):** When a message keeps failing after repeated processing attempts, SQS automatically routes it to a "dead letter queue" so a developer can investigate the root cause without it blocking the rest of the messages.

## Conclusion

As the service that opened AWS's story, Amazon SQS is the clearest proof of Amazon's design philosophy: start from the most fundamental, simplest problem — but make it work with extreme reliability at massive scale.

It isn't as flashy as today's AI or Big Data tools, but SQS still quietly runs through the bloodstream of millions of applications worldwide, every single second.

### Original Post

* **Link:** [The Journey of Amazon SQS](https://www.facebook.com/groups/awsstudygroupfcj/?multi_permalinks=2225888418176118&notif_id=1785400015854661&notif_t=feedback_reaction_generic&ref=notif)
