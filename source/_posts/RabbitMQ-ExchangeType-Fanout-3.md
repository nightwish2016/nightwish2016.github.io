title: RabbitMQ_ExchangeType Fanout(3)
author: Kevin Zhou
tags:
  - RabbitMQ
  - 'C#'
categories:
  - RabbitMQ
date: 2018-08-30 14:26:00
---
Exchange: Exchange is a rout map
ExchangeType: FanOut, the message will be pushed to all queues which bind to queue,Routingkey is not needed
1.Send project code:
```Csharp
FanOutExchangeTest(args);
static void FanOutExchangeTest(string[] args)
        {
            string queueName1 = "Queue1";
            string queueName2 = "Queue2";
            var factory = new ConnectionFactory() { HostName = "localhost" };
            using (var connection = factory.CreateConnection())
            {
                using (var channel = connection.CreateModel())
                {
                    string exchangeName = "FanOutTest";
                    string routingKey = "";
                    channel.ExchangeDeclare(exchangeName, ExchangeType.Fanout, false, false, null);
                    channel.QueueDeclare(queue: queueName1, durable: false, exclusive: false, autoDelete: false, arguments: null);
                    channel.QueueDeclare(queue: queueName2, durable: false, exclusive: false, autoDelete: false, arguments: null);
                    channel.QueueBind(queueName1, exchangeName, routingKey, null);
                    channel.QueueBind(queueName2, exchangeName, routingKey, null);
                    string message = args.Length > 0 ? exchangeName + " " + args[0] : "Hello RabbitMQ";
                    var body = Encoding.UTF8.GetBytes(message);
                    channel.BasicPublish(exchangeName, routingKey, basicProperties: null, body: body);
                    Console.WriteLine("[x] Sent {0} ", message);

                }
            }
        }
```
<!--more-->
2. Receive project code:
```Csharp
 ConsumeMsg(args, "Queue1");
 ConsumeMsg(args, "Queue2");
```
3.Run command for Send


{% asset_img Sendcmd.png SendCmd %}


{% asset_img FanOutExchange.png FanOutExchange %}

4.Run command for receive

{% asset_img Receivecmd.png Receivecmd %}

{% asset_img Queue.png Queue %}