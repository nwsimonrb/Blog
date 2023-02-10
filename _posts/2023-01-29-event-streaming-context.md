---
layout: post
title: 'Event streaming in context'
tags: [Context, Event Streaming, Kafka]
featured_image_thumbnail:
featured_image: assets/images/posts/Event_Streaming/time_critical_data.jpg
featured: true
hidden: 
---
Until the late 2000s, most data storage and delivery solutions followed a fairly standardized format. SQL databases<sup>1</sup> were the preferred choice to store and manipulate data across various IT applications. Their tabular structure enables an intuitive understanding of data models and query formulation. For lower query frequencies and throughput requirements, they are still the go-to solution for data management today.<br />
However, since the 2010s new storage and delivery technology has emerged to overcome slow data pipelines. **Event streaming**<sup>2</sup> supports continuous processing of smaller, event-based data packets. From e-commerce recommendations to controlling logistics operations, it can be found in the majority of near-real-time processes at large enterprises worldwide. Household names are Apache Kafka, Spark, Flink and Storm. Each of them runs under the Apache Foundation's open source license and has a unique capability spectrum for certain use cases. In addition to these frameworks, there are an even larger number of solutions from proprietary vendors. For now, we will focus on [Apache Kafka](https://kafka.apache.org), since it has a high profile and is widely used (in over 80% of Fortune 100 companies).  

#### Why use event streaming?
The main advantage of providing data almost instantly is the ability to act proactively, rather than just reactively, as events unfold. Operational decisions - whether on the internal or on the customer side - are made in short time windows that require an assessment of the context that is as up-to-date as possible. Data arriving in batches that cover long timespans might be useful for tactical and strategic purposes, but lose their criticality fast (featured image above). Imagine a competitive market with a fast moving product catalog. Many potential customers are lost to a company that updates all the ads in its online store at long intervals, instead of displaying each single update as soon as it is ready. The advantage of high operating performance is often offset by two disadvantages. Let's look at the economically motivated first.

#### Does it pay for itself? 
Higher (data) throughput usually increases operating costs due to higher maintenance and energy consumption. This needs to be set against the value won by a higher degree of operational efficiency. Depending on the use case, data processing and storage solutions differ greatly in their technological composition and scaling. This makes it difficult to find scientific publications on the economic comparison between batch and stream delivery without specifying the context. 

