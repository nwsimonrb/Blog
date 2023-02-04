---
layout: post
title:  "How to build your first event-streaming dashboard"
tags: [ Tutorial ]
featured_image_thumbnail:
featured_image: assets/images/posts/Event_Streaming/streaming_dash.png
featured: true
hidden: true
---
15min read, 30 min total

The following tutorial is intended to give you a lightweight and practical introduction to event-streaming. Look forward to quick visual results when building a dashboard with real-time data. If you do not need the short explanation of Kafka's components in Part 1, you can jump straight into the step-by-step guide of Part 2.  

#### Our toolset
My post [Event Streaming in Context](https://simon.richebaecher.org/event-streaming-context) looked at the pros and cons of choosing Apache Kafka for event-streaming solutions. We can avoid the issue of technical complicacy by using **Confluent Cloud** in this tutorial. This way we can have a beginner-friendly Kafka experience.<br /> 
Choosing **Microsoft Azure** as the cloud provider where we host our Confluent service has two advantages. First, we can use your existing Office365 account to easily create an azure account with free trial benefits. Second, no additional Confluent account setup is required as Azure manages identity and payment details in the background. Of course, Confluent also offers free trial bonuses. As does our dashboarding tool Power BI. More concretely we will use the cloud-native **Power BI Service**. This means that all you need is your browser - no software installation is required.<br />
If you already are working with other cloud providers like AWS or Google Cloud, you can still follow the tutorial of part 1 from step 2) onwards. Confluent provides documentation on how to set up its service on your platform as well. Both Confluent Cloud and Power BI are widely used in many industries and businesses. So there is a high chance that you will have an option to further work with them in the future. 

#### Part 1: Kafka Basics
Our main goal with event-streaming is to deliver messages from one side to the other at the moment events unfold. In between the producing and consuming side lies a delivery and storage system like Apache Kafka. Kafka's most relevant components can be seen in figure X. 
{% include image-caption.html imageurl="/assets/images/posts/Event_Streaming/kafka_basics.jpg" title="Apache Kafka Components" caption="Apache Kafka Components" %}

