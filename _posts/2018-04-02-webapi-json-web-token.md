---
layout: post
title: WebAPI 应用Json Web Token（jwt）
author: kinshines
date:   2018-04-02
categories: webapi
permalink: /archivers/webapi-json-web-token
---

<p class="lead">WebAPI 不能像MVC项目中应用cookie机制在浏览器中保存用户验证信息，需要在每次Request请求中附件Token信息，常见的有方式有在请求的url后附加appKey参数信息，或者在header中附加token信息，本文将介绍一种目前较为成熟的Bearer 验证方式</p>

### 添加 MessageHandler

首先，新建WebAPI项目，安装package JwtAuthForWebAPI

        Install-Package JwtAuthForWebAPI -Version 2.0.7

然后，在WebApiConfig类中的Register方法中添加：
{% highlight java %}

var builder = new SecurityTokenBuilder();
     var jwtHandler = new JwtAuthenticationMessageHandler
     {
         AllowedAudience = "http://www.example.com",
         Issuer = "corp",
         SigningToken = builder.CreateFromCertificate("CN=JwtAuthForWebAPI Example"),
         CookieNameToCheckForToken = "ut"
     };

     config.MessageHandlers.Add(jwtHandler);

{% endhighlight %}

此处的参数也可通过web.config配置：

{% highlight xml %}

<configuration>
  <configSections>
    <section name="JwtAuthForWebAPI" type="JwtAuthForWebAPI.JwtAuthForWebApiConfigurationSection" />
  </configSections>
  <JwtAuthForWebAPI AllowedAudience="http://www.example.com" Issuer="corp" SymmetricKey="cXdlcnR5dWlvcGFzZGZnaGprbHp4Y3Zibm0xMjM0NTY=" />
  </configuration>

{% endhighlight %}

这样，WebApiConfig类中的Register方法修改如下：
{% highlight java %}

var builder = new SecurityTokenBuilder();
            var reader = new ConfigurationReader();
            config.MessageHandlers.Add(
                new JwtAuthenticationMessageHandler
                {
                    AllowedAudience = reader.AllowedAudience,
                    Issuer = reader.Issuer,
                    SigningToken = builder.CreateFromKey(reader.SymmetricKey)
                });

{% endhighlight %}

配置完毕后，在Controller中添加[Authorize]筛选器后，再次请求便得到401 Unauthorized响应，此时我们需要在header中添加token方能得到正常的响应，下一步将演示如何生成token

### 生成Token

一般将生成token的方法放在验证用户登录方法中，即判断用户登录成功后，将用户的ID，Name和Role等信息存放在Claim中，再生成token，返回给客户端，方法如下：

{% highlight java %}

        string CreateJwt(UserDto user)
        {
            var reader = new ConfigurationReader();
            var key = Convert.FromBase64String(reader.SymmetricKey);
            var credentials = new SigningCredentials(
                new InMemorySymmetricSecurityKey(key),
                "http://www.w3.org/2001/04/xmldsig-more#hmac-sha256",
                "http://www.w3.org/2001/04/xmlenc#sha256");

            var claimsIdentity = new ClaimsIdentity(new[]
            {
                    new Claim(ClaimTypes.NameIdentifier,user.ID,
                    new Claim(ClaimTypes.Name, user.Name)
            });

            var roleClaims = user.Roles.Select(r => new Claim(ClaimTypes.Role, r.RoleName));
            claimsIdentity.AddClaims(roleClaims);

            var tokenDescriptor = new SecurityTokenDescriptor
            {
                Subject = claimsIdentity,
                TokenIssuerName = reader.Issuer,
                AppliesToAddress = reader.AllowedAudience,
                SigningCredentials = credentials,
                Lifetime = new Lifetime(DateTime.UtcNow, DateTime.UtcNow.AddDays(10))
            };

            var tokenHandler = new JwtSecurityTokenHandler();
            var token = tokenHandler.CreateToken(tokenDescriptor);
            var tokenString = tokenHandler.WriteToken(token);

            return tokenString;
        }

{% endhighlight %}

### Header中添加token调用WebAPI
客户端在首次登录成功，得到token后，在后续的请求中，需在header中附加token信息，具体附加方式为：

在header中添加key:Authorization,value:Bearer token，注意此处Bearer和token中间有一个空格

假如上一步得到的token为：eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJuYW1laWQiOiJjM2FiYjU2Yy1mYTEzLTQ3M2MtODY2NC00MjQzZWIxY2UwYWIiLCJ1bmlxdWVfbmFtZSI6ImFkbWluIiwiZ3JvdXBzaWQiOiJDR1EiLCJyb2xlIjpbIlVzZXIiLCJBZG1pbiJdLCJpc3MiOiJjb3JwIiwiYXVkIjoiaHR0cDovL3d3dy5leGFtcGxlLmNvbSIsImV4cCI6MTUyMzI2MDYwMCwibmJmIjoxNTIyMzk2NjAwfQ.DoGnM87RfaqB9amFOjxJaODDWimlITHhh7LY5GZEI8Q

则请求如下：

        curl -X GET "http://localhost:49221/api/Value" -H "accept: application/json" -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJuYW1laWQiOiJjM2FiYjU2Yy1mYTEzLTQ3M2MtODY2NC00MjQzZWIxY2UwYWIiLCJ1bmlxdWVfbmFtZSI6ImFkbWluIiwiZ3JvdXBzaWQiOiJDR1EiLCJyb2xlIjpbIlVzZXIiLCJBZG1pbiJdLCJpc3MiOiJjb3JwIiwiYXVkIjoiaHR0cDovL3d3dy5leGFtcGxlLmNvbSIsImV4cCI6MTUyMzI2MDYwMCwibmJmIjoxNTIyMzk2NjAwfQ.DoGnM87RfaqB9amFOjxJaODDWimlITHhh7LY5GZEI8Q"

若客户端是C#代码，参考如下：

{% highlight java %}

    HttpRequestHeaders headers;
    headers.Authorization = new AuthenticationHeaderValue("Bearer", token);

{% endhighlight %}

### Swagger 配置
WebAPI 添加权限验证后，Swagger调用也要附加token信息，方式如下：

安装Swagger-Net

        Install-Package Swagger-Net

修改SwaggerConfig类中Register方法：

{% highlight java %}

    //c.ApiKey("apiKey", "header", "API Key Authentication");

{% endhighlight %}
        
修改为：

{% highlight java %}

    c.ApiKey("Authorization", "header", "Filling bearer token here");

{% endhighlight %}

在Swagger的Authorization 页面中，添加Value值：Bearer token 即可