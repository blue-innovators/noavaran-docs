# command های سیستم رستورا

command ها به شرح زیر می باشند:

## لیست رزروهای فعال یک مشتری (GetActiveCustomerReservationsCommand)

این command لیست رزروهای آنلاین پیش پرداخت دار پذیرش شده یک کاربر را بر می گرداند و به شرح زیر می باشد:

ورودی:

```cs
public class GetActiveCustomerReservationsCommand : IRequestMessage<GetActiveCustomerReservationsCommandResponse>
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
public class GetActiveCustomerReservationsCommandResponse : IResponseMessage
{
    public IEnumerable<ReservationDto> Reservations { get; set; }
    public string Error { get; set; }
}
```

Error: اگر خطایی وجود داشته باشد در این فیلد برگرداننده می شود  
Reservations: لیست رزروها که لیستی از موجودیت ReservationDto به شکل زیر می باشد:

```cs
public class ReservationDto
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
var response = await _messagingClient.Request<GetActiveCustomerReservationsCommand, GetActiveCustomerReservationsCommandResponse>(
    new GetActiveCustomerReservationsCommand
        {
            CustomerGuid = new Guid("68700794-F63B-46CA-B10B-D4B54C6081F8"),
            BranchGuid = new Guid("43620794-F63B-46CA-B10B-D4B54C6081B4"),
            ReservationTime = DateTime.UtcNow
        });
```

## استفاده از پیش پرداخت یک رزرو (UseReservationPrepaymentCommand)

این command برای اعلام استفاده از پیش پرداخت یک رزرو استفاده می شود  
به این شکل است که پیش پرداخت هر رزرو یک بار بیشتر نمی تواند استفاده شود و بعد از استفاده شدن با این command می توان آن را به حالت استفاده شده برد  
این command به شرح زیر می باشد:

ورودی:

```cs
public class UseReservationPrepaymentCommand : IRequestMessage<UseReservationPrepaymentCommandResponse>
{
    public Guid ReservationGuid { get; set; }
    public Guid BranchGuid { get; set; }
}
```

ReservationGuid: شناسه یکتای رزرو  
BranchGuid: شناسه یکتای شعبه

خروجی:

```cs
public class UseReservationPrepaymentCommandResponse : IResponseMessage
{
    public string Error { get; set; }
}
```

Error: اگر خطایی وجود داشته باشد در این فیلد برگرداننده می شود  
در صورتی که خطایی بر نگردد یعنی عملیات با موفقیت انجام شده

نمونه درخواست:

```cs
var response = await _messagingClient.Request<UseReservationPrepaymentCommand,  UseReservationPrepaymentCommandResponse>(
    new UseReservationPrepaymentCommand
        {
            CustomerGuid = new Guid("68700794-F63B-46CA-B10B-D4B54C6081F8"),
            BranchGuid = new Guid("43620794-F63B-46CA-B10B-D4B54C6081B4"),
        });
```
