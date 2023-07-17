---
layout: post
title:  "Hyrums' Law and Data"
tags: [books, data-engineering]
---


My recent reading of Zhamak Dehghani's [Data Mesh](https://www.amazon.co.uk/Data-Mesh-Delivering-Data-Driven-Value/dp/1492092398) has illuminated some remarkable similarities between its principles and the methodologies we've been striving to implement at GoCardless, primarily concerning [Data Contracts](https://www.amazon.co.uk/Driving-Data-Quality-Contracts-comprehensive/dp/1837635005) (a valuable book penned by our own Andrew Jones).

The cornerstone of our parallels lies in the concept of decentralised data ownership. Historically, GoCardless relied on a centralised data platform managed by specialised data teams, such as our "Data Platform" team, responsible for providing a robust data platform for the entire company, and the "Business Intelligence" team, tasked with crafting Looker dashboards for various divisions. This arrangement inevitably led to a disconnection where data producers remained oblivious to the identities of data consumers, while consumers assumed, quite erroneously, that the data responsibility lay with the BI and Data Platform teams.

Our adoption of Data Contracts is shifting this dynamic significantly. Rather than being the custodians of all data, we're now building a framework for data generators to create and manage their personalised data platforms and govern access. Supported by a Backstage-based data catalogue, data consumers can now easily locate specific data types and directly engage with the data owners to obtain access.  This is also discussed in Zhamak's book, as there is a need for self-service data infrastructure tooling to provide standardised golden paths for the rest of the organisation.

![ExtractPublish](/assets/img/ep.png)

Dehghani's critical evaluation of the traditional ETL (Extract, Transform, Load) process struck a chord with me. She posits that the term 'extraction' implies a passive role for the provider and an intrusive role for the consumer, often leading to an unhealthy and fragile data ecosystem. This fragility stems from the fact that operational databases are typically not optimized for external usage beyond their intrinsic applications, resulting in pathological coupling.

In contrast, a paradigm shift towards publishing, serving, or sharing data heralds an era where data sharing moves from raw database access to intentional sharing of domain events or aggregates. With Data Contracts, we're manifesting this very transition. BI's traditional role of ETL/ELT is morphing, and teams are encouraged to approach their data in the same manner as they would their APIs - precise scoping for specific use cases, versioning, and thorough documentation.

This brings to mind [Hyrum's Law](https://www.hyrumslaw.com/), which essentially asserts that any observable behaviour of an API will eventually be relied upon by some users, irrespective of the promises in the contract. If the published data isn't well-considered, it may expose certain quirks that users latch onto, resulting in opaque and potentially damaging dependencies. Therefore, data owners have to ensure the data they publish is comprehensive, meticulously structured, and thoroughly documented to prevent the inadvertent creation of such dependencies, and it's our job to help them do this.