I therefore resort to an exemplary calculation of my own. It is intended to provide an approach for an initial cost-benefit calculation of data delivery technologies. To avoid the difficulty of dealing with non-comparable deployment practices and product categories, the technical specifications focus on common infrastructure categories in the cloud. The major cloud providers have different specializations, but compete in the same price range when it comes to basic compute and storage provisioning. For simplicity, I assume that market prices include infrastructure management and energy consumption. The source for the upcoming numbers is the [pricing calculator](https://azure.microsoft.com/en-us/pricing/calculator/) from Microsoft Azure. Azure is chosen as the exemplary cloud provider as it is suitable for practical exercises of the associated [tutorial](https://simon.richebaecher.org/streaming-dashboard-tutorial). 

##### 1) Calculating scenarios: batch vs. stream delivery
Lets go back to the use case of a company listing fast changing product and price updates online. By comparing two scenarios, one with a batch approach and one with a streaming approach, we can gain insights. The examples are deliberately kept simple for demonstration purposes. We first look into the technical assumptions and their current pricing (West Europe region):
1. Batch delivery setup:
	- **Azure Database for PostgreSQL** is a general purpose SQL database with an open source licence. Answers to JDBC<sup>3</sup> queries can be given quickly, but their timeliness depends on the data integration flows. A Single server (Gen 5, 8vCore, $0.8336/hour) with 1TB of storage ($0.137/GB) costs $757.33/month.
	- A **virtual machine (VM)** acts as a staging area for new data on products/prices. CSV<sup>4</sup> files are retrieved at regular intervals from various application programming interfaces (API). Scheduled scripts are used to summarize the data and write it to the database. A small Linux VM (A3: 4 Cores, 7GB RAM, 285GB temporary storage, $0.240/hour) costs $175.20/month. 
2. Stream delivery setup:
	- **Apache Kafka** works as a near real-time data delivery system. New information about products/prices is continuously written into a Kafka topic by producers and consumed as needed (more explanation: [tutorial](https://simon.richebaecher.org/streaming-dashboard-tutorial)). Kafka, which runs in the Azure Service **HDInsight** in its most basic configuration (Instances: A6 for Head and Worker node, A5 for Zookeeper node, D4 for Edge node, Standard 1024GB Disk size), costs $1,662.21/month.  

{% include image-caption.html imageurl="/assets/images/posts/Event_Streaming/scenarios_batch_stream.jpg" title="Scenarios batch vs stream" %} 
<br />
At first glance, we see that the streaming setup is about 1.8 times more expensive at $1,662.21/month than the monthly $932.53 for a batch delivery setup. How can this significant cost increase be justified for a small online business? Let's assume realistic figures and calculate economic indicators. The context:
- On average, each sale of a product earns the business a profit of 5$. Bookmakers have already deducted the cost of procurement, wages, website, etc. from revenues, so the only cost to calculate is the cost of data delivery, which now makes a difference.
- The website is visited by around 500 potential customers per hour. In the current constellation of scenario 1), one out of 200 (0.5%) buys a product. The other customers leave the store because they can't find the latest product they are looking for or because the competition advertised faster.
<br />

With these propabilites we can calculate the expected values, i.e. the expected profit for both scenarios.
1. Scenario:
	- New data is always collected in the VM for an hour until the scripts start and the database is updated. On an hourly basis the setup costs around: $$ \frac{$932\ /month}{730\ hours/month} = $1.2\ /hour $$. 
	- The expected profit from each website visit (on average) is: $$ $5\ /purchase * 0.005\ purchases/visit = $0.025\ /visit $$<br />
	The expected hourly profit is therefore: $$ 500\ visits/hour * $0.025\ /visit = $12.5\ /hour$$
	- The remaining hourly profit is: $$ $12.5\ /hour - $1.2\ /hour = $11.3\ /hour $$
2. Scenario: 
	- The Kafka instance provides continous data updates to the store, which are on average 40 per hour. Each actualisation therefore costs around: $$ \frac{$1.600,00\ /month}{(40\ act./hour\ *\ 730\ hours/month)} = $0.056\ /act.$$ <br />
	But to make costs comparable to scenario 1) we calculate the hourly costs as: $$ \frac{$1.600,00\ /month}{730\ hours/month} = $2.27\ /hour$$
	- Frequent product and price updates have elevated customer acquisition to every 150th (around 0.67%). This means that the expected profit is now around: $$ $5\ /purchase * 0.0067\ purchase/visit = $0.033\ /visit$$ <br />
	The expected hourly profit therefore rises to: $$ 500\ visits/hour * $0.033\ /visit = $16.5\ /hour$$
	- The remaining hourly profit is: $$ $16.5\ /hour - $2.3\ /hour = $14.2\ /hour$$

Using these figures, we can see how an increase in the cost of data delivery is justified by an increase in overall profit. If we set $11.3/hour as the outcome for scenario 2), we can calculate the minimum increase in customer acquisition required to reach the break-even point for an event streaming transition. The result of the calculation postulates: If the probability of a purchase by event streaming changes to 1:184 (from 0.5% to 0.54%), the hourly profit is already the same as in scenario 1).

##### 2) Assessing streaming potential: volatility and urgency 
The potential impact of this technology on customer satisfaction becomes clearer when we look at the actualization costs of scenario 2). At first glance, $0.056/act. appears uneconomical compared to an expected profit of $0.033/visit. However, the expected profit is an average value for all product categories and points in time. We have already found that the customer base is quite volatile, as interest quickly wanes if expectations are not urgently met.If every single one of the 40 product/price updates per hour is a potential reason for customer acquisition, the picture is different. Now $0.056/act. is the cost we pay for a high chance of 1:n additional $5 profit. Imagine 25% of all potential customers are looking for an updated product/price that is included in the 40 updates provided in near real-time. 
The expected profit would rise to: $$ $5\ /purchase * 0.25\ purchase/visit = $1.25\ /visit$$ <br /> And the expected hourly profit would increase to: $$ 500\ visits/hour * $1.25\ /visit = $625\ /hour$$<br />  This shows the enourmous potential of event streaming technologies in use cases with high volatility and urgency. Such is not surprising for scenarios in which rapid adjustments to environmental influences or customer requirements are essential.

Please note that I do not discuss more extensive infrastructure requirements at this point. The transition to event streaming typically requires extensive deployment or modernization of data sources and appropriate network infrastructure. These long-term investments are strategic in nature and beyond the scope of this showcase.

