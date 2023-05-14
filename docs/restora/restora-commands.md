# command های سیستم رستورا

## لیست رزروهای فعال یک مشتری - نسخه یک (GetActiveCustomerReservationsCommandV1)

این کامند لیست رزروهای آنلاین پیش پرداخت دار پذیرش شده یک کاربر را بر می گرداند و به شرح زیر می باشد:

ورودی:

```cs
public class GetActiveCustomerReservationsCommandV1 
    : IRequestMessage<GetActiveCustomerReservationsCommandResponseV1>
{
    public Guid CustomerGuid { get; set; }
    public Guid BranchGuid { get; set; }
    public DateTime ReservationTime { get; set; }
}
```

CustomerGuid: شناسه یکتای مشتری  
BranchGuid: شناسه یکتای شعبه  
ReservationTime: تاریخ رزرو

خروجی:

```cs
public class GetActiveCustomerReservationsCommandResponseV1
{
    public IEnumerable<ReservationV1> Reservations { get; set; }
    public string Error { get; set; }
}
```

Error: اگر خطایی وجود داشته باشد در این فیلد برگرداننده می شود  
Reservations: لیست رزروها که لیستی از موجودیت ReservationV1 به شکل زیر می باشد:

```cs
namespace AbrAbi.Common.Messaging.Messages.Restora.Models
{
    public class ReservationV1
    {
        public long Id { get; set; }
        public Guid Guid { get; set; }
        public DateTime StartTime { get; set; }
        public DateTime EndTime { get; set; }
        public bool IsPrepaid { get; set; }
        public int Guests { get; set; }
        public int Duration { get; set; }
        public double PrepaidAmount { get; set; }
        public bool PrepaidUsed { get; set; }
        public string Tables { get; set; }
    }
}
```

Id: شناسه عددی رزرو  
Guid: شناسه یکتای رزرو  
StartTime: زمان شروع رزرو  
EndTime: زمان پایان رزرو  
IsPrepaid: پیش پرداخت دارد یا خیر  
Guests: تعداد مهمان ها  
Duration: مدت زمان رزرو  
PrepaidAmount: مبلغ پیش پرداخت  
PrepaidUsed: این فیلد مشخص می کند که آیا پیش پرداخت این رزرو استفاده شده یا خیر  
Tables: نام میز رزرو

نمونه درخواست:

```cs
var response = await _messagingClient.Request<GetActiveCustomerReservationsCommandV1, GetActiveCustomerReservationsCommandResponseV1>(
    new GetActiveCustomerReservationsCommandV1
    {
        CustomerGuid = new Guid("68700794-F63B-46CA-B10B-D4B54C6081F8"),
        BranchGuid = new Guid("43620794-F63B-46CA-B10B-D4B54C6081B4"),
        ReservationTime = DateTime.UtcNow
    });
```

## استفاده از پیش پرداخت یک رزرو - نسخه یک (UseReservationPrepaymentCommandV1)

این کامند برای اعلام استفاده از پیش پرداخت یک رزرو استفاده می شود  
به این شکل است که پیش پرداخت هر رزرو یک بار بیشتر نمی تواند استفاده شود و بعد از استفاده شدن با این کامند می توان آن را به حالت استفاده شده برد  
این کامند به شرح زیر می باشد:

ورودی:

```cs
public class UseReservationPrepaymentCommandV1 
    : IRequestMessage<ResponseMessageBase>
{
    public Guid ReservationGuid { get; set; }
    public Guid BranchGuid { get; set; }
}
```

ReservationGuid: شناسه یکتای رزرو  
BranchGuid: شناسه یکتای شعبه

خروجی:

```cs
public class ResponseMessageBase : IResponseMessage
{
    public string Error { get; set; }
}
```

Error: اگر خطایی وجود داشته باشد در این فیلد برگرداننده می شود  
در صورتی که خطایی بر نگردد یعنی عملیات با موفقیت انجام شده

نمونه درخواست:

```cs
var response = await _messagingClient.Request<UseReservationPrepaymentCommandV1,  ResponseMessageBase>(
    new UseReservationPrepaymentCommandV1
    {
        CustomerGuid = new Guid("68700794-F63B-46CA-B10B-D4B54C6081F8"),
        BranchGuid = new Guid("43620794-F63B-46CA-B10B-D4B54C6081B4"),
    });
```
