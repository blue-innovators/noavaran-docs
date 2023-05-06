# command ها 

## 1 ) اطلاعات اولیه سیستم
### 1-1 AbrAbi.Common.Data
این پروژه مربوط به کانفیگ ها و migration  دیتابیس موارد عمومی از جمله notification ها می باشد
### 2-1 AbrAbi.NotificationService
این پروژه شامل command هایی است که با فراخوانی آن ها می توان notification  های گوناگون(پیامک،ایمیل و ...) را به روش های گوناگون (در لحظه، زمانبندی شده و ...) ارسال یا حذف کرد
Command های این پروژه از طریق پیاده سازی های اینترفیس  ICommandMessagingClient  که در پروژه AbrAbi.Common.Messaging است انجام می شود
### 3-1	AbrAbi.Common.Api
این پروژه شامل تعدادی Api برای یک سری از کارهای عمومی می باشد


## 2 ) command ها به شرح زیر است:
### 1-2 SendNotificationImmediatelyCommand
این command  برای ارسال نوتیفیکیشن در همان لحظه استفاده می شود و ورودی این command به شرح زیر است: 

| Description  | Type| FieldName  |
|:----:|:-------------:|:---:|
| نوع مسیج ارسالی | string         | MessageTypeName  |
| پارامترهای ارسالی | Dictionary<string, string>        | Parameters  |
| آیدی یوزر دریافت کننده پیام | long         | ReceiverUserId  |
| آیدی یوزر لاگین شده | long        | ReceiverUserLoginId  |
| موبایل مخاطب | string         | PhoneNumber  |
| شناسه عملیات ثبت شده | long        | ReferenceId  |
| شناسه کسب و کار | long?        | BusinessId  |
| شناسه شعبه | long?        | BranchId  |