#### Is it complicated? 
Now we arrive at the second potential drawback. The complexity ... is a technological hindrance for many interested in event streaming. The innovations behind it do not only require a rethinking on how data is processed and stored. They come with the need for trained personell to manage a more complicated, high-maintenance infrastructure. This makes adoption more difficult, especially for companies who do not have the means for these highly sought after streaming engineers. <br />
But as with many complicated solutions in IT, after some time there are software and service solutions which package technology in a more user friendly way. The HD Insight solution of section X is one of 4 popular ways of working with Kafka on Azure <sup>3</sup>. But as in-depth Kafka knowledge is needed, it still falls in the "more difficult" category. Ease of use comes with the fully managed Kafka service provided by the company Confluent. It offers a Software-as-a-Service (SaaS) solution that integrates natively into cloud environments of the three biggest providers Amazon, Microsoft and Google. Once the basic workings of Kafka are understood, Confluent Cloud can be a way to quickly and efficiently adopt event streaming for the own use case. An introduction with a hands-on tutorials on how to create your first streaming dashboard with Confluent and Power-BI can be found in the [new tutorial](https://simon.richebaecher.org/streaming-dashboard-tutorial).

difficult wihtout bias. many package good, kafka under the hood. 

{% include image-caption.html imageurl="/assets/images/posts/Event_Streaming/provider_venn.jpg" title="Streaming Providers" caption="Selection of streaming providers and their foundational frameworks"%} 

point at source: https://www.kai-waehner.de/blog/2022/12/21/data-streaming-landscape-2023/

use [Gartner Portal](https://www.gartner.com/reviews/market/event-stream-processing) with filters to find suitable reviews.

#### What about its environmental impact?
The ongoing trend of digitalization consumes increasing amounts of electricity, which is not yet generated renewably in its majority. Event-streaming technology with its high throughput requirements is no exception in this regard. But as with the economic calculations, one can weigh its environmental cost against its beneficial impacts. These considerations are, again, highly depend on the industry and use cases. Efficiency increases in high-emission industries can often only be corrected by carbon offsett programs. In contrast, companies with energetic or material sustainability concepts can immediately have a positive impact through advancements in operational efficiency. 

This does not necessarily mean that more output or growth is achieved by applying streaming technology. Savings potential can be just as valuable. Route optimization in logisitcs or scheduling of fabrication windows outside of peak loads in the power grid are two examples. A similar approach as with the exemplary calculation of section X could be used - but instead of profit per unit one would look at savings/cost reductions per unit. <br />
Finally, it needs to be said that - as with all technologies - that there is no one size fits all. SQL databases are still cost efficient storage solutions and batch delivieries make sense in time insensitive scenarios. Investments in IT infrastructure incorporate a lot of risk and produce technical debt. The carefull calculation of business cases is still a must.

#### Where does the story go?
If you follow trends in the market for data processing and storage solutions, you will be familiar with two companies that are currently driving change. Snowflake and Databricks are gaining market shares because they champion unique perspectives and ultimately have the same goal - to be "the one cloud platform" for all data related processes: integration, storage, analytics, machine learning (ML), delivery, etc. Snowflake is praised for its performant and easily scalable storage solutions. Databricks on the other side is favoured by customers focusing on data science and ML applications. A great read on the competitive development between both companies and their product can be found on [Kostas Pardalis blog](https://www.cpard.xyz/posts/the_battle_of_the_data_platform/). 

If both platforms are so versatile and incorporate streaming technologies, why did I go for Confluent Cloud in the hands-on tutorial? The reason is that currently no other company packages event streaming the way Confluent does. Founded by Apache Kafkas inventors, it focuses on further developing fully managed solutions to realtime data delivery. The recent aquisition of Immerok to add Apache Flink services to the mix underlines this objective. Snowflake and Databricks incorporate more and more features, but do not have a lead in this regard. For now all three companies are therefore seen as complementary services on the market. And they all know it, offering connectors between features of their products until they might be able to convince you of their own solution. 

These developments are crucial to the current architecural trend towards a highly networked "data mesh" approach. If such continues, future cloud applications will consist of a centralized delivery system and decentralized processing systems. Solutions like Apache Kafka will likely take the role of a "central nervous system" in this regard. I highly reccomend looking into the data mesh topic and will propably feature it in a future post (see announcement X). After this outlook, I hope you are motivated to learn the basics of event streaming in my [tutorial](https://simon.richebaecher.org/streaming-dashboard-tutorial). 
-> next page



<sup>1</sup> Structured Query Language - relational database language
<sup>2</sup> Also known as stream processing (see [Wikipedia](https://en.wikipedia.org/wiki/Stream_processing)) 
<sup>3</sup> Java Database Connectivity - database interface
<sup>4</sup> Comma-separated values - delimited text file format 

<sup>1</sup>
https://github.com/apache/kafka
<sup>2</sup>
https://www.confluent.io/press-release/survey-apache-kafka-users-highlights-impact-streaming-data-global-businesses/
<sup>3</sup>
https://speakerdeck.com/hpgrahsl/4-different-ways-of-working-with-kafka-on-azure-at-global-azure-2021
https://itnext.io/apache-kafka-in-azure-6985ccdce89f

https://azure.microsoft.com/en-us/pricing/calculator/