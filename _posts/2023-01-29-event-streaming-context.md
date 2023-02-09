---
layout: post
title: 'Event Streaming in Context'
tags: [Event Streaming, Monitoring, Practical Tips]
featured_image_thumbnail:
featured_image: assets/images/posts/Event_Streaming/time_critical_data.jpg
featured: true
hidden: true
---
Up until the late 2000s data storage was seen as a stationary and rather undynamic process. SQL databases were the preferred choice to store and manipulate data across various IT applications. Their tabular structure enable an intuitive understanding of data models and query formulation. For lower frequencies in data delivery they still are the go-to solution nowadays (*hint 1*). <br />
However, a new storage and processing technology tackling slow-paced data pipelines has found widespread use since the 2010s. **Event streaming** supports the continuous delivery of smaller, event-based data packages. From e-commerce recommendations to steering logistic operations, it can be found in the majority of near real-time processes of major companies worldwide. Household names are Apache Kafka<sup>1</sup>, Storm, Flink and Kinesis. Each one of them runs under the open-source license of the Apache foundation, with Kafka being the most widely used at the moment <sup>2</sup>.

Other proprietary additions?
Abstract und Absprung?

#### Why streaming?
The main benefit of provisioning data almost instantly is the ability to be proactive and not just reactive once events unfold. Operational decisions - be it from an internal or customer side - are made in short time-windows that require the most up-to-date assessment of a context. Data arriving in low-frequent batches which cover longer timespans might still be usefull for tactical and strategic purposes, but loose their criticality fast (featured image above). Imagine a competitive market with a fast moving product catalog. Many potential customers are lost to a company that updates advertisements in low-frequent batches at its online store, instead of displaying an update as soon as it becomes known.

#### Does it pay for itself? 
Two drawbacks can be brought forward against event-streaming. Let's look at the economically motivated first. Increased (data) throughput usually increases operating costs due to increased maintanance and energy consumption. This needs to be set against the value won by a higher degree of operational efficiency. 
Depending on the use case, data processing and storage solutions vary widely in technological composition and scaling. It is therefore hard to find scientific publications on the economic comparison between batch and stream delivery without specifying the context. 

I therefore resort to an exemplary calculation of my own. Its purpose is to provide an approach on how to make estimates for an initial cost-benefit calculation regarding data delivery technologies. To avert the difficulty of looking into incomparable setup practices and product categories, technical specs focus on common infrastructure categories in the cloud. The major cloud providers have different specialties but compete in the same price range when it comes to basic compute and storage provisioning. For simplification, I presume that marketprices also encorporate infrastructure management and energy consumption. The source of upcoming numbers is the [pricing calculator](https://azure.microsoft.com/en-us/pricing/calculator/) from Microsoft Azure. Azure has been chosen as the exemplary cloud provider due to its suitability for the hands on tutorials of chapter X. 

##### 1) Calculating Scenarios: Batch vs. Stream Delivery
Lets go back to the scenario of a company listing fast changing product portfolios with prices online. We first look into the technical assumptions and their current pricing (EU West):
1. Batch delivery setup:
	- **Azure Database for PostgreSQL** as a general purpose SQL database with an open source licence. Answers to JDBC queries can be quickly provided, but their actuality relies on the data integration workflows. A Single server (Gen 5, 8vCore, $0.8336/hour) and 1TB storage ($0.137/GB) -> $757.33/month
	- A **Virtual Machine (VM)** acts as a staging area for new data on products/prices. CSV files are pulled from a provided API or SFTP server in regular intervals. Timed scripts are used to summarize and write the data to the database. Additional database management tasks can also be run from this machine. A small Linux VM (A3: 4 Cores, 7GB RAM, 285GB temporary storage, $0.240/hour) costs $175.20/month. 
2. Stream delivery setup:
	- **Apache Kafka** works as a near real-time data delivery system. New information on products/prices is continuously written by producers to a Kafka topic and consumed as needed (more explanation: [tutorial](https://simon.richebaecher.org/streaming-dashboard-tutorial)). Kafka running in the Azure Service **HDInsight** on its most basic configuration (Instances: A6 for Head and Worker node, A5 for Zookeeper node, D4 for Edge node, Standard 1024GB Disk size) costs $1,662.21/month.  

{% include image-caption.html imageurl="/assets/images/posts/Event_Streaming/scenarios_batch_stream.jpg" title="Scenarios Batch vs Stream" %} 
<br />
At the first glance, we see that the streaming setup with $1,662.21/month is around 1.8 times more expensive than the monthly $932,53 for a batch delivery setup. How might this large increase in cost be justified for a small online business? Lets assume some realistic numbers and calculate economic indicators. 
The context:
- On average, each sale of a product earns the business a profit of 5$. The bookies have already subtracted costs for procurement, wages, website, etc. from the revenue, so that only the data delivery costs are left to be calculated and make a difference now.
- The website is visited by around 500 potential customers per hour. With the current setup of scenario 1), one in every 200 (0.5% of such) is buying a product. The other customers leave because they can not find the most up-to-date product they are looking for, or because competitors have reduced prices more quickly.
The expected value:
Scenario 1):
	- In the VM new data is always collected for an hour until the scripts start and the database is updated. On an hourly basis the setup costs around 932$/month / 730 hours/month = 1.2 $/hour. 
	- The expected profit from each website visit (on average) is 5 $/purchase x 0.005 purchase/visit=0.025 $ /visit. The hourly profit is therefore 500 x 0.025=12.5$ .
	- Overall an hourly profit of 12.5 $ -1.2 $ = 11.3$ remains.
Scenario 2) 
	- The Kafka instance provides continous data updates to the store, which are on average 40 per hour. Each actualisation therefore costs around 1.600,00 $/month / (40 x 730) = 0.056 $/ac. But to make costs comparable to scenario 1 we calculate the hourly costs as  1.600,00 $/month / 730 = 2.27 $/hour.
	- Frequent product and price updates have increased customer acquisition to one in every 150th (around 0.67%). This means expected profit lies now at around 5 $/purchase x 0.067 purchase/visit=0.033 $ /visit. The houly profit therefore rises to 500 x 0.033 = 16.5 $/hour.
	- Overall an hourly profit of 16.5 $ - 2.3$ = 14.2 $ remains.

