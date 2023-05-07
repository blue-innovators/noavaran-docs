# Api ها 

## AbrAbi.Common.Api
در این پروژه یک تعداد Rest Api  مشترک داریم که برای بخش های مختلف سیستم می توان از آن ها استفاده کرد
در زیر Api های مربوط به notification  آن را بررسی میکنیم

### 1 ) CreateMessageTemplate
این api برای ساخت یک قالب پیام استفاده می شود و آدرس دسترسی آن به صورت زیر است:

| Address  | HttpType |
|:----:|:-------------:|
| Api/Message/CreateMessageTemplate| Post  |

ورودی این متد به اسم CreateMessageTemplateRequest به صورت زیر است :

| Description  | Type| FieldName  |
|:----:|:-------------:|:---:|
| شناسه مسیج ارسالی | long | MessageTypeId  |
| پروایدر مسیج| messageProvider | Provider  |
| قالب مسیج| string  | Template  |
| وضعیت قالب ایجاد شده| bool | IsActive  |

در حال حاضر MessageProvider در سیستم ما دو مدل است :

```
public enum MessageProvider : short
{
    Null = 0,
    NegarSmsUrl = 11,
    NegarSmsSoap = 12,
}
```

خروجی این متد به نام MessageTemplateDto و به صورت زیر است :

| Description  | Type| FieldName  |
|:----:|:-------------:|:---:|
| شناسه قالب | long  | Id  |
| پروایدر مسیج| messageProvider | Provider  |
|قالب پیام ارسالی | string | Template  |
|نوع پیام ارسالی| MessageType | Type  |

#### MessageType به صورت زیر تعریف شده است که مقادیر آن به صورت یک انتیتی در جدول ذخیره میشود.

| Description  | Type| FieldName  |
|:----:|:-------------:|:---:|
| شناسه نوع مسیج | long | Id  |
| نام نوع مسیج | string | Name  |
|نام فارسی نوع مسیج | string | Label  |

برای مثال نمونه تایپ های مسیج:

```
MobileVerification	تایید کاربر
ConfirmReserve	رزرو-تایید رزرو
CancelReserve	رزرو-کنسل رزرو
EditReserve	رزرو-ویرایش رزرو
NotificationBeforeReserve	رزرو-یادآوری رزرو
ResendMessage	ارسال دوباره پیام
```

### 2 ) EditMessageTemplate
این api برای ویرایش یک قالب پیام استفاده می شود و آدرس دسترسی آن به صورت زیر است:

| Address  | HttpType |
|:----:|:-------------:|
| Api/Message/EditMessageTemplate| Post  |

ورودی این متد به اسم EditMessageTemplateRequest به صورت زیر است :

| Description  | Type| FieldName  |
|:----:|:-------------:|:---:|
| شناسه مسیج ارسالی | long | Id  |
|شناسه تایپ مسیج ارسالی | long | MessageTypeId  |
| پروایدر مسیج| messageProvider | Provider  |
| قالب مسیج| string  | Template  |
| وضعیت قالب ایجاد شده| bool | IsActive  |

خروجی این متد به نام MessageTemplateDto و به صورت زیر است :

| Description  | Type| FieldName  |
|:----:|:-------------:|:---:|
| شناسه قالب | long  | Id  |
| پروایدر مسیج| messageProvider | Provider  |
|قالب پیام ارسالی | string | Template  |
|نوع پیام ارسالی| MessageType | Type  |

### 3 ) DeleteMessageTemplate
این api برای پاک کردن یک قالب پیام استفاده می شود و آدرس دسترسی آن به صورت زیر است:

| Address  | HttpType |
|:----:|:-------------:|
| Api/Message/DeleteMessageTemplate| Post  |

ورودی این متد به صورت زیر است :

| Description  | Type| FieldName  |
|:----:|:-------------:|:---:|
| شناسه مسیج ارسالی | long | Id  |

خروجی این متد به صورت زیر است :

| Description  | Type| FieldName  |
|:----:|:-------------:|:---:|
| موفقیت آمیز بودن عملیات| bool  | result  |

### 4 ) ChangeProviderActivation
این api برای تغییر فعال بود یا نبودن یک قالب پیام بر اساس پروایدر استفاده می شود و آدرس دسترسی آن به صورت زیر است:

| Address  | HttpType |
|:----:|:-------------:|
| Api/Message/ChangeProviderActivation| Patch  |

ورودی این متد به اسم ChangeProviderActivationRequest به صورت زیر است :

| Description  | Type| FieldName  |
|:----:|:-------------:|:---:|
| پروایدر مسیج| messageProvider | Provider  |
| وضعیت قالب ایجاد شده| bool | IsActive  |

خروجی این متد به صورت یک عدد است که نشان دهننده تعداد سطهای تغغیر یافته بر اساس پروایدر ارسالی است .

### 5 ) MessageTypes
این api  برای گرفتن لیست MessageType پیاده شده است و ادرس آن به صورت زیر است :

| Address  | HttpType |
|:----:|:-------------:|
| Api/Message/MessageTypes| Get  |

خروجی این متد به صورت لیستی از MessageType است .




