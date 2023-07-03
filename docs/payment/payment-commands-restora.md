# کامند های پرداخت سیستم رستورا

## پرداخت رزرو - نسخه یک (ReservationPaymentCommandV1)

این کامند برای پیش پرداخت یک رزرو استفاده می شود و مبلغ را از حساب مشتری به حساب رزرو شعبه انتقال می دهد که به شرح زیر است:

ورودی:

```cs
public class ReservationPaymentCommandV1 
    : IRequestMessage<PaymentCreateCommandResponseV1>
{
    public string Name { get; set; }
    public Guid UserGuid { get; set; }
    public string UserName { get; set; }
    public Guid CustomerUserGuid { get; set; }
    public Guid BusinessGuid { get; set; }
    public Guid BranchGuid { get; set; }
    public Guid ReservationGuid { get; set; }
    public long Amount { get; set; }
    public bool UseWalletCredit { get; set; }
    public IEnumerable<PaymentMetaDataV1> MetaData { get; set; }
    public string CallbackUrl { get; set; }
}
```

Name: عنوان پرداخت  
UserGuid: شناسه یکتای کاربر ثبت کننده درخواست  
UserName: نام کاربری ثبت کننده درخواست  
CustomerUserGuid: شناسه یکتای مشتری  
BusinessGuid: شناسه یکتای کسب و کار  
BranchGuid: شناسه یکتای شعبه  
ReservationGuid: شناسه یکتای رزرو  
Amount: مبلغ  
UseWalletCredit: این پارامتر مشخص می کند که آیا از موجودی حساب کاربر استفاده بشود یا نه  
MetaData: متا دیتا که میتواند با توجه به نیاز لیستی را در آن ذخیره کرد  
CallbackUrl: آدرسی که بعد از پرداخت به آن ریدایرکت می شود

خروجی:

```cs
public class PaymentCreateCommandResponseV1
{
    public long PaymentId { get; set; }
    public Guid PaymentGuid { get; set; }
    public string RedirectUrl { get; set; }
    public string Error { get; set; }
}
```

PaymentId: شناسه عددی پرداخت  
PaymentGuid: شناسه یکتای پرداخت  
RedirectUrl: آدرسی که برای پرداخت آنلاین باید به آن ریدایرکت کرد

نمونه درخواست:

```cs
var response = await _messagingClient.Request<ReservationPaymentCommandV1, PaymentCreateCommandResponseV1>(
    new ReservationPaymentCommandV1
    {
        Name = "پیش پرداخت رزرو ایمان",
        UserGuid = new Guid("54700794-F63B-46CA-B10B-D4B54C6081F2"),
        UserName = "iman",
        CustomerUserGuid = new Guid("36700794-F63B-46CA-B10B-D4B54C6082G6"),
        BusinessGuid = new Guid("12700794-F63B-46CA-B10B-D4B54C6081B5"),
        BranchGuid = new Guid("74700794-F63B-46CA-B10B-D4B54C6083A6"),
        ReservationGuid = new Guid("85400794-F63B-46CA-B10B-D4B54C6084C1"),
        Amount = 100000,
        UseWalletCredit = true,
        CallbackUrl = "https://restora.keepapp.ir/api/payment/callback",
        Metadata = new PaymentMetaDataV1[]
        {
            new PaymentMetaDataV1 { Name = "meta name", Value = "meta value" }
        }
    });
```

## پذیرش رزرو - نسخه یک (ReservationAcceptPaymentCommandV1)

این کامند بعد از پذیرش رزرو صدا زده می شود و میزان سود ها را حساب کرده و به حساب های مخصوص آن ها و سهم شعبه را به حساب اصلی شعبه انتقال می دهد که به شرح زیر است:

ورودی:

```cs
public class ReservationAcceptPaymentCommandV1 
    : IRequestMessage<ResponseMessageBase>
{
    public string Name { get; set; }
    public Guid UserGuid { get; set; }
    public string UserName { get; set; }
    public Guid CustomerGuid { get; set; }
    public Guid BusinessGuid { get; set; }
    public Guid BranchGuid { get; set; }
    public Guid ReservationGuid { get; set; }
    public long PrepaymentAmount { get; set; }
    public IEnumerable<PaymentMetaDataV1> MetaData { get; set; }
}
```

Name: عنوان پرداخت  
UserGuid: شناسه یکتای کاربر ثبت کننده درخواست  
UserName: نام کاربری ثبت کننده درخواست  
BusinessGuid: شناسه یکتای کسب و کار  
BranchGuid: شناسه یکتای شعبه  
CustomerGuid: شناسه یکتای مشتری  
ReservationGuid: شناسه یکتای رزرو  
PrepaymentAmount: مبلغ پیش پرداخت

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
var response = await _messagingClient.Request<ReservationAcceptPaymentCommandV1, ResponseMessageBase>(
    new ReservationAcceptPaymentCommandV1
    {
        Name = "تایید رزرو ایمان",
        UserGuid = new Guid("54700794-F63B-46CA-B10B-D4B54C6081F2"),
        UserName = "iman",
        BusinessGuid = new Guid("12700794-F63B-46CA-B10B-D4B54C6081B5"),
        BranchGuid = new Guid("74700794-F63B-46CA-B10B-D4B54C6083A6"),
        CustomerGuid = new Guid("36700794-F63B-46CA-B10B-D4B54C6082G6"),
        ReservationGuid = new Guid("85400794-F63B-46CA-B10B-D4B54C6084C1"),
        PrepaymentAmount = 120000,
    });
```

## لغو رزرو - نسخه یک (ReservationCancelPaymentCommandV1)

این کامند برای کنسل کردن رزرو استفاده می شود
این دستور جریمه کنسلی و سودهای مربوطه را محاسبه کرده و از حساب رزرو شعبه به حساب های مربوطه انتقال می دهد و مابقی پول مشتری را به حساب مشتری باز می گرداند که به شرح زیر می باشد:

ورودی:

```cs
public class ReservationCancelPaymentCommandV1 
    : IRequestMessage<ResponseMessageBase>
{
    public string Name { get; set; }
    public Guid UserGuid { get; set; }
    public string UserName { get; set; }
    public Guid CustomerGuid { get; set; }
    public Guid BusinessGuid { get; set; }
    public Guid BranchGuid { get; set; }
    public Guid ReservationGuid { get; set; }
    public long PrepaymentAmount { get; set; }
    public long PenaltyAmount { get; set; }
    public IEnumerable<PaymentMetaDataV1> MetaData { get; set; }
}
```

Name: عنوان پرداخت  
UserGuid: شناسه یکتای کاربر ثبت کننده درخواست  
UserName: نام کاربری ثبت کننده درخواست  
BusinessGuid: شناسه یکتای کسب و کار  
BranchGuid: شناسه یکتای شعبه  
CustomerGuid: شناسه یکتای مشتری  
ReservationGuid: شناسه یکتای رزرو  
PrepaymentAmount: مبلغ پیش پرداخت  
PenaltyAmount: مبلغ جریمه

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
var response = await _messagingClient.Request<ReservationCancelPaymentCommandV1, ResponseMessageBase>(
    new ReservationCancelPaymentCommandV1
    {
        Name = "لغو رزرو ایمان",
        UserGuid = new Guid("54700794-F63B-46CA-B10B-D4B54C6081F2"),
        UserName = "ali",
        BusinessGuid = new Guid("12700794-F63B-46CA-B10B-D4B54C6081B5"),
        BranchGuid = new Guid("74700794-F63B-46CA-B10B-D4B54C6083A6"),
        CustomerGuid = new Guid("36700794-F63B-46CA-B10B-D4B54C6082G6"),
        ReservationGuid = new Guid("85400794-F63B-46CA-B10B-D4B54C6084C1"),
        PrepaymentAmount = 120000,
        PenaltyAmount = 2400,
    });
```
