---
layout: post
title: 使用Swagger上传文件至ASP.NET Core Web API
author: kinshines
date:   2018-09-28
categories: core
permalink: /archivers/upload-file-swagger-net-core
---

<p class="lead">在开发Web API的过程中，我们常常借助Swagger进行接口的测试，本文将演示如何配置使用Swagger上传文件至Web API
</p>

#### Controller
首先，我们在Controller层添加代码：

{% highlight java %}
        /// <summary>
        /// Action to upload file
        /// </summary>
        /// <param name="file"></param>
        [HttpPost]
        [Route("upload")]
        public void PostFile(IFormFile file)
        {
            var stream = file.OpenReadStream();
            var name = Path.GetFileName(file.FileName);

            //TODO: Save file
        }
{% endhighlight %}

此时，我们打开Swagger，发现IFormFile对应的文件输入域是一个文本框，很明显我们无法将一个文件上传到这个文本框里，而是期望借助文件上传控件来上传文件，因此，就需要自定义IOperationFilter 

#### IOperationFilter

{% highlight java %}
    public class FileOperation : IOperationFilter
    {
        public void Apply(Operation operation, OperationFilterContext context)
        {
            if (operation.OperationId.ToLower() == "apifileuploadpost")
            {
                operation.Parameters.Clear();//Clearing parameters
                operation.Parameters.Add(new NonBodyParameter
                {
                    Name = "File",
                    In = "formData",
                    Description = "Upload Image",
                    Required = true,
                    Type = "file"
                });
                operation.Consumes.Add("application/form-data");
            }
        }
    }
{% endhighlight %}

此处Filter将查找OperationId为"apifileuploadpost"(OperationId 的格式为[routeurl]+[httpverb])，它将清除IFormFile的属性接口，并添加一个文件上传控件，需要注意的是其中Name须与原来的参数名相同，In须为formData并且Type须为file，最后，需要添加一个新的consumes type为"application/form-data"

### Startup ConfigureServices

最后，将FileOperation注册到Swagger中，在Startup类中的ConfigureServices方法中，添加：

{% highlight java %}
            services.AddSwaggerGen(options =>
            {
                options.SingleApiVersion(new Info
                {
                    Version = "v1",
                    Title = "My awesome API",
                    Description = "My Awesome API by @janaks09",
                    TermsOfService = "NA",
                    Contact = new Contact() { Name="Your name", Email="your email", Url="your url" }
                });

                options.IncludeXmlComments(swaggerCommentXmlPath); //Includes XML comment file

                options.OperationFilter<FileOperation>();//Registering FileOperation config

                options.DescribeAllEnumsAsStrings();
            });
{% endhighlight %}

此时，文本输入框已转换为文件输入控件，就可以方便的进行文件上传了