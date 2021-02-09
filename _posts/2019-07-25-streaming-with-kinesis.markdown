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

<h1>{{ "Data Collection with Kinesis" }}</h1>

There are three ways to write producer applications:

<ul>
<li>AWS SDK (PutRecord / PutRecords)</li>
<li>Kinesis Producers Library (KPL)</li>
<li>Kinesis Agent</li>
</ul>

Let's first create a data stream before going any further:

{% highlight ruby %}
# aws kinesis create-stream --stream-name TestStream --shard-count 1
{% endhighlight %}

Create stream is an asynchronous operation. This command will initially return the stream status CREATING which will be changed to ACTIVE when the create stream operation is completed. Sharding is a way to partition the data. It uniquely identifies the data flowing to Kinesis and it's base throughput unit for Kinesis data streams. A stream can have many shards, and there are some guidelines on how many shards we should have but for start let's just create one.

{% highlight ruby %}
# aws kinesis describe-stream --stream-name TestStream
# aws kinesis list-streams
{% endhighlight %}

<h1>{{ "AWS SDK (PutRecord / PutRecords)" }}</h1>

This CLI internally uses AWS SDK. Offcourse, you can directly use AWS SDK as well.

{% highlight ruby %}
# aws kinesis put-record --stream-name TestStream --partition-key 001 --data "Hello World 1"
# aws kinesis put-record --stream-name TestStream --partition-key 002 --data "Hello World 2"
{% endhighlight %}

The partition key is used by Kinesis data streams to distribute the data across multiple shards. The partition key is hashed and then divided by the number of shards. The modulo is used to determine which shard will be used. This way same shard will be used for same partition key. You can as well define the explicit shard by using ExplicitHashKey parameter. PutRecord returns the shard ID of where the data record was placed and the sequence number that was assigned to the data record. You can use the get-shard-iterator and then get-records to read the stream records.

{% highlight ruby %}
# aws kinesis get-shard-iterator --shard-id shardId-000000000000 --shard-iterator-type TRIM_HORIZON --stream-name TestStream
{
    "ShardIterator": "AAAAAAAAAAFT9/AkABBngD2Nus+P5JOZl70DGKSBYV58t8yf9DctHf51xfW13u1kbl5lojtKFFR8Lu224LsxUit7BWblJ1FvazrjebSJZMteNoY+gKds7MaORb5uJlw38a6TsJqVdVctVVMr8YOhOcopy8rP1eaSdzfXMY/d+J90sF4KMNZ5CvodbRTr5km4X0M+lFCii0gUgjzJU7EegJgz+v1O8Hpp"
}
{% endhighlight %}

{% highlight ruby %}
# aws kinesis get-records --shard-iterator AAAAAAAAAAFT9/AkABBngD2Nus+P5JOZl70DGKSBYV58t8yf9DctHf51xfW13u1kbl5lojtKFFR8Lu224LsxUit7BWblJ1FvazrjebSJZMteNoY+gKds7MaORb5uJlw38a6TsJqVdVctVVMr8YOhOcopy8rP1eaSdzfXMY/d+J90sF4KMNZ5CvodbRTr5km4X0M+lFCii0gUgjzJU7EegJgz+v1O8Hpp
{
    "Records": [
        {
            "SequenceNumber": "49608865602728722881888990266460632711628129014028173314",
            "ApproximateArrivalTimestamp": 1594701015.523,
            "Data": "SGVsbG8gV29ybGQgMQ==",
            "PartitionKey": "001"
        },
        {
            "SequenceNumber": "49608865602728722881888990266463050563267360127803457538",
            "ApproximateArrivalTimestamp": 1594701042.242,
            "Data": "SGVsbG8gV29ybGQgMg==",
            "PartitionKey": "002"
        }
    ],
    "NextShardIterator": "AAAAAAAAAAFfckXHJa/X7wBYNtRq5yXPm45zwRwuavedAw/DsIqbmiQSu+n5u2wgJmU3fBEtz7KSLtILqwx/MUsnZK0S3w6P29+ypG+qzLqE9uyxPU1uSfxF93KYUUYerYvMl27NiU0v4qNnvhZ6+jjTHp2vJ4NIIDQqeSaXlGslO2Df/kRGvc83/lOdyGcvWCCyuuGYR2t0s+CS87d7F33tuiDGZ5B2",
    "MillisBehindLatest": 0
}
{% endhighlight %}