At the center, we see a **cluster** that houses other components as an infrastructure backbone. It is a compute and storage group that can be easily scaled to adapt Kafka to volatile throughput. From the inside out, we find the following within a cluster:<br />
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
To envision our goal of building a real-time dashboard, let's follow the use-case scenario of the [previous post](https://simon.richebaecher.org/event-streaming-context). As owners of the online business, we continually get updates on price changes from our pricing backend. Besides updating our online shop with this data we want to give procurement a head start on restocking items which will soon spark interest in our customers. For this purpose, our dashboard needs to show the product-price pairs that just changed in the last 20 minutes or less.<br /> 
With the following step-by-step instructions, you will build a small prototype. In addition to reading the task descriptions you can also play the video I made to show tasks 2), 4) and 5) visually. Follow [this link to YouTube](https://youtu.be/16gxdVTbB04) or open the embedded version at the end of step 2).

##### 1) Azure Account and Confluent Cloud setup
We start by creating a free tier Microsoft Azure account in one of two ways:
- **For Students**: If you have access to a university or school email start a plan without credit card details [here](https://azure.microsoft.com/en-us/free/students/). You will first need to log in with your (school/uni) Office365 credentials and then be required to identify with a valid phone number. The free credit is $100 and popular Azure services can be tested without credit usage.
- **Everyone Else**: Follow the same process with regular credentials [here](https://azure.microsoft.com/en-us/free/), with the addition of providing your credit card details. The free credit is $200 (for 1 month) and popular Azure services can be tested without credit usage (for 12 months or more).

After signing up we arrive at the Quickstart Center of the Azure portal. Navigate to the menu in the top left corner and choose 'Create a resource'. Now we can search the marketplace for **Apache Kafka® on Confluent Cloud™** and follow the setup guide [here](https://docs.confluent.io/cloud/current/billing/ccloud-azure-payg.html#get-started-with-ccloud-on-the-az-marketplace-with-pay-as-you-go). For the first trial month, $400 of credit is available for spending. This amount is more than sufficient for our upcoming scenario. Afterward, all 'pay-as-you-go' billings will be handled by your Azure account.

##### 2) Kafka infrastructure (Scenario)
The setup guide should have already prompted us to enter Confluent Cloud via automated login. In the [tutorial video](https://youtu.be/16gxdVTbB04) we start the following tasks from the Azure Platform to ensure repeatability:
- Navigate to Confluent Cloud. Create a **cluster** on a basic plan.
- Create a **topic** for the messages we want to handle. Our scenario is focused on product data so you could name it "products". Skip the creation of a schema for now - I plan to include the Confluent schema registry in an upcoming post (see [here]). 
- Go to the connector marketplace and select the **Datagen connector**. It will write dummy data into our "products" topic to simulate a producer. Generate a global access API key that will allow the connector to access your topic. Although we just do exemplary prototyping, it is always best to store the (downloaded) key securely. Choose the "Product" data template and JSON format so that we get suitable and simple messages delivered to our topic. In the advanced options, increase the production interval to 10.000 ms to simulate an incoming event every 10 seconds. Create the Datagen connector and wait a moment until it is provisioned.  
- Navigate back into our topic and view incoming messages in the respective tab. Note that we just receive increasing numbers as product names, prices, descriptions, etc. An upcoming post will introduce more diverse data sources (see [here]).<!--  In the overview tab, we can see the current write and read throughput. Our production rate has gone up as expected, while our small consumption rate can be explained by your short lookup in the messages tab.--> Now pause the connector until we reactivate it once the consumer side is ready. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/16gxdVTbB04" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

##### 3) Power BI Service setup
Before we can set up an endpoint to receive data from Confluent Cloud - which is not a functionality of Power BI Desktop - we need to create a Power BI Service account. You find the free tier offering [here](https://powerbi.microsoft.com/en-us/getting-started-with-power-bi/). It lasts for 60 days and includes the basic features which are more than sufficient.<br />
After selecting "try Power BI for free" we are prompted to enter an email address. If you have an existing student or workplace account for Microsoft/Office365 you can use it and that will usually require no additional setup. The other option is to sign up with a private email that is not linked to a standard provider like Outlook, Gmail, or iCloud. This way you can get past the barrier for personal addresses. Emails linked to private domains (ex. john@johnssurename.com) can be created for a monthly fee at providers like Google (see [Workspace-Domains](https://domains.google/get-started/email/)). 

##### 4) Power BI API and dashboard (Scenario) 
We now prepare Power BI as the receiver before sending data from Confluent Cloud in step 5). Once you are within Power BI Service navigate to your workspace. It is found on the bottom of the left-side navigation bar or below the recommendations of the "Home" page. From within the workspace, do the following:
- Create a new **Streaming dataset**. Select the basic API source and name your dataset (ex. "productUpdates"). The values from stream fields refer to the JSON keys we find in each message soon delivered by Kafka. Pay attention to correct spelling when entering, you can check the example JSON created in the window below. Also, change the data type from text to number for "price" and "id". In the end, check the "Historic data analysis" slider as we want to keep data from at least the last 20min.
- After creating the streaming dataset we see a **Push URL** which we copy and store securely for usage in step 5). The URL contains a token that should only be known to the sender (Confluent) and receiver (Power BI) - it is not for sharing. Although the connection will be securely encrypted through HTTPS, please keep in mind that data is sent over the public web and such entails residual risk (more information [here](https://medium.com/smallcase-engineering/web-security-access-token-in-url-79366a2bcb49)).       
- Create a new **Dashboard** and add a tile connected to your streaming dataset. For starters choose a clustered column chart as the visualization type. Enter "name" as the axis and "price" as values. Finish by choosing 20 minutes as the time window to display. You can optionally add more visualizations as shown in the video.  

##### 5) Confluent to Power BI connection
Data is now produced to our topic in Confluent, but we are still missing a consumer who can pull that data and push it to the Power BI API. Confluent has a fully managed connector for such purposes:  
- Navigate back to the connector marketplace and select the **HTTP Sink connector**. We follow the known procedure for the API key. Enter your URL from the Power BI, "TLSv1.2" as SSL Protocol (see why issue 1 below ) and continue. Choose JSON as the record value format and open the advanced configurations. Here, select "json" as the request body format and "true" for the "Batch json as array" field. Create the Connector and wait for the provisioning.
- Now you can reactivate the Datagen connector to produce new messages again. We had to pause it before so that no large backlog of unconsumed messages is built up. The HTTP connector would otherwise try to catch up with an increased number of API requests and cross the limitations of the Power BI API (see [restrictions](https://learn.microsoft.com/en-us/power-bi/developer/embedded/push-datasets-limitations)). Error codes like 429 (too many requests) are found within the messages of an error-connector_name topic. Such topics are automatically created once sink connectors are provisioned and are meant to help you troubleshoot.
- Let's go back to the Power BI dashboard and look at the influx of incoming dummy values. Dashboards provide rapidly adapting visuals for streaming purposes, but miss the comprehensive list of features a classic Power BI report has to offer. The video tutorial shows how streaming datasets can also be used in a report - with the drawback of manual refreshments.

Congratulations on building your first real-time Dashboard in the Cloud. Our end result is a simple prototype with dummy data but is highly customizable and scalable thanks to the capabilities of Confluent/Kafka. In the upcoming posts, we will build on this to explore more useful data integration and processing features. Stay tuned for updates with my [newsletter]() or over LinkedIn. 

Newsletter?


[1]: https://forum.confluent.io/t/confluent-http-sink-azure-event-hubs/2371 "explanation on TLS issue with this link"
[2]:  "Cause for 429 error"

too many : https://community.powerbi.com/t5/Service/API-Request-Blocked-by-Keyblocker/m-p/2888065


**jumped over the lazy [dog][2].**

[2]: https://en.wikipedia.org/wiki/Dog "Wikipedia: Dog"
