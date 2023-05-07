# command ها 


## command ها به شرح زیر است:
### 1 ) SendNotificationImmediatelyCommand
این command  برای ارسال نوتیفیکیشن در همان لحظه استفاده می شود و ورودی این command به شرح زیر است: 

| Description  | Type| FieldName  |
|:----:|:-------------:|:---:|
| نوع مسیج ارسالی | string  | MessageTypeName  |
| پارامترهای ارسالی | Dictionary<string, string> | Parameters  |
| آیدی یوزر دریافت کننده پیام | long  | ReceiverUserId  |
| آیدی یوزر لاگین شده | long | ReceiverUserLoginId  |
| موبایل مخاطب | string  | PhoneNumber  |
| شناسه عملیات ثبت شده | long | ReferenceId  |
| شناسه کسب و کار | long?  | BusinessId  |
| شناسه شعبه | long? | BranchId  |

### 2 ) SendNotificationWithDueDateCommand
این command برای ارسال نوتیفیکیشن در یک زمان خاص استفاده می شود

| Description  | Type| FieldName  |
|:----:|:-------------:|:---:|
| نوع مسیج ارسالی | string  | MessageTypeName  |
| پارامترهای ارسالی | Dictionary<string, string>  | Parameters  |
| آیدی یوزر دریافت کننده پیام | long | ReceiverUserId  |
| آیدی یوزر لاگین شده | long | ReceiverUserLoginId  |
| موبایل مخاطب | string  | PhoneNumber  |
| شناسه عملیات ثبت شده | long | ReferenceId  |
| شناسه کسب و کار | long?  | BusinessId  |
| شناسه شعبه | long? | BranchId  |
| نام عملیات ثبت شده| string | ReferenceType  |
| زمان ارسال| DateTime  | DueDate  |
| زمان منقضی شدن در صورت ارسال نشدن| DateTime | ExpireDate  |

### 3 ) SendNotificationTaskDeleteCommand
با استفاده از این command می توان یک نوتیفیکیشن ثبت شده در سیستم را حذف کرد

| Description  | Type| FieldName  |
|:----:|:-------------:|:---:|
| شناسه عملیات ثبت شده | long | ReferenceId  |
| نام عملیات ثبت شده| string | ReferenceType  |
| نوع مسیج ارسالی| string | MessageTypeName  |


