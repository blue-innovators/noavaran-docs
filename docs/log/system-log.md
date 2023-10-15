# ثبت system Log

<p style="text-align:justify;">
 لاگ های سیستم به روش های مختلف قابل ثبت هستند. برای مثال ما از کتابخانه seq برای ثبت لاگ استفاده میکنیم.
 برای جمع آوری بهتر اطلاعات و استفاده مفیدتر از داده ها ما از engine  الستیک برای ثبت لاگ استفاده میکنیم. برای پیاده سازی این روش در هر سیستمی ابتدا باید پکیج زیر نصب شود :
</p>

Serilog.Sinks.Elasticsearch
در حال حاضر ورژن این پکیج  Version="9.0.3" است

<p style="text-align:justify;">
 در هر نرم افزاری با تغییرات زیر در appsetting و program.cs اعمال شود.
</p>

### appsetting :
در این قسمت تکه کد زیر اضافه شود  این تنظیم مربوط به آدرس سرور و یوزر و پسورد برای ارتباط با سرور است.

```cs
  "Elasticsearch": {
    "uri": "https://backend:123456@10.50.60.13:9200",
    "disabled": false
  },
```

### program.cs :
 در این فایل در قسمت main وبخش logger باید تنظیمات مربوط به elasticSearch برای ارتباط با سرور آن اضافه شود :

```cs
 var loggerConfigurations = new LoggerConfiguration()
        .ReadFrom.Configuration(builder.Configuration)
        .Enrich.WithProperty("Application", DomainConstants.Systems.AuthServer)
        .Enrich.WithProperty("AppDomain", Environment.GetEnvironmentVariable("AppDomain"));
    var isElasticSearchDisabled = builder.Configuration.GetSection("Elasticsearch").GetValue<bool?>("disabled") == true;
        if (isElasticSearchDisabled == false)
        {
            var elasticNodeUrl = builder.Configuration.GetSection("Elasticsearch").GetValue<string>("uri");
            loggerConfigurations = loggerConfigurations
              .WriteTo.Elasticsearch(new Serilog.Sinks.Elasticsearch.ElasticsearchSinkOptions(new Uri(elasticNodeUrl))
                {
                    IndexFormat = "logs-{0:yyyy.MM}",
                    ModifyConnectionSettings = config => config.ServerCertificateValidationCallback(
                            (o, cert, chain, errors) => { return true; })
                });
        }
        Log.Logger = loggerConfigurations.CreateLogger();
```     

