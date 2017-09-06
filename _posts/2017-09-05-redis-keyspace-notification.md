---
layout: post
title: Redis 键空间通知 keyspace notification
author: kinshines
date:   2017-09-05
categories: db
permalink: /archivers/redis-keyspace-notification
---

<p class="lead">使用redis持久化数据时，当内存资源比较紧缺时，会导致部分使用不太频繁的redis数据丢失，这种情况下，需要监控redis的数据，以便在redis数据意外丢失的情况下，及时补救。另外还有一种应用场景是倒计时功能，例如淘宝的还剩几天自动确认收货，在这种情况下，需要程序在某个倒计时的时机自动触发。下面介绍的redis 键空间通知(keyspace notifications)的功能可以完成此类功能</p>

键空间通知使得客户端可以通过订阅频道或模式， 来接收那些以某种方式改动了 Redis 数据集的事件，例如redis的缓存过期事件，缓存移除事件等等，因为开启键空间通知功能需要消耗一些 CPU ， 所以在默认配置下， 该功能处于关闭状态，可以通过修改 redis.conf 文件， 或者直接使用 CONFIG SET 命令来开启或关闭键空间通知功能：

* 当 notify-keyspace-events 选项的参数为空字符串时，功能关闭
* 另一方面，当参数不是空字符串时，功能开启


notify-keyspace-events 的参数可以是以下字符的任意组合， 它指定了服务器该发送哪些类型的通知：

<table>
  <thead>
    <tr>
      <th>字符</th>
      <th>发送的通知</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>K</td>
      <td>键空间通知，所有通知以 __keyspace@<db>__ 为前缀</td>
    </tr>
    <tr>
      <td>E</td>
      <td>键事件通知，所有通知以 __keyevent@<db>__ 为前缀</td>
    </tr>
    <tr>
      <td>g</td>
      <td>DEL 、 EXPIRE 、 RENAME 等类型无关的通用命令的通知</td>
    </tr>
    <tr>
      <td>$</td>
      <td>字符串命令的通知</td>
    </tr>
    <tr>
      <td>l</td>
      <td>列表命令的通知</td>
    </tr>
    <tr>
      <td>s</td>
      <td>集合命令的通知</td>
    </tr>
    <tr>
      <td>h</td>
      <td>哈希命令的通知</td>
    </tr>
    <tr>
      <td>z</td>
      <td>有序集合命令的通知</td>
    </tr>
    <tr>
      <td>x</td>
      <td>过期事件：每当有过期键被删除时发送</td>
    </tr>
    <tr>
      <td>e</td>
      <td>驱逐(evict)事件：每当有键因为 maxmemory 政策而被删除时发送</td>
    </tr>
    <tr>
      <td>A</td>
      <td>参数 g$lshzxe 的别名，即所有通知</td>
    </tr>
    </tbody>
  </table>


这里为了演示，我开启了所有通知，在生产环境下，只需要开启需要的通知即可，命令如下：


        CONFIG SET notify-keyspace-events KEA

然后是订阅客户端代码：

{% highlight java %}
using System;

namespace StackExchange.Redis
{
    static class RedisKeyspaceNotifications
    {
        /// <summary>
        /// NOTE: For this sample to work, you need to go to the Azure Portal and configure keyspace notifications with "Kxge$" to
        ///       1) turn on expiration notifications (x), 
        ///       2) general command notices (g) and 
        ///       3) Evicted events (e).
        ///       4) STRING operations ($).
        /// IMPORTANT - MAKE SURE YOU UNDERSTAND THE PERFORMANCE IMPACT OF TURNING ON KEYSPACE NOTIFICATIONS BEFORE PROCEEDING
        /// See http://redis.io/topics/notifications for more details
        public static void NotificationsExample(ConnectionMultiplexer connection)
        {
            var subscriber = connection.GetSubscriber();
            int db = 0; //what Redis DB do you want notifications on?
            string notificationChannel = "__keyspace@" + db + "__:*";

            //you only have to do this once, then your callback will be invoked.
            subscriber.Subscribe(notificationChannel, (channel, notificationType) =>
            {
                // IS YOUR CALLBACK NOT GETTING CALLED???? 
                // -> See comments above about enabling keyspace notifications on your redis instance
                var key = GetKey(channel);
                switch (notificationType) // use "Kxge" keyspace notification options to enable all of the below...
                {
                    case "expire": // requires the "Kg" keyspace notification options to be enabled
                        Console.WriteLine("Expiration Set for Key: " + key);
                        break;
                    case "expired": // requires the "Kx" keyspace notification options to be enabled
                        Console.WriteLine("Key EXPIRED: " + key);
                        break;
                    case "rename_from": // requires the "Kg" keyspace notification option to be enabled
                        Console.WriteLine("Key RENAME(From): " + key);
                        break;
                    case "rename_to": // requires the "Kg" keyspace notification option to be enabled
                        Console.WriteLine("Key RENAME(To): " + key);
                        break;
                    case "del": // requires the "Kg" keyspace notification option to be enabled
                        Console.WriteLine("KEY DELETED: " + key);
                        break;
                    case "evicted": // requires the "Ke" keyspace notification option to be enabled
                        Console.WriteLine("KEY EVICTED: " + key);
                        break;
                    case "set": // requires the "K$" keyspace notification option to be enabled for STRING operations
                        Console.WriteLine("KEY SET: " + key);
                        break;
                    default:
                        Console.WriteLine("Unhandled notificationType: " + notificationType);
                        break;
                }
            });

            Console.WriteLine("Subscribed to notifications...");

            // setup for delete notification example
            connection.GetDatabase(db).StringSet("DeleteExample", "Anything");

            // key rename callbacks example
            connection.GetDatabase(db).StringSet("{RenameExample}From", "Anything");
            connection.GetDatabase(db).KeyRename("{RenameExample}From", "{RenameExample}To");

            var random = new Random();
            //add some keys that will expire to test the above callback configured above.
            for (int i = 0; i < 10; i++)
            {
                var expiry = TimeSpan.FromSeconds(random.Next(2, 10));
                connection.GetDatabase(db).StringSet("foo" + i, "bar", expiry);
            }

            // should result in a delete notification callback.
            connection.GetDatabase(db).KeyDelete("DeleteExample");
        }

        private static string GetKey(string channel)
        {
            var index = channel.IndexOf(':');
            if (index >= 0 && index < channel.Length - 1)
                return channel.Substring(index + 1);

            //we didn't find the delimeter, so just return the whole thing
            return channel;
        }
    }
}
{% endhighlight %}


以上代码需要引用 [StackExchange.Redis](https://www.nuget.org/packages/StackExchange.Redis)