---
layout: post
title: ServiceStack.Redis 使用总结
author: kinshines
date:   2017-02-15
categories: c#
permalink: /archivers/ServiceStack-Redis-Utility
---

<p class="lead">本文是对ServiceStack.Redis的使用总结.</p>

[ServiceStack.Redis](https://servicestack.net/redis)是.NET平台下非常流行的Redis客户端，遗憾的是它在版本4之后开始收费，本文将以3.9.71版本为基础总结Redis使用操作。

完整代码参见[Github-ServiceStack.Redis.Utility](https://github.com/kinshines/ServiceStack.Redis.Utility)

NuGet Packages[Nuget-RedisUtility](https://www.nuget.org/packages/RedisUtility)

### 配置客户端管理器
{% highlight java %}
using System;
using System.Collections.Generic;
using System.Configuration;
using System.Linq;
using System.Text;

namespace ServiceStack.Redis.Utility
{
    public class RedisClientManager
    {
        private static PooledRedisClientManager _defaultManager;
        private static readonly Dictionary<string, PooledRedisClientManager> ManagerDictionary;

        static RedisClientManager()
        {
            _defaultManager = CreateDefaultManager();
            ManagerDictionary = new Dictionary<string, PooledRedisClientManager>();
        }
        private static string[] SplitString(string strSource, string split)
        {
            return strSource.Split(split.ToArray(), StringSplitOptions.RemoveEmptyEntries);
        }

        private static PooledRedisClientManager CreateDefaultManager()
        {
            string defaultRedisServer = ConfigurationManager.AppSettings["DefaultRedisServer"];
            string defaultReadRedisServer = ConfigurationManager.AppSettings["DefaultReadRedisServer"];
            string defaultWriteRedisServer = ConfigurationManager.AppSettings["DefaultWriteRedisServer"];
            if (string.IsNullOrEmpty(defaultReadRedisServer))
            {
                defaultReadRedisServer = defaultRedisServer;
            }
            if (string.IsNullOrEmpty(defaultWriteRedisServer))
            {
                defaultWriteRedisServer = defaultRedisServer;
            }
            string[] writeServerList = SplitString(defaultWriteRedisServer, ",");
            string[] readServerList = SplitString(defaultReadRedisServer, ",");
            return new PooledRedisClientManager(readServerList, writeServerList,
                new RedisClientManagerConfig()
                {
                    AutoStart = true,
                    MaxWritePoolSize = 100,
                    MaxReadPoolSize = 100
                });
        }

        /// <summary>
        /// 创建redis应用程序池
        /// </summary>
        /// <param name="connectStr"></param>
        /// <returns></returns>
        private static PooledRedisClientManager CreateManager(string connectStr)
        {
            string[] arrStr = connectStr.Split(':');
            string host = arrStr.Length > 0 ? arrStr[0] : "";
            int port = 6379;
            if (arrStr.Length > 1)
            {
                int.TryParse(arrStr[1], out port);
            }

            long db = 0;
            if (arrStr.Length > 2)
            {
                long.TryParse(arrStr[2], out db);
            }

            RedisClientManagerConfig config = new RedisClientManagerConfig { MaxReadPoolSize = 100, MaxWritePoolSize = 100, AutoStart = true };
            return new PooledRedisClientManager(new[] { host + ":" + port }, new[] { host + ":" + port },
                config, db, null, null);
        }

        public static IRedisClient GetClient(string connectStr = null)
        {
            if (string.IsNullOrEmpty(connectStr))
            {
                if (_defaultManager == null)
                {
                    _defaultManager = CreateDefaultManager();
                }
                return _defaultManager.GetClient();
            }

            connectStr = connectStr.Trim();
            var manager = ManagerDictionary[connectStr];
            if (manager == null)
            {
                manager = CreateManager(connectStr);
                ManagerDictionary[connectStr] = manager;
            }
            return manager.GetClient();
        }
    }
}
{% endhighlight %}

### Redis操作类
{% highlight java %}
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using NLogUtility;
using ServiceStack.Text;

namespace ServiceStack.Redis.Utility
{
    public class Redis
    {

        #region hash
        /// <summary>
        /// hashid key value的递增
        /// </summary>
        /// <param name="hashId"></param>
        /// <param name="key"></param>
        /// <param name="incrementBy"></param>
        /// <param name="connectStr"></param>
        public static long IncreameValue(string hashId, string key, int incrementBy,string connectStr=null)
        {
            long res = 0;
            try
            {
                using (var redisClient = RedisClientManager.GetClient(connectStr))
                {
                    res = redisClient.IncrementValueInHash(hashId, key, incrementBy);
                }
            }
            catch (Exception ex)
            {
                Logger.Error(ex, "hashId:{0},key:{1},incrementBy:{2},connectStr:{3}", hashId, key, incrementBy,connectStr);
            }

            return res;
        }

        /// <summary>
        /// 通过hasId删除缓存
        /// </summary>
        /// <param name="hashId"></param>
        /// <param name="connectStr"></param>
        /// <returns></returns>
        public static bool RemoveHashRedisValueByHashId(string hashId,string connectStr=null)
        {
            bool result = true;
            try
            {
                using (var redisClient = RedisClientManager.GetClient(connectStr))
                {
                    List<string> keyList = redisClient.GetHashKeys(hashId);
                    foreach (string key in keyList)
                    {
                        var removed = redisClient.RemoveEntryFromHash(hashId, key);
                        result = result && removed;
                    }
                }
            }
            catch (Exception ex)
            {
                Logger.Error(ex, "hashId:{0},connectStr:{1}", hashId, connectStr);
                result = false;
            }

            return result;
        }

        /// <summary>
        /// 通过hashId和key删除缓存
        /// </summary>
        /// <param name="hash"></param>
        /// <param name="key"></param>
        /// <param name="connectStr"></param>
        /// <returns></returns>
        public static bool RemoveHasRedisValueByHasdIdAndKey(string hash, string key, string connectStr = null)
        {
            bool result = false;
            try
            {
                using (var redisClient = RedisClientManager.GetClient(connectStr))
                {
                    if (redisClient.HashContainsEntry(hash, key))
                    {
                        result = redisClient.RemoveEntryFromHash(hash, key);
                    }
                    else
                    {
                        result = true;
                    }
                }
            }
            catch (Exception ex)
            {
                Logger.Error(ex, "hashId:{0},key:{1},connectStr:{2}", hash, key, connectStr);
            }

            return result;
        }

        /// <summary>
        /// 设置HashRedis
        /// </summary>
        /// <param name="hash"></param>
        /// <param name="key"></param>
        /// <param name="value"></param>
        /// <param name="connectStr"></param>
        /// <returns></returns>
        public static bool SetHashRedisValue(string hash, string key, string value, string connectStr = null)
        {
            bool result = false;
            try
            {
                using (var redisClient = RedisClientManager.GetClient(connectStr))
                {
                    result = redisClient.SetEntryInHash(hash, key, value);
                }
            }
            catch (Exception ex)
            {
                Logger.Error(ex, "hashId:{0},key:{1},value:{2},connectStr:{3}", hash, key, value, connectStr);
            }

            return result;
        }

        /// <summary>
        /// 获取HashId下的所有Keys
        /// </summary>
        /// <param name="hash"></param>
        /// <param name="connectStr"></param>
        /// <returns></returns>
        public static List<string> GetHashKeys(string hash, string connectStr = null)
        {
            List<string> value = null;
            try
            {
                using (var redisClient = RedisClientManager.GetClient(connectStr))
                {
                    value = redisClient.GetHashKeys(hash);
                }
            }
            catch (Exception ex)
            {
                Logger.Error(ex, "hashId:{0},connectStr:{1}", hash, connectStr);
            }

            return value ?? new List<string>();
        }

        /// <summary>
        /// 获取Hashredis
        /// </summary>
        /// <param name="hash"></param>
        /// <param name="key"></param>
        /// <param name="connectStr"></param>
        /// <returns></returns>
        public static string GetHashRedisValue(string hash, string key, string connectStr = null)
        {
            string value = null;
            try
            {
                using (var redisClient = RedisClientManager.GetClient(connectStr))
                {
                    value = redisClient.GetValueFromHash(hash, key);
                }
            }
            catch (Exception ex)
            {
                Logger.Error(ex, "hashId:{0},key:{1},connectStr:{2}", hash, key, connectStr);
            }
            return value;
        }

        /// <summary>
        /// 判断hashid是否存在
        /// </summary>
        /// <param name="hash"></param>
        /// <param name="key"></param>
        /// <param name="connectStr"></param>
        /// <returns></returns>
        public static bool HashContainsEntry(string hash, string key, string connectStr = null)
        {
            bool exist = false;
            try
            {
                using (var redisClient = RedisClientManager.GetClient(connectStr))
                {
                    exist = redisClient.HashContainsEntry(hash, key);
                }
            }
            catch (Exception ex)
            {
                Logger.Error(ex, "hashId:{0},key:{1},connectStr:{2}", hash, key, connectStr);
            }
            return exist;
        }

        #endregion


        /// <summary>
        /// 通过keytype删除缓存
        /// </summary>
        /// <param name="keyType"></param>
        /// <param name="connectStr"></param>
        /// <returns></returns>
        public static bool RemoveRedisValueByKeyType(string keyType, string connectStr = null)
        {
            bool result = true;
            try
            {
                using (var redisClient = RedisClientManager.GetClient(connectStr))
                {
                    List<string> keyList = redisClient.SearchKeys(keyType + "*");
                    foreach (string key in keyList)
                    {
                        if (key.StartsWith(keyType, StringComparison.OrdinalIgnoreCase))
                        {
                            var removed = redisClient.Remove(key);
                            result = result && removed;
                        }
                    }
                }
            }
            catch (Exception ex)
            {
                result = false;
                Logger.Error(ex, "keyType:{0},connectStr:{1}", keyType, connectStr);
            }
            return result;
        }

        /// <summary>
        /// 通过key，删除redis
        /// </summary>
        /// <param name="key"></param>
        /// <param name="connectStr"></param>
        /// <returns></returns>
        public static bool RemoveRedisValueByKey(string key, string connectStr = null)
        {
            bool result = false;
            try
            {
                using (var redisClient = RedisClientManager.GetClient(connectStr))
                {
                    result = redisClient.Remove(key);
                }
            }
            catch (Exception ex)
            {
                Logger.Error(ex, "key:{0},connectStr:{1}", key, connectStr);
            }
            return result;
        }

        /// <summary>
        /// 获取redis
        /// </summary>
        /// <param name="key"></param>
        /// <param name="connectStr"></param>
        /// <returns></returns>
        public static string GetRedisValue(string key, string connectStr = null)
        {
            string value = null;
            try
            {
                using (var redisClient = RedisClientManager.GetClient(connectStr))
                {
                    value = redisClient.GetValue(key);
                }
            }
            catch (Exception ex)
            {
                Logger.Error(ex, "key:{0},connectStr:{1}", key, connectStr);
            }
            return value;
        }
        
        /// <summary>
        /// 获取redis
        /// </summary>
        /// <param name="key"></param>
        /// <param name="connectStr"></param>
        /// <returns></returns>
        public static object GetRedisObjectValue(string key, string connectStr = null)
        {
            object value = null;
            try
            {
                using (var redisClient = RedisClientManager.GetClient(connectStr))
                {
                    value = redisClient.Get<object>(key);
                }
            }
            catch (Exception ex)
            {
                Logger.Error(ex, "key:{0},connectStr:{1}", key, connectStr);
            }
            return value;
        }

        /// <summary>
        /// 获取redis
        /// </summary>
        /// <param name="key"></param>
        /// <param name="connectStr"></param>
        /// <returns></returns>
        public static T GetRedisValue<T>(string key, string connectStr = null)
        {
            T value = default(T);
            try
            {
                using (var redisClient = RedisClientManager.GetClient(connectStr))
                {
                    value = redisClient.Get<T>(key);
                }
            }
            catch (Exception ex)
            {
                Logger.Error(ex, "key:{0},connectStr:{1}", key, connectStr);
            }
            return value;
        }

        /// <summary>
        /// 过期时间
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="key"></param>
        /// <param name="value"></param>
        /// <param name="expiresAt"></param>
        /// <param name="connectStr"></param>
        /// <returns></returns>
        public static bool SetRedisValue<T>(string key, T value, DateTime expiresAt, string connectStr = null)
        {
            try
            {
                using (var redisClient = RedisClientManager.GetClient(connectStr))
                {
                    return redisClient.Set(key, value, expiresAt);
                }
            }
            catch (Exception ex)
            {
                Logger.Error(ex, "key:{0},value:{1},connectStr:{2}", key, JsonSerializer.SerializeToString(value), connectStr);
                return false;
            }
        }

        /// <summary>
        /// 设置缓存
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="key"></param>
        /// <param name="value"></param>
        /// <param name="expiresAt"></param>
        /// <param name="connectStr"></param>
        /// <returns></returns>
        public static bool SetRedisValue<T>(string key, T value, TimeSpan expiresAt, string connectStr = null)
        {
            try
            {
                using (var redisClient = RedisClientManager.GetClient(connectStr))
                {
                    return redisClient.Set(key, value, expiresAt);
                }
            }
            catch (Exception ex)
            {
                Logger.Error(ex, "key:{0},value:{1},connectStr:{2}", key, JsonSerializer.SerializeToString(value), connectStr);
                return false;
            }
        }

        /// <summary>
        /// 设置无过期时间缓存
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="key"></param>
        /// <param name="value"></param>
        /// <param name="connectStr"></param>
        /// <returns></returns>
        public static bool SetRedisValue<T>(string key, T value, string connectStr = null)
        {
            try
            {
                using (var redisClient = RedisClientManager.GetClient(connectStr))
                {
                    return redisClient.Set(key, value);
                }
            }
            catch (Exception ex)
            {
                Logger.Error(ex, "key:{0},value:{1},connectStr:{2}", key, JsonSerializer.SerializeToString(value), connectStr);
                return false;
            }
        }


        #region list

        /// <summary>
        /// 入列
        /// </summary>
        /// <param name="listId"></param>
        /// <param name="value"></param>
        /// <param name="connectStr"></param>
        /// <returns></returns>
        public static bool Enqueue(string listId, string value, string connectStr = null)
        {
            bool result = false;
            try
            {
                using (var redisClient = RedisClientManager.GetClient(connectStr))
                {
                    redisClient.EnqueueItemOnList(listId, value);
                    result = true;
                }
            }
            catch (Exception ex)
            {
                Logger.Error(ex, "listId:{0},value:{1},connectStr:{2}", listId, value, connectStr);
            }

            return result;
        }

        /// <summary>
        /// 入列
        /// </summary>
        /// <param name="listId"></param>
        /// <param name="value"></param>
        /// <param name="connectStr"></param>
        /// <returns></returns>
        public static bool Enqueue<T>(string listId, T value, string connectStr = null)
        {
            bool result = false;
            try
            {
                using (var redisClient = RedisClientManager.GetClient(connectStr))
                {
                    redisClient.EnqueueItemOnList(listId, JsonSerializer.SerializeToString(value));
                    result = true;
                }
            }
            catch (Exception ex)
            {
                Logger.Error(ex, "listId:{0},value:{1},connectStr:{2}", listId, value, connectStr);
            }

            return result;
        }

        /// <summary>
        /// 出列
        /// </summary>
        /// <param name="listId"></param>
        /// <param name="connectStr"></param>
        /// <returns></returns>
        public static T Dequeue<T>(string listId, string connectStr = null)
        {
            T value = default(T);
            try
            {
                using (var redisClient = RedisClientManager.GetClient(connectStr))
                {
                    var item = redisClient.DequeueItemFromList(listId);
                    if (item != null)
                    {
                        value = JsonSerializer.DeserializeFromString<T>(item);
                    }
                }
            }
            catch (Exception ex)
            {
                Logger.Error(ex, "listId:{0},connectStr:{1}", listId, connectStr);
            }
            return value;
        }
        
        /// <summary>
        /// 队列数量
        /// </summary>
        /// <param name="listId"></param>
        /// <param name="connectStr"></param>
        /// <returns></returns>
        public static long QueueCount(string listId, string connectStr = null)
        {
            try
            {
                using (var redisClient = RedisClientManager.GetClient(connectStr))
                {
                    return redisClient.GetListCount(listId);
                }
            }
            catch (Exception ex)
            {
                Logger.Error(ex, "listId:{0},connectStr:{1}", listId, connectStr);
                return 0;
            }
        }

        #endregion
    }
}
{% endhighlight %}

### 简单说明
1. 配置Redis服务器地址，在App.config或Web.config中添加配置节DefaultRedisServer
2. 如果Redis读写服务器地址不一致，需要分别设置配置节DefaultReadRedisServer，DefaultWriteRedisServer
3. 如果你的代码中需要连接多个Redis服务器，可以在直接调用Redis操作方法时传入相关地址，如果未传入地址信息，则使用配置节中配置的默认地址
4. Redis提供的功能十分强大，本文提供的Redis操作方法只是Redis操作的冰山一角，抛砖引玉，读者后续可以根据需要增加更多的Redis操作方法