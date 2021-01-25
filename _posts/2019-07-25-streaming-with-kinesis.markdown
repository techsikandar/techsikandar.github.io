---
layout: post
title:  "Streaming with Kinesis & Time value of data"
date:   2019-07-25 21:13:27 -0500
categories: aws kinesis
---

<h1>{{ "Data Streaming" }}</h1>

Data streaming is nothing but a continuous flow, and in most cases never ending flow of data from various sources, in different formats, sizes & frequencies. Streaming data from financial institutions includes payments data, credit card transactions, AML data, stock prices, audit data based upon overall banking activities, etc. Streaming data from retail industry includes supply chain data, sales, shipment and live prices data. Data may even come from different mobile and IOT devices.

<h1>{{ "Time value of data" }}</h1>

This data, in majority cases, require an immediate action, otherwise it will be of very little use. Real opportunity will be missed because this data which is coming in a real time will gradually turn into historical data and lose its shine and you will end up doing batch processing instead of real-time processing.

Take fraud monitoring as an example. If someone is misusing your credit card then you expect your bank to take immediate action by alerting you and then blocking the transaction. Fraud detection involves monitoring the transaction in real time, processing the data, find out the anomalies, and then take action. Another example is ocean acoustics which is the key for underwater communication & navigation and travel-time analysis for Autonomous underwater vehicle (AUV). This also requires real-time or at-least near real-time processing and analysis. There are lots of such cases.

<h1>{{ "How do we design such a system?" }}</h1>

If you start thinking about designing such a system, then you must consider few design concerns, like:

<ul>
<li>Rate of producers producing messages and then rate of consumers consuming the messages</li>
<li>Rate of data transfer</li>
<li>Concurrency</li>
<li>Data sources & destinations</li>
<li>Data format, size & mapping</li>
</ul>

Its not scary but addressing these concerns require time & effort. We have been designing & implementing these systems from scratch in past, but over the period of time, business ecosystem & expectations are evolved. Plus, the data sources these days are heterogeneous. In fact, everything, everything, literally everything is a data source. And it’s not only textual data i am talking about here. It can be audio or video as well. And it’s not only processing i am talking about here, it’s about processing, filtering, analyzing and reacting to the data in real time.

Remember the time value of data.

If you have decided to go ahead with your own design & implementation then you have to take care of the infrastructure as well. There are some awesome solutions like Apache Kafka which will do everything for you but you still have to manage the infrastructure.

<h1>{{ "Enter Kinesis" }}</h1>

Amazon Kinesis enables you to ingest, buffer, and process streaming data in real-time, so you can derive insights in seconds or minutes instead of hours or days. Its Fully managed and scalable. You can perform rapid and continuous data intake and aggregation.