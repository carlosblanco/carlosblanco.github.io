---
layout: post
title: "The DLQ Doesn't Wait: Engineering for Trust Over Deadlines"
date: 2021-06-10
author: Carlos Blanco
categories: [engineering-culture, backend]
tags: [nubank, reliability, culture, observability, ci-cd]
---

There's a line of thinking in software engineering that treats bugs as interruptions — things that pull you away from the "real work" of shipping features. At Nubank I found the opposite to be true. When something lands in the dead letter queue, that *is* the real work. The DLQ is a centralized queue shared across services where messages end up when processing fails. At any given moment it represents real transactions, real customers, real money that didn't move the way it was supposed to. The culture here is that engineers don't triage it at the end of the sprint or escalate it up a chain — they jump on it. Before the feature, before the deadline, before the next standup. You open the logs, trace the failure, open a Jira ticket yourself, and start fixing.

What makes this actually work — and not just a nice-sounding value that collapses under pressure — is the infrastructure underneath it. The observability stack at Nubank is genuinely world-class. Distributed tracing, structured logs, stacktraces linked to kafka payloads and the ability to replay failed messages on demand, dashboards that correlate events across services with enough precision that you can usually pinpoint a failure in minutes rather than hours. Pair that with CI/CD pipelines that let you go from a fix to production without a deployment ceremony, and troubleshooting stops feeling like a disruption to your day. It becomes a loop you can close quickly and cleanly. The investment in tooling is what gives engineers the confidence to own incidents without needing to ask for permission or wait for a war room.

The deeper thing this reflects is trust. Nubank trusts engineers to self-organize around problems, create their own tickets, and report back at standup with status rather than waiting to be assigned. That's a small cultural detail with big implications — it means the person closest to the code is also the person making the call on priority. No handoff, no delay, no context lost in translation. We'd often walk into a standup having already found the issue, opened the ticket, shipped the fix, and confirmed it in production. The DLQ was back to zero before anyone formally asked about it. That's the kind of reliability culture that's hard to document in a process guide because it's not really a process — it's just what good engineering looks like when you trust the people doing it.
