---
layout: post
title:  "How to build your first event streaming dashboard"
tags: [ Tutorial ]
featured_image_thumbnail:
featured_image: assets/images/posts/2018/12.jpg
# featured: true
# hidden: true
---

The following tutorial is intended to give you a lightweight and practical introduction to event streaming. Look forward to quick visual results when building a dashboard with real-time data. If you do not need the short explanation of Kafka's components in Part 1, you can jump straight into the step-by-step guide of Part 2.  

#### Our toolset
My post [Event Streaming in Context](https://simon.richebaecher.org/event-streaming-context) looked at the pros and cons of choosing Apache Kafka for event streaming solution. By using Confluent in this tutorial we will have a beginner-friendly Kafka experience.<br /> 
Choosing Microsoft Azure as the cloud provider where we host our confluent service has two advantages. First, we can use your existing Office365 account to easily create an azure account with free trial benefits. Second, no additional confluent account setup is required as Azure manages identity and payment details in the background. Of course, confluent also offers free trial bonuses. As does our dashboarding tool Power BI. More concretely we will use the cloud-native Power BI Service. This means that all you need is your browser and no software installation will be required.<br />
All these tools are widely used in many industries and businesses. So there is a high chance that you will have an option to further work with them in the future. 

#### Part 1: Kafka Basics
Our main goal with event streaming is to deliver messages from one side to the other at the exact moment events unfold. In between the producing side and the consuming side lies a delivery and storage system like Apache Kafka. Kafkas most relevant components can be seen in figure X. At the center, we see a **cluster** that houses other components as an infrastructure backbone. Basically, it is a compute and storage group that can be easily scaled to adapt Kafka to volatile throughput. From the inside out, we find the following within a cluster:<br />
- **Messages**: Key, value pairs that describe an event like a notification or a state. A value contains information in a structured form (serialized object, for ex. in JSON). Keys do not have to be unique identifiers. However, they pre-define in which order messages are stored and later processed. For example, updates on the price of a product should have a unique product id as a key to make sure that they are consumed in order. 
- **Topics**: Thematic structures to group events - like one topic for product and one for customer updates. A topic stores events in immutable logs that are not accessible by index but by offset. The retention period for data within these logs can be set as wished.
- **Partitions**: Break topics into smaller entities to increase processing performance through distribution among multiple compute instances. If messages have the same key they will always land in the same partition. Otherwise, messages are written into partitions via round robin method. 
- **Brokers**: A network of independent machines (can be computers, containers, servers, ...). They manage partitions as well as read and write requests. Partitions are replicated and distributed among brokers to ensure security and redundancy. 

Data delivery to and from the cluster has two roles: 
- **Producer**: Utilizes a Kafka-specific application programming interface (API) to write messages to one or more topics. The java-based API has many adjustable settings for the write service itself (client, network buffering, connection pooling,...). The written messages are also referred to as a Producer record.
- **Consumer**: Reads messages by utilizing another type of Kafka API. The connection follows a subscription principle to pull messages from one or more topics. Such messages are called Consumer records.

Connected systems can be producers and consumers at the same time. Keep in mind that a whole network of systems can use Kafka to exchange data back and forth. Many connected services also do not work with event-based data the way Kafka does or require a different communication protocol. For such cases, so-called **Connectors** are used to wrap translation functionalities around the APIs. With a big and ever-growing ecosystem of (cloud) technologies and services, platform providers like Confluent offer marketplaces for pluggable connectors. This means that there is no hindrance when wanting to exchange data with systems like SQL databases.

Before the practical part starts, I want to point you at some additional learning material. [Confluent](https://developer.confluent.io/learn-kafka/apache-kafka/events/) has an array of great videos that go from beginner-friendly to advanced content. They alternate between technical explanation and hands-on practice. I purposely presented you with the minimum Kafka vocabulary needed for Part 2. The high-quality courses by Confluent provide more than enough material to go more in-depth.  

#### Part 2: Realtime Dashboard with Cloud Services
In addition to the step-by-step instructions, you can also play the video I made to see certain tasks in detail. Follow [this link to YouTube](https://www.youtube.com/watch?v=qpa-7RvLqb8) or open the embedded version at the end of this section. 

##### 1) Azure Account and Confluent Cloud setup

##### 2) Create Kafka infrastructure (Scenario)

##### 3) Power BI Service setup

##### 4) Power BI API and dashboard (Scenario) 

##### 5) Confluent to Power BI connection
 

- Look into Confluent and create cluster, env., topic
- Scenario of online shop: datagen prices and product
- For now visualize all incoming changes in dashboard: Power BI, setup account
- Power BI setup API and dashboard with tiles 
- Last: connection via http connector and look at final result

Review:
Overall simple convenient solution, keep in mind showcase, real life use case requires more thoughts on architecture aspects like secure networking. 

More producers possible: connectors, just have to be careful: schema. And maybe aggregate/process data before in dashboard: ksql processing. Next posts. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/qpa-7RvLqb8" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>


**The quick brown [fox][1], jumped over the lazy [dog][2].**

[1]: https://en.wikipedia.org/wiki/Fox "Wikipedia: Fox"
[2]: https://en.wikipedia.org/wiki/Dog "Wikipedia: Dog"
