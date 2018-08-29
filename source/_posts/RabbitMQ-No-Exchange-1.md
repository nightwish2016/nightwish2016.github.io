title: RabbitMQ_No Exchange (1)
author: Kevin Zhou
date: 2018-08-29 22:18:45
tags:
---
### No Exchange test
1.Install RabbitMQ :https://www.rabbitmq.com/download.html 
2.Create two .net core projects for testing: Send and Receive
3.Send project Code:
```Csharp
   static void Main(string[] args)
        {
           NoExchangeTest(args,"NoExchangeQueue");           
        }     
         static void NoExchangeTest(string[] args,string queuename)
        {
            var factory = new ConnectionFactory() { HostName = "localhost" };
            using (var connection = factory.CreateConnection())
            {
                using (var channel = connection.CreateModel())
                {
                    channel.QueueDeclare(queue: queuename, durable: false, exclusive: false, autoDelete: false, arguments: null);

                    string message = args.Length > 0 ? args[0] : "Hello RabbitMQ";
                    var body = Encoding.UTF8.GetBytes(message);
                    channel.BasicPublish(exchange: "", routingKey: queuename, basicProperties: null, body: body); 
                    Console.WriteLine("[x] Sent {0} ", message);

                }
            }
        }
```
<!--more-->

  Receive project code:
  ```Csharp
  static void Main(string[] args)
        {
             ConsumeMsg(args, "NoExchangeQueue");
           
        }
        
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
                        Thread.Sleep(6000);
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
4.Run Receive:
  ```
  dotnet run 
  ```
  

{% asset_img pasted-1.png This is an image %}
5.Run Send:
  ```
  dotnet run "aaa"
  ```
  

{% asset_img pasted-0.png This is an image %}
6.Open RabitMQ admin :http://localhost:15672/ usrname :guest password:guest
7.Click queue tab,you will find a new queue was created named "NoExchangeQueue"
{% asset_img NoExchangeQueue.png This is an image %}
