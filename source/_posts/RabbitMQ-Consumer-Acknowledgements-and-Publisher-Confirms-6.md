title: RabbitMQ_Consumer Acknowledgements and Publisher Confirms(6)
author: Kevin Zhou
tags:
  - RabbitMQ
  - 'C#'
categories:
  - RabbitMQ
date: 2018-09-07 22:01:00
---
# Background:
Sometimes the consumers may crash or have connection issues. In this case,We may miss the queue messages  if we set the  acknowledgement to auto confirm, Please ses the following code:
Receive project code:
```Csharp
   ConsumeMsg(args, "DirectTestQueue");
    static void ConsumeMsg(string[] args, string queueName)
        {
            var factory = new ConnectionFactory() { HostName = "localhost" };
            using (var connection = factory.CreateConnection())
            {
                using (var channel = connection.CreateModel())
                {
                    channel.QueueDeclare(queue: queueName, durable: false, exclusive: false, autoDelete: false, arguments: null);
                    var consumer = new EventingBasicConsumer(channel);
                    consumer.Received += (model, ea) =>
                    {
                        var message = Encoding.UTF8.GetString(ea.Body);
                       Console.WriteLine("queue:"+queueName);
                        Console.WriteLine("[x] Received {0}", message);
                        Thread.Sleep(60000);
                        Console.WriteLine(" [x] Done");
                        //channel.BasicAck(deliveryTag: ea.DeliveryTag, multiple: false); //Manually send message acknowledgments

                    };
                    channel.BasicConsume(queue: queueName, autoAck: true, consumer: consumer);
                    Console.WriteLine(" Press [enter] to exit.");
                    Console.ReadLine();
                }
            }
        }

```
<!--more-->
This consumer will sleep 1 minutes once consumer receive th message. Then please press CTR+C to quit this console  application,you will find no message in the queue, We have missed the message due to consumer issues. How to handle this scenario?

1. We only  need to do a bit changes for our code，Please see the code as below:

```Csharp
 static void ConsumeMsg(string[] args, string queueName)
        {
            var factory = new ConnectionFactory() { HostName = "localhost" };
            using (var connection = factory.CreateConnection())
            {
                using (var channel = connection.CreateModel())
                {
                    channel.QueueDeclare(queue: queueName, durable: false, exclusive: false, autoDelete: false, arguments: null);
                    var consumer = new EventingBasicConsumer(channel);
                    consumer.Received += (model, ea) =>
                    {
                        var message = Encoding.UTF8.GetString(ea.Body);
                       Console.WriteLine("queue:"+queueName);
                        Console.WriteLine("[x] Received {0}", message);
                        Thread.Sleep(60000);
                        Console.WriteLine(" [x] Done");
                        channel.BasicAck(deliveryTag: ea.DeliveryTag, multiple: false); //Manually send message acknowledgments

                    };
                    channel.BasicConsume(queue: queueName, autoAck: false, consumer: consumer);
                    Console.WriteLine(" Press [enter] to exit.");
                    Console.ReadLine();
                }
            }
        }
```
2.Run the receive project and quit this console application once the message found in the screen

{% asset_img command.png command %}

2.Check the queue via RabbitMQ management once you quit this console application,You will find this queue message is still in the queue


{% asset_img queue1.png Unacked message %}

{% asset_img queue.png No consumed message %}