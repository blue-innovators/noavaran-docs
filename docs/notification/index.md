# اطلاعات اولیه سیستم

## اطلاعات اولیه سیستم
### 1 AbrAbi.Common.Data
این پروژه مربوط به کانفیگ ها و migration  دیتابیس موارد عمومی از جمله notification ها می باشد
### 2 AbrAbi.NotificationService
این پروژه شامل command هایی است که با فراخوانی آن ها می توان notification  های گوناگون(پیامک،ایمیل و ...) را به روش های گوناگون (در لحظه، زمانبندی شده و ...) ارسال یا حذف کرد
Command های این پروژه از طریق پیاده سازی های اینترفیس  ICommandMessagingClient  که در پروژه AbrAbi.Common.Messaging است انجام می شود
### 3 AbrAbi.Common.Api
این پروژه شامل تعدادی Api برای یک سری از کارهای عمومی می باشد.
