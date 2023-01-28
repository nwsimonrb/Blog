---
layout: post
title:  "How to build your first event streaming dashboard"
tags: [ Tutorial ]
featured_image_thumbnail:
featured_image: assets/images/posts/2018/12.jpg
# featured: true
# hidden: true
---

The following tutorial is intended to give you a lightweight and practical introduction to event streaming. Look forward to quick visual results when building a dashboard with real-time data.

##### Our toolset
My post [Event Streaming in Context](https://simon.richebaecher.org/event-streaming-context) looked at the pros and cons of choosing Apache Kafka as an event streaming solution. By using Confluent in this tutorial we will have a beginner-friendly Kafka experience.<br /> 
Choosing Microsoft Azure as the cloud provider where we host our confluent service has two advantages. First, we can use your existing Office365 account to easily create an azure account with free trial benefits. Second, no additional confluent account setup is required as Azure manages identity and payment details in the background. Of course, confluent also offers free trial bonuses. As does our dashboarding tool Power BI. More concretely we will use the cloud-native Power BI Service. This means that all you need is your browser and no software installation will be required.<br />
All these tools are widely used in many industries and businesses. So there is a high chance that you will have an option to further work with them in the future. 


##### Part 1: Kafka Basics
- message
- topics
- producer (with connector)
- consumer (with connector)
- partition
- brokers and replication

Part 2: Cloud SaaS
- Setup Azure Account and Confluent SaaS
- Look into Confluent and create cluster, env., topic
- Scenario of online shop: datagen prices and product
- For now visualize all incoming changes in dashboard: Power BI, setup account
- Power BI setup API and dashboard with tiles 
- Last: connection via http connector and look at final result

Review:
Overall simple convenient solution, keep in mind showcase, real life use case requires more thoughts on architecture aspects like secure networking. 

More producers possible: connectors, just have to be careful: schema. And maybe aggregate/process data before in dashboard: ksql processing. Next posts. 



**The quick brown [fox][1], jumped over the lazy [dog][2].**

[1]: https://en.wikipedia.org/wiki/Fox "Wikipedia: Fox"
[2]: https://en.wikipedia.org/wiki/Dog "Wikipedia: Dog"
