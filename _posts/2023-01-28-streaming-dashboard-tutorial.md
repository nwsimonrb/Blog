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
If you already are working with other cloud providers like AWS or Google Cloud, you can still follow the tutorial from step 2) after setting up Confluent on your platform. Both Confluent and Power BI are widely used in many industries and businesses. So there is a high chance that you will have an option to further work with them in the future. 

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
To envision our goal of building a real-time dashboard, let's follow the use-case scenario of the [previous post](https://simon.richebaecher.org/event-streaming-context). As owners of the online business, we continually get updates on product amounts and price changes from our warehousing and pricing teams. Besides updating our online shop with this data we want to give procurement a head start on purchasing items with low inventory or high turnover. For this purpose, our dashboard needs to show a list of product and price pairs that just changed in the last 20 minutes.<br /> 
With the following step-by-step instructions, you will build a small prototype. In addition to reading the task descriptions you can also play the video I made to show tasks 2), 4) and 5) visually. Follow [this link to YouTube](https://www.youtube.com/watch?v=qpa-7RvLqb8) or open the embedded version at the end of this section.

##### 1) Azure Account and Confluent Cloud setup
We start by creating a free tier Microsoft Azure account in one of two ways:
- **For Students**: If you have access to a university or school email start a plan without credit card details [here](https://azure.microsoft.com/en-us/free/students/). You will first need to log in with your (school/uni) Office365 credentials and then be required to identify with a valid phone number. The free credit is $100 and popular Azure services can be tested without credit usage.
- **Everyone Else**: Follow the same process with regular credentials [here](https://azure.microsoft.com/en-us/free/), with the addition of providing your credit card details. The free credit is $200 (for 1 month) and popular Azure services can be tested without credit usage (for 12 months or more).

After signing up we arrive at the Quickstart Center of the Azure portal. Navigate to the menu in the top left corner and choose 'Create a resource'. Now we can search the marketplace for **Apache Kafka® on Confluent Cloud™** and follow the setup guide [here](https://docs.confluent.io/cloud/current/billing/ccloud-azure-payg.html#get-started-with-ccloud-on-the-az-marketplace-with-pay-as-you-go). For the first trial month, $400 of credit is available for spending. This amount is more than sufficient for our upcoming scenario. Afterward, all 'pay-as-you-go' billings will be handled by your Azure account.

##### 2) Create Kafka infrastructure (Scenario)
The setup guide should have already prompted us to enter Confluent Cloud via automated login. In the [tutorial video](https://www.youtube.com/watch?v=qpa-7RvLqb8) we start the following tasks from the Azure Platform to ensure repeatability:
- Navigate to Confluent Cloud.
- Create an **environment** and subsequently a **cluster** on a basic plan.
- Create a **topic** for the messages we want to handle. Our scenario is focused on product data so we name it "products". 
- Go to the connector marketplace and select the **Datagen connector**. It will simulate incoming data from a producer. Choose the Product data template and schemaless JSON format so that we get suitable and simple messages delivered to our topic. Create the Datagen connector.  
- Navigate back into our topic and select an offset like 0 to view incoming messages. You can click on a single message to view its contents. On the front page of the topic, we can see the current write and read throughput. For now, we see no read activity, let's change that.   

##### 3) Power BI Service setup
Before we can set up an endpoint to receive data from Confluent Cloud - which is not a functionality of Power BI Desktop - we need to create a Power BI Service account. You find the free tier offering [here](https://powerbi.microsoft.com/en-us/getting-started-with-power-bi/). It lasts for 60 days and includes the basic features which are more than sufficient.<br />
After selecting "try Power BI for free" we are prompted to enter an email address. If you have an existing student or workplace account for Microsoft/Office365 you can use it and that will usually require no additional setup. The other option is to sign up with a private email that is not linked to a standard provider like Outlook, Gmail, or iCloud. This way you can get past the barrier for personal addresses. Emails linked to private domains (ex. john@johnssurename.com) can be created for a monthly fee at providers like Google (Google Workspace-Domains [here](https://domains.google/get-started/email/)). 

##### 4) Power BI API and dashboard (Scenario) 
We now prepare Power BI as the receiver before sending data from Confluent Cloud in step 5). Once you are within Power BI Service navigate to your workspace. It is found on the bottom of the left-side navigation bar or below the recommendations of the "Home" page. From within the workspace, do the following:
- Create a new **Streaming dataset**. Select the basic API source and name your dataset (ex. "productUpdates"). The values from stream fields refer to the JSON keys we find in each message soon delivered by Kafka. Pay attention to correct spelling when entering, you can check the example JSON created in the window below. Also, edit the data type from Text to number for price and id. In the end, check the "Historic data analysis" slider as we want to keep data from at least the last 20min.
- After creating the streaming dataset we see a **Push URL** which we copy and store securely for usage in step 5). The URL contains a token that should only be known to the sender (Confluent) and receiver (Power BI) - it is not for sharing. Although the connection will be securely encrypted through HTTPS, please keep in mind that data is sent over the public web and such entails residual risk (more information [here](https://medium.com/smallcase-engineering/web-security-access-token-in-url-79366a2bcb49)).       
- Create a new **Dashboard** and add a tile with your custom streaming dataset. For starters choose a clustered bar chart as the visualization type. Enter product as the axis and price as values. Finish by choosing 20 minutes as the time window to display. You can optionally add additional visualizations as shown in the video.  

##### 5) Confluent to Power BI connection
- Last: connection via http connector and look at final result

Review:
Overall simple convenient solution, keep in mind showcase, real life use case requires more thoughts on architecture aspects like secure networking. 

More producers possible: connectors, just have to be careful: schema. And maybe aggregate/process data before in dashboard: ksql processing. Next posts. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/qpa-7RvLqb8" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

secure api tokens: 


**The quick brown [fox][1], jumped over the lazy [dog][2].**

[1]: https://en.wikipedia.org/wiki/Fox "Wikipedia: Fox"
[2]: https://en.wikipedia.org/wiki/Dog "Wikipedia: Dog"
