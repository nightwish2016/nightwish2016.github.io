title: RabbitMQ_ExchangeType Direct (二)
author: Kevin Zhou
date: 2018-08-29 11:07:47
tags:
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
 
![upload successful](/images/DirectSend.png)
 4.Run command for receive project
 
![upload successful](/images/DirctReceive.png)
 5.Check Exchange and queue
 
![upload successful](/images/DirctExchange.png)
 
![upload successful](/images/image.png)
6. Publish a meesage via RabbitMQ management,  You will find the message can't be  published if the routingkey is not equal with "DirectRK"

![upload successful](/images/error.png)