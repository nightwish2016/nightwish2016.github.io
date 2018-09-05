title: RabbitMQ_ExchangeType Direct(2)
author: Kevin Zhou
tags:
  - RabbitMQ
  - 'C#'
categories:
  - RabbitMQ
date: 2018-08-29 22:22:00
---
## DirectExchangeTest
Exchange: Exchange is a rout map
ExchangeType: Direct, the message will be pushed to specified queue which have binded with the exchange.
1.Send project code:
```Csharp
 DirectExchangeTest(args, "DirectTestQueue");
static void DirectExchangeTest(string[] args,string quename)
        {
            var factory = new ConnectionFactory() { HostName = "localhost" };
            using (var connection = factory.CreateConnection())
            {
                using (var channel = connection.CreateModel())
                {
                    string exchangeName = "DirectTest";
                    string routingKey = "DirectRK";
                    channel.ExchangeDeclare(exchangeName,ExchangeType.Direct,false,false,null);
                    channel.QueueDeclare(queue: "DirectTestQueue", durable: false, exclusive: false, autoDelete: false, arguments: null);
                    channel.QueueBind(quename, exchangeName, routingKey, null);
                    string message = args.Length > 0 ? exchangeName+" "+args[0] : "Hello RabbitMQ";
                    var body = Encoding.UTF8.GetBytes(message);
                    channel.BasicPublish(exchangeName,routingKey, basicProperties: null, body: body);
                    Console.WriteLine("[x] Sent {0} ", message);

                }
            }
        }
  ```
 2.Receive project code:
 ```Csharp
  ConsumeMsg(args, "DirectTestQueue");
 ```
 <!--more-->
 3.Run command for send project
 
 {% asset_img DirectSend.png This is an image %}
 
 4.Run command for receive project
 
 
 {% asset_img DirctReceive.png This is an image %}
 5.Check Exchange and queue
  {% asset_img DirctExchange.png This is an image %}
 
 {% asset_img image.png This is an image %}
6. Publish a meesage via RabbitMQ management,  You will find the message can't be  published if the routingkey is not equal with "DirectRK"
{% asset_img error.png This is an image %}