Decode the data to view the actual data that was sent to Kinesis: 

{% highlight ruby %}
# echo "SGVsbG8gV29ybGQgMQ=="| base64 -d
# echo "SGVsbG8gV29ybGQgMg=="| base64 -d
{% endhighlight %}

Next shard iterator won't return any further record.

{% highlight ruby %}
# aws kinesis get-records --shard-iterator AAAAAAAAAAFfckXHJa/X7wBYNtRq5yXPm45zwRwuavedAw/DsIqbmiQSu+n5u2wgJmU3fBEtz7KSLtILqwx/MUsnZK0S3w6P29+ypG+qzLqE9uyxPU1uSfxF93KYUUYerYvMl27NiU0v4qNnvhZ6+jjTHp2vJ4NIIDQqeSaXlGslO2Df/kRGvc83/lOdyGcvWCCyuuGYR2t0s+CS87d7F33tuiDGZ5B2
{
    "Records": [],
    "NextShardIterator": "AAAAAAAAAAEqiT2iIC+HAW1WllfDvHac2b4KY3txxwA/74NnzfNXK86ageX6wGFTDR88ULbvE4Xgdh8VtJ+54N62fQ8TDQP+omVqiWpcH8wz5kfALNLsD7oETxq8uVoNVQ4Pbw/ocySFyOqLxQh+9ndykk1tc5Xy40LqeZP1zICHK61IKGxwhY3pL09BOWnUamRvoB/tetiZnjMCOm5bpKd77FNk7OBz",
    "MillisBehindLatest": 0
}
{% endhighlight %}

<h1>{{ "Kinesis Producers Library (KPL)" }}</h1>

KPL is an abstraction over AWS Kinesis SDK. It batches & aggregates the user records, has built-in retry mechanism, integrates well with Kinesis Client Library and supports both synchronous and asynchronous communication. Refer <a href="https://docs.aws.amazon.com/streams/latest/dev/kinesis-kpl-writing.html" target="_blank">this</a> article to see how to write to Kinesis data streams using KPL.

<h1>{{ "Kinesis Agent" }}</h1>

Let's install Kinesis Agent on the EC2 instance. It can be installed on-premise as well. Create an appropriate EC2 role ("EC2Kinesis") with appropriate access (Admin for testing). Launch an EC2 instance and attached this role.

{% highlight ruby %}
# sudo yum install –y aws-kinesis-agent
# sudo yum install –y https://s3.amazonaws.com/streaming-data-agent/aws-kinesis-agent-latest.amzn1.noarch.rpm
{% endhighlight %}

For the article, I copied 120 years of Olympics history from Kaggle and downloaded it under /var/log/olympics. Create a Kinesis data stream and make sure it's ready.

{% highlight ruby %}
# aws kinesis create-stream --stream-name OlympicStream --shard-count 1
{% endhighlight %}

Edit the file /etc/aws-kinesis/agent.json and make these changes.

{% highlight ruby %}
{
  "cloudwatch.emitMetrics": true,
  "kinesis.endpoint": "",
  "flows": [
    {
		"filePattern": "/var/log/olympics/*.csv",
		"kinesisStream": "OlympicStream",
		"initialPosition": "START_OF_FILE",
		"dataProcessingOptions": [
			{
				"optionName": "CSVTOJSON",
				"customFieldNames": ["ID","Name","Sex","Age","Height","Weight","Team","NOC","Games","Year","Season","City","Sport","Event","Medal"]
			}
		]
    }
  ]
}
{% endhighlight %}

Start the kinesis agent and tail the logs:

{% highlight ruby %}
# sudo service aws-kinesis-agent start
# tail -f /var/log/aws-kinesis-agent/aws-kinesis-agent.log
{% endhighlight %}

<h1>{{ "Conclusion" }}</h1>

I prefer using KPL because it improves throughput and provides retry mechanism. The fact that it batches the records before sending to Kinesis may introduce delay of up to RecordMaxBufferedTime. This can be configured. If you do not want this delay then you can use PutRecord(s) API. Kinesis Agent obviously can be used in cases where we have a continuous feed of data in a file and we want to send this data as it's arriving. You can optionally perform data transformation as well.