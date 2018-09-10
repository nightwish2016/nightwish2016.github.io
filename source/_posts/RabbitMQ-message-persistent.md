title: RabbitMQ_message persistent-Durable queue(5)
author: Kevin Zhou
tags:
  - RabbitMQ
  - 'C#'
categories:
  - RabbitMQ
date: 2018-09-07 17:13:00
---
Background:
Queue message may miss once RabbitMQ restart or server crash. How can we handle this situation ?


Solution:
1.Declare a durable exchange 
2.Declare a durable queue
3.Set Persistent to true when we publish message

Send project code:
```Csharp
DurableQueueTest(args, "DurableQueue");
static void DurableQueueTest(string[] args, string quename)
        {
            var factory = new ConnectionFactory() { HostName = "localhost" };
            using (var connection = factory.CreateConnection())
            {
                using (var channel = connection.CreateModel())
                {
                    string exchangeName = "DurableQueueTest";
                    string routingKey = "DRK";
                    channel.ExchangeDeclare(exchangeName, ExchangeType.Direct, true, false, null);
                    channel.QueueDeclare(queue: quename, durable: true, exclusive: false, autoDelete: false, arguments: null);
                    channel.QueueBind(quename, exchangeName, routingKey, null);
                    var properties = channel.CreateBasicProperties();
                    properties.Persistent = true;
                    string message = args.Length > 0 ? exchangeName + " " + args[0] : "Hello RabbitMQ";
                    var body = Encoding.UTF8.GetBytes(message);
                    channel.BasicPublish(exchangeName, routingKey, basicProperties: properties, body: body);
                    Console.WriteLine("[x] Sent {0} ", message);

                }
            }
        }
```
<!--more-->

Step:
1.Run the send project code
 ```
 dotnet run
 ```
2.Check exchange and queue, you will find  both of them are druable.

{% asset_img DurableExchange.png DurableExchange %}
{% asset_img DurableQueue.png DurableQueue %}
3.Send a persistent message and non-persistent message by Rabbitmq management

{% asset_img Non-persistentMsg.png Non-persistentMsg %}
{% asset_img persistentMsg.png persistentMsg %}
{% asset_img QueueMsg.png QueueMsg %}

4.Stop rabbitmq service to mock rabbitmq crash
5.Restart rabbitmq service

{% asset_img StopSartRMQ.png StopStartRMQ %}
6.Check queue message from Rabbitmq management,You will find that persistent message still could be found in the queue,but another non-persistent message is missing

{% asset_img queue.png queue %}
{% asset_img msg.png msg %}