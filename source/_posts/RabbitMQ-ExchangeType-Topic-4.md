title: RabbitMQ_ExchangeType Topic(4)
author: Kevin Zhou
tags:
  - RabbitMQ
  - 'C#'
categories:
  - RabbitMQ
date: 2018-09-05 09:48:00
---
Exchange: Exchange is a rout map
ExchangeType: Topic, the message will be pushed to the specified queues which are matched to the Routingkey like  regular expression.
1.Send project code:
```Csharp
  TopicExchangeTest(args, "TopicQueue");
 static void TopicExchangeTest(string[] args, string quename)
        {
            var factory = new ConnectionFactory() { HostName = "localhost" };
            using (var connection = factory.CreateConnection())
            {
                using (var channel = connection.CreateModel())
                {
                    string exchangeName = "TopicTest";
                    string routingKey = "TopicRK.*";  //* for single word ,# for multiple words
                    channel.ExchangeDeclare(exchangeName, ExchangeType.Topic, false, false, null);
                    channel.QueueDeclare(queue: quename, durable: false, exclusive: false, autoDelete: false, arguments: null);
                    channel.QueueBind(quename, exchangeName, routingKey, null);
                    string message = args.Length > 0 ? exchangeName + " " + args[0] : "Hello RabbitMQ";
                    var body = Encoding.UTF8.GetBytes(message);
                    channel.BasicPublish(exchangeName, "TopicRK.one", basicProperties: null, body: body);
                    Console.WriteLine("[x] Sent {0} ", message);

                }
            }
        }
```
<!--more-->
2.Receive project code:
```Csharp
ConsumeMsg(args, "TopicQueue");
```
3.Run command for Send,The Routingkey is "TopicRK.one"


{% asset_img Send.png SendCmd %}
4.Run command for Receive

{% asset_img receive.png receive %}
5.Publish a message and The Routingkey is "TopicRK.one.123", The message will not be routed
{% asset_img RMQpng.png RMQpng %}




 