With these numbers we can see how an increase in data delivery costs is justified by an increase in the overall profit. By setting 11.3$/hour as a result for scenario 2) we can calculate the minimum increase in customer acquisition needed to arrive at the break-even point for an event-streaming transition. If the event-streaming changes the propability of a purchase to 1:184 (from 0.5 to 0.54%) we already arrive at the same hourly profit as in scenario 1). 

##### 2) Assessing streaming potential: Urgency and Volatility 
The potential impact of this technology on customer satisfaction becomes clearer once we look at the actualization cost of scenario 2). At first glance 0.056 $/ac. seems uneconomical compared to an expected profit of 0.033 $/visit. However, the expected profit is an average over all product categories and we already established that the market is quite volatile, as customers quickly leave once their expectations are not met. If every one of the 40 product/pricing updates is a potential reason for customer acquisition we get a different view. Now 0.056 $/ac. becomes the cost we pay for a high chance to earn 1:n additional 5 $ profit. Just imagine if 25% of all potential customers look for an updated product/price, which will always be part of the 40 realtime provided updates. The expected profit would rise to 5$ /purchase x 0.25 purchase/visit = 1.25$ / visit and the hourly profit to 500 x 1.25$ /visit  = 625 $ / hour (5.000% increase to scenario 1)). This shows the enourmous potential of event-streaming technologies in use cases with high volatility.

every step of the way: volatilite, be prepared/adapt for short decision timeframe.

Please keep in mind that I do not go into detail concerning greater infrastructure requirements at this point. Transitioning to event-streaming usually demands the widespread setup or modernization of data sources and a suitable network infrastructure. These long-term investments are of strategic nature and out of the scope of this showcase.

#### Is it complicated? 
more stuff like event hub ... -> pic
difficult wihtout bias. many package good, kafka under the hood. 
Now we arrive at the second potential drawback. The complexity ... is a technological hindrance for many interested in event-streaming. The innovations behind it do not only require a rethinking on how data is processed and stored. They come with the need for trained personell to manage a more complicated, high-maintenance infrastructure. This makes adoption more difficult, especially for companies who do not have the means for these highly sought after streaming engineers. <br />
But as with many complicated solutions in IT, after some time there are software and service solutions which package technology in a more user friendly way. The HD Insight solution of section X is one of 4 popular ways of working with Kafka on Azure <sup>3</sup>. But as in-depth Kafka knowledge is needed, it still falls in the "more difficult" category. Ease of use comes with the fully managed Kafka service provided by the company Confluent. It offers a Software-as-a-Service (SaaS) solution that integrates natively into cloud environments of the three biggest providers Amazon, Microsoft and Google. Once the basic workings of Kafka are understood, Confluent Cloud can be a way to quickly and efficiently adopt event-streaming for the own use case. An introduction with a hands-on tutorials on how to create your first streaming dashboard with Confluent and Power-BI can be found in the [new tutorial](https://simon.richebaecher.org/streaming-dashboard-tutorial).

{% include image-caption.html imageurl="/assets/images/posts/Event_Streaming/provider_venn.jpg" title="Streaming Providers" caption="Selection of streaming providers and their foundational frameworks"%} 

point at source: https://www.kai-waehner.de/blog/2022/12/21/data-streaming-landscape-2023/

use [Gartner Portal](https://www.gartner.com/reviews/market/event-stream-processing) with filters to find suitable reviews.

#### What about its environmental impact?
The ongoing trend of digitalization consumes increasing amounts of electricity, which is not yet generated renewably in its majority. Event-streaming technology with its high throughput requirements is no exception in this regard. But as with the economic calculations, one can weigh its environmental cost against its beneficial impacts. These considerations are, again, highly depend on the industry and use cases. Efficiency increases in high-emission industries can often only be corrected by carbon offsett programs. In contrast, companies with energetic or material sustainability concepts can immediately have a positive impact through advancements in operational efficiency. 

This does not necessarily mean that more output or growth is achieved by applying streaming technology. Savings potential can be just as valuable. Route optimization in logisitcs or scheduling of fabrication windows outside of peak loads in the power grid are two examples. A similar approach as with the exemplary calculation of section X could be used - but instead of profit per unit one would look at savings/cost reductions per unit. <br />
Finally, it needs to be said that - as with all technologies - that there is no one size fits all. SQL databases are still cost efficient storage solutions and batch delivieries make sense in time insensitive scenarios. Investments in IT infrastructure incorporate a lot of risk and produce technical debt. The carefull calculation of business cases is still a must.

#### Where does the story go?
If you follow trends in the market for data processing and storage solutions, you will be familiar with two companies that are currently driving change. Snowflake and Databricks are gaining market shares because they champion unique perspectives and ultimately have the same goal - to be "the one cloud platform" for all data related processes: integration, storage, analytics, machine learning (ML), delivery, etc. Snowflake is praised for its performant and easily scalable storage solutions. Databricks on the other side is favoured by customers focusing on data science and ML applications. A great read on the competitive development between both companies and their product can be found on [Kostas Pardalis blog](https://www.cpard.xyz/posts/the_battle_of_the_data_platform/). 

If both platforms are so versatile and incorporate streaming technologies, why did I go for Confluent Cloud in the hands-on tutorial? The reason is that currently no other company packages event-streaming the way Confluent does. Founded by Apache Kafkas inventors, it focuses on further developing fully managed solutions to realtime data delivery. The recent aquisition of Immerok to add Apache Flink services to the mix underlines this objective. Snowflake and Databricks incorporate more and more features, but do not have a lead in this regard. For now all three companies are therefore seen as complementary services on the market. And they all know it, offering connectors between features of their products until they might be able to convince you of their own solution. 

These developments are crucial to the current architecural trend towards a highly networked "data mesh" approach. If such continues, future cloud applications will consist of a centralized delivery system and decentralized processing systems. Solutions like Apache Kafka will likely take the role of a "central nervous system" in this regard. I highly reccomend looking into the data mesh topic and will propably feature it in a future post (see announcement X). After this outlook, I hope you are motivated to learn the basics of event-streaming in my [tutorial](https://simon.richebaecher.org/streaming-dashboard-tutorial). 
-> next page


<sup>1</sup>
https://github.com/apache/kafka
<sup>2</sup>
https://www.confluent.io/press-release/survey-apache-kafka-users-highlights-impact-streaming-data-global-businesses/
<sup>3</sup>
https://speakerdeck.com/hpgrahsl/4-different-ways-of-working-with-kafka-on-azure-at-global-azure-2021
https://itnext.io/apache-kafka-in-azure-6985ccdce89f

https://azure.microsoft.com/en-us/pricing/calculator/