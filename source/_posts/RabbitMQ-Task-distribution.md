title: RabbitMQ_Task distribution(7)
author: Kevin Zhou
tags:
  - RabbitMQ
  - 'C#'
categories:
  - RabbitMQ
date: 2018-09-10 11:14:00
---
Firt of all ,Let's take a look the following receive code,You will find that If the message is 1,The consumer will sleep 1 seconds, if the message is 10 ,The consumer will sleep 10 seconds.
```Csharp
TasksDistributeLimit(args, "DirectTestQueue");
static void TasksDistributeLimit(string[] args, string queueName)
        {
            var factory = new ConnectionFactory() { HostName = "localhost" };
            using (var connection = factory.CreateConnection())
            {
                using (var channel = connection.CreateModel())
                {
                    channel.QueueDeclare(queue: queueName, durable: false, exclusive: false, autoDelete: false, arguments: null);
                  //  channel.BasicQos(prefetchSize: 0, prefetchCount: 1, global: false);
                    var consumer = new EventingBasicConsumer(channel);
                    consumer.Received += (model, ea) =>
                    {
                        var message = Encoding.UTF8.GetString(ea.Body);
                        Console.WriteLine("queue:" + queueName);
                        Console.WriteLine("[x] Received {0}", message);
                        int len = int.Parse(message);                        
                        Console.WriteLine("Sleep:{0} seconds", len);
                        Thread.Sleep(len*1000);
                        Console.WriteLine(" [x] Done");
                        channel.BasicAck(deliveryTag: ea.DeliveryTag, multiple: false); //Manually confirm message

                    };
                    channel.BasicConsume(queue: queueName, autoAck: false, consumer: consumer);
                    Console.WriteLine(" Press [enter] to exit.");
                    Console.ReadLine();
                }
            }
        }
    }
```
<!--more-->
Let's mock the following scenario:
1.Launch two consumers:
  ```
  dotnet run
  ```
2.Send a message to make consumer sleep 1 second
3.Send a message to make consumer sleep 2 seconds
4.Send a message to make consumer sleep 3 seconds
5.Send a message to make consumer sleep 4 seconds
6.Send a message to make consumer sleep 60 seconds
7.Send a message to make consumer sleep 1 seconds
8.Send a message to make consumer sleep 2 seconds

Result:
Consumer1:
{% asset_img consumer1.png consumer1 %}

Consumer2:
{% asset_img Consumer2.png Consumer2 %}

Conclusion:
You will find all messages will be ditributed equally until the consumer2 sleep 60s complete.Actually this distribution rule is not good for server since the other consumers are free, They could be able to consume these messages

How to improve the efficiency of comsumer processor:
Add this code in your consomer project,The consumer only fetch 1 message from queue until 
this requst is finished. The other messages which are not fetched will be processed by the other consumers.
```Charp
  channel.BasicQos(prefetchSize: 0, prefetchCount: 1, global: false);
```
See the detail code of receive:
```Charp
 static void TasksDistributeLimit(string[] args, string queueName)
        {
            var factory = new ConnectionFactory() { HostName = "localhost" };
            using (var connection = factory.CreateConnection())
            {
                using (var channel = connection.CreateModel())
                {
                    channel.QueueDeclare(queue: queueName, durable: false, exclusive: false, autoDelete: false, arguments: null);
                    channel.BasicQos(prefetchSize: 0, prefetchCount: 1, global: false);
                    var consumer = new EventingBasicConsumer(channel);
                    consumer.Received += (model, ea) =>
                    {
                        var message = Encoding.UTF8.GetString(ea.Body);
                        Console.WriteLine("queue:" + queueName);
                        Console.WriteLine("[x] Received {0}", message);
                        int len = int.Parse(message);                        
                        Console.WriteLine("Sleep:{0} seconds", len);
                        Thread.Sleep(len*1000);
                        Console.WriteLine(" [x] Done");
                        channel.BasicAck(deliveryTag: ea.DeliveryTag, multiple: false); //Manually confirm message

                    };
                    channel.BasicConsume(queue: queueName, autoAck: false, consumer: consumer);
                    Console.WriteLine(" Press [enter] to exit.");
                    Console.ReadLine();
                }
            }
```

Retest this scenario we just tested:
Result:
	Consumer1:   
    
{% asset_img Consumer1-1.png Consumer1-1 %}

    Consumer2:
    
 {% asset_img Consumer2-1.png Consumer2-1 %}

Conclusion:

The message will be distributed to which consumers depends on whether consumers are busy or not.