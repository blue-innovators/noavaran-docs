# کامندهای سیستم پرداخت

کامند های مربوط به پرداخت همگی در فضای نامی `AbrAbi.Common.Messaging.Messages.Payment` قابل دسترس میباشند:

## ایجاد کیف پول (WalletCreateCommandV1)

این کامند برای ایجاد یک کیف پول استفاده می شود . که در زمان ایجاد شعبه و کاربر جدید به صورت خودکار بعد از ثبت موجودیت فراخوانی میشود  
همچنین این کامند دارای ریسپانس نمی باشد و ورودی آن به شرح زیر است:

```cs
public class WalletCreateCommandV1 : ICommandMessage
{
    public Guid ReferenceId { get; set; }
    public string ReferenceType { get; set; }
    public string ReferenceWalletType { get; set; }
    public string ReferenceTitle { get; set; }
    public string ReferenceAccountOwnerName { get; set; }
}
```

ReferenceId: شناسه موجودیت  
ReferenceType: نوع موجودیت  
ReferenceTitle: عنوان موجودیت  
ReferenceAccountOwnerName: عنوان نمایشی صاحب حساب در سیستم رایان پی
ReferenceWalletType: نوع کیف پول که این نوع ها سمت رایان پی تعریف شده است و در حال حاضر به شرح زیر است:  

```cs
namespace AbrAbi.Common.Constants
{
    public class ReferenceWalletType
    {
        public const string System = "System";
        public const string BranchMain = "BranchMain";
        public const string BranchReservation = "BranchReservation";
        public const string CustomerMain = "CustomerMain";
    }
}
```

نمونه درخواست:

```cs
await _commonMessagingClient.Send(new WalletCreateCommandV1
{
    ReferenceId = 1004,
    ReferenceType = "Branch",
    ReferenceWalletType = ReferenceWalletType.BranchMain,
    ReferenceTitle = $"شعبه-اصلی-لئون-الهیه",
    ReferenceAccountOwnerName = $"شعبه-لئون-الهیه"
});
```

## موجودی کیف پول (WalletGetBalanceCommandV1)

این کامند برای بدست آوردن موجودی کیف پول های یک موجودیت (کاربر،شعبه، ...) استفاده می شود و به شرح زیر می باشد:

ورودی:

```cs
public class WalletGetBalanceCommandV1 : IRequestMessage<WalletGetBalanceCommandResponseV1>
{
    public Guid ReferenceId { get; set; }
    public string ReferenceType { get; set; }
}
```

ReferenceId: شناسه یکتای موجودیت  
ReferenceType: نوع موجودیت

خروجی:

```cs
public class WalletGetBalanceCommandResponseV1 : IResponseMessage
{
    public IEnumerable<WalletBalance> Balances { get; set; }
    public string Error { get; set; }
}
```

Error: اگر خطایی وجود داشته باشد در این فیلد برگرداننده می شود  
Balances: لیستی از WalletBalance ها را برمیگرداند که در آن موجودی تمام کیف پول های آن کاربر مشخص است که به شرح زیر است:

```cs
public class WalletBalance
{
    public string ReferenceWalletType { get; set; }
    public long Balance { get; set; }
}
```

ReferenceWalletType: نوع کیف پول  
Balance: موجودی

نمونه درخواست:

```cs
var response = await _messagingClient.Request<WalletGetBalanceCommandV1, WalletGetBalanceCommandResponseV1>(
    new WalletGetBalanceCommandV1
    {
        ReferenceId = new Guid("68700794-F63B-46CA-B10B-D4B54C6081F8"),
        ReferenceType = "Branch"
    });
```

## شارژ کیف پول کاربر (WalletUserChargeCommandV1)

این کامند برای شارژ کیف پول یک کاربر استفاده می شود و به شرح زیر می باشد:

ورودی:

```cs
public class WalletUserChargeCommandV1 : IRequestMessage<WalletUserChargeCommandResponseV1>
{
    public Guid? Guid { get; set; }
    public long UserId { get; set; }
    public Guid UserGuid { get; set; }
    public string UserName { get; set; }
    public long Amount { get; set; }
    public string CallbackUrl { get; set; }
}
```

Guid: شناسه یکتا که در صورت پر نکردن توسط خود سرور ساخته می شود  
UserId: شناسه عددی کاربر  
UserGuid: شناسه یکتای کاربر  
UserName: نام کاربری  
Amount: مبلغ  
CallbackUrl: آدرسی که بعد از پرداخت به آن ریدایرکت می شود

خروجی:

```cs
public class WalletUserChargeCommandResponseV1 : IResponseMessage
{
    public long OnlineDepositId { get; set; }
    public Guid OnlineDepositGuid { get; set; }
    public string RedirectUrl { get; set; }
    public string Error { get; set; }
}
```

OnlineDepositId: شناسه عددی واریز آنلاین  
OnlineDepositGuid: شناسه یکتای واریز آنلاین  
RedirectUrl: آدرسی که برای پرداخت آنلاین باید به آن ریدایرکت کرد  
Error: اگر خطایی وجود داشته باشد در این فیلد برگرداننده می شود

نمونه درخواست:

```cs
var response = await _messagingClient.Request<WalletUserChargeCommandV1, WalletUserChargeCommandResponseV1>(
    new WalletUserChargeCommandV1
    {
        UserId = 15,
        UserGuid = new Guid("54700794-F63B-46CA-B10B-D4B54C6081F2"),
        UserName = "mohsen",
        Guid = new Guid("15700794-F63B-46CA-B10B-D4B54C6025c3"),
        Amount = 50000,
        CallbackUrl = "marketplace.ir/wallet"
    });
```

## تسویه پرداخت (PaymentSettleCommandV1)

این کامند برای تسویه یک پرداخت که شامل پرداخت آنلاین بوده استفاده می شود و پس از رفتن به درگاه بانکی و انجام شدن عملیات بانکی باید صدا زده شود که به شرح زیر می باشد:

ورودی:

```cs
public class PaymentSettleCommandV1 : IRequestMessage<ResponseMessageBase>
{
    public Guid Guid { get; set; }
    public string BankAuthority { get; set; }
    public string BankStatus { get; set; }
}
```

Guid: شناسه پرداخت  
BankAuthority: این فیل یک شناسه یکتا می باشد که بعد از پرداخت در درگاه بانکی بانک بر می گرداند  
BankStatus: این فیلد وضعیت پرداخت بانکی می باشد که از بانک بر میگردد

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
var response = await _messagingClient.Request<PaymentSettleCommandV1, ResponseMessageBase>(
    new PaymentSettleCommandV1
    {
        Guid = new Guid("54700794-F63B-46CA-B10B-D4B54C6081F2"),
        BankAuthority = "1f41a9c5-57b3-4712-89c4-f90eae422f19",
        BankStatus = "OK"
    });
```

## لغو دستور پرداخت (PaymentRejectCommandV1)

این کامند برای لغو دستور پرداخت استفاده می شود و تمام اجزای یک دستور پرداخت (تراکنش ها ، واریز آنلاین ها ) و خود دستور پرداخت را منقضی می نماید که به شرح زیر می باشد:

ورودی:

```cs
public class PaymentRejectCommandV1 : ICommandMessage
{
    public Guid ReferenceId { get; set; }
    public string ReferenceType { get; set; }
}
```

ReferenceId: شناسه موجودیت  
ReferenceType: نوع موجودیت

این کامند خروجی ندارد

نمونه درخواست:

```cs
await _messagingClient.Send(new PaymentRejectCommandV1
{
    ReferenceId = new Guid("1f41a9c5-57b3-4712-89c4-f90eae422f19"),
    ReferenceType = "Branch"
});
```

## لغو واریز آنلاین (OnlineDepositRejectCommandV1)

این کامند برای منقضی کردن یک واریز آنلاین به کار می رود و به شرح زیر می باشد:

ورودی:

```cs
public class OnlineDepositRejectCommandV1 : ICommandMessage
{
    public Guid Guid { get; set; }
}
```

Guid: شناسه یکتای واریز آنلاین

این کامند خروجی ندارد

نمونه درخواست:

```cs
await _messagingClient.Send(new OnlineDepositRejectCommandV1
{
    Guid = new Guid("1f41a9c5-57b3-4712-89c4-f90eae422f19"),
});
```



## ایجاد دستور پرداخت (منقضی شده) (PaymentCreateCommandV1)

این کامند برای ایجاد یک دستور پرداخت استفاده می شود و به شرح زیر است:

ورودی:

```cs
public class PaymentCreateCommandV1 : IRequestMessage<PaymentCreateCommandResponseV1>
{
    public Guid? Guid { get; set; }
    public long UserId { get; set; }
    public string UserName { get; set; }
    public Guid ReferenceId { get; set; }
    public string ReferenceType { get; set; }
    public long Amount { get; set; }
    public bool UseWalletCredit { get; set; }
    public WalletReference SourceWallet { get; set; }
    public IEnumerable<TransactionWalletAmount> DestinationWallets { get; set; }
    public IEnumerable<PaymentMetaDataV1> MetaData { get; set; }
    public string CallbackUrl { get; set; }
}
```

Guid: شناسه یکتای پرداخت که در صورتی که ارسال نشود توسط سرور ایجاد می شود  
UserId: شناسه عددی کاربر  
UserName: نام کاربری  
ReferenceId: شناسه موجودیت  
ReferenceType: نوع موجودیت  
Amount: مبلغ  
UseWalletCredit: این پارامتر مشخص می کند که آیا از موجودی حساب کاربر استفاده بشود یا نه  
CallbackUrl: آدرسی که بعد از پرداخت به آن ریدایرکت می شود  
MetaData: متا دیتا که میتواند با توجه به نیاز لیستی را در آن ذخیره کرد  
SourceWallet: کیف پول مبدا که از نوع زیر می باشد:

```cs
public class WalletReference
{
    public Guid ReferenceId { get; set; }
    public string ReferenceType { get; set; }
    public string ReferenceWalletType { get; set; }
}
```

DestinationWallets: لیست کیف پول های مقصد که از نوع زیر می باشد:

```cs
public class TransactionWalletAmount : WalletReference
{
    public long Amount { get; set; }
}
```

خروجی:

```cs
public class PaymentCreateCommandResponseV1 : IResponseMessage
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
var response = await _messagingClient.Request<PaymentCreateCommandV1, PaymentCreateCommandResponseV1>(
    new PaymentCreateCommandV1
    {
        Guid = new Guid("54700794-F63B-46CA-B10B-D4B54C6081F2"),
        UserId = 11,
        UserName = "ali",
        ReferenceId = reserve.Guid,
        ReferenceType = nameof(Reservation),
        Amount = 100000,
        UseWalletCredit = true,
        SourceWallet = new WalletReference
        {
            ReferenceId = customer.Id,
            ReferenceType = nameof(User),
            ReferenceWalletType = ReferenceWalletType.CustomerMain
        },
        DestinationWallets = new TransactionWalletAmount[]
        {
            new TransactionWalletAmount
            {
                ReferenceId = ReservationAcceptPaymentCommand.Branch.Guid,
                ReferenceType = nameof(Branch),
                ReferenceWalletType = ReferenceWalletType.BranchReservation,
                Amount = 100000
            }
        },
        CallbackUrl = "https://restora.keepapp.ir/api/payment/callback",
        Metadata = new PaymentMetaDataV1[]
        {
            new PaymentMetaDataV1 { Name = "meta name", Value = "meta value" }
        }
    });
```
