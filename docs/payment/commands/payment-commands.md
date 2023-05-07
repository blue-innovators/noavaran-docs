# command های سیستم پرداخت

command ها به شرح زیر می باشند:

## خرید غذا (CreateOrderPaymentCommand)

**این command برای خرید یک سفارش به کار می رود و به شرح زیر می باشد:**  
این نوع تراکنش هر سه حالت پرداخت(نقدی، آنلاین، کیف پول) و همین طور ترکیبی آن ها را ساپورت میکند

**اگر مبلغ نقدی داشته باشد تراکنش های آن به شرح زیر اتفاق می افتد:**  
ابتدا یک تراکنش از حساب ورودی نقدی رایان پی به کیف پول کاربر زده می شود و کیف پول کاربر به میزان مبلغ نقدی شارژ می شود
سپس یک تراکنش دوتایی زده می شود که در آن یک تراکنش به مبلغ کل خرید از کیف پول کاربر به حساب اصلی شعبه زده می شود و یک تراکنش دیگر به مبلغ خرید نقدی از حساب اصلی شعبه به حساب خروجی نقدی زده می شود

** ورودی و خروجی این command به شرح زیر می باشد **:

ورودی:

```cs
public class CreateOrderPaymentCommand : IRequestMessage<CreatePaymentCommandResponse>
{
    public long UserId { get; set; }
    public string UserName { get; set; }
    public Guid OrderGuid { get; set; }
    public Guid CustomerUserGuid { get; set; }
    public Guid BranchGuid { get; set; }
    public long Amount { get; set; }
    public long CashAmount { get; set; }
    public bool UseWalletCredit { get; set; }
    public IEnumerable<MetaData> MetaData { get; set; }
    public string CallbackUrl { get; set; }
}
```

UserId: شناسه کاربر  
UserName: نام کاربر  
OrderGuid: شناسه یکتای سفارش  
CustomerUserGuid: شناسه یکتای مشتری  
BranchGuid: شناسه یکتای شعبه  
Amount: مبلغ کل خرید  
CashAmount: مبلغ نقدی  
UseWalletCredit: این پارامتر مشخص می کند که آیا از موجودی حساب کاربر استفاده بشود یا نه
MetaData: متا دیتا که میتواند با توجه به نیاز لیستی را در آ ذخیره کرد  
CallbackUrl: آدرسی که در صورت داشتن درگاه پرداخت به آن مراجعه می شود

خروجی:

```cs
public class CreatePaymentCommandResponse : PaymentCommandResponse
{
    public long PaymentId { get; set; }
    public Guid PaymentGuid { get; set; }
    public string RedirectUrl { get; set; }
    public string Error { get; set; }
}
```

PaymentId: شناسه یکتای عددی پرداخت  
PaymentGuid: شناسه یکتای پرداخت  
RedirectUrl: اگر سفارش منجر به پرداخت آنلاین شود درخواست دهنده باید برای پرداخت آنلاین به این آدرس مراجعه کند تا کاربر مراحل پرداخت را انجام دهد
Error: اگر خطایی وجود داشته باشد در این فیلد برگرداننده می شود

نمونه درخواست:

```cs
var response = await _messagingClient.Request<CreateOrderPaymentCommand, CreatePaymentCommandResponse>(
    new CreateOrderPaymentCommand
    {
        UserId = 10,
        UserName = "iman",
        OrderGuid = new Guid("53E8708C-6B49-4F83-B727-09BBC4CCC4a4"),
        BranchGuid = new Guid("E98B3227-C3DC-43D8-B879-D78F7C111DAF"),
        CustomerUserGuid = new Guid("60E8708C-6B49-4F83-B727-09BBC4CCC4C7"),
        Amount = 100000,
        CashAmount = 80000,
        UseWalletCredit = true,
        CallbackUrl = "leon.ir/order",
            MetaData = new List<MetaData>()
            {
                new MetaData { Name="label", Value="burger"},
                new MetaData { Name="table", Value="m2"}
            }
    });
```

## ایجاد کیف پول (CreateWalletCommand)

این command برای ایجاد یک کیف پول استفاده می شود . که در زمان ایجاد شعبه و کاربر جدید به صورت خودکار بعد از ثبت موجودیت فراخوانی میشود  
همچنین این command دارای ریسپانس نمی باشد و ورودی آن به شرح زیر است:

```cs
public class CreateWalletCommand : ICommandMessage
{
    public Guid ReferenceId { get; set; }
    public string ReferenceType { get; set; }
    public string ReferenceWalletType { get; set; }
    public string ReferenceTitle { get; set; }
}
```

ReferenceId: شناسه موجودیت  
ReferenceType: نوع موجودیت  
ReferenceTitle: عنوان موجودیت  
ReferenceWalletType: نوع کیف پول که این نوع ها سمت رایان پی تعریف شده است و در حال حاضر به شرح زیر است:

```cs
public class ReferenceWalletType
{
	public const string BranchMain = "BranchMain";
	public const string BranchReservation = "BranchReservation";
	public const string CustomerMain = "CustomerMain";
}
```

نمونه درخواست:

```cs
await _commonMessagingClient.Send(new Messaging.Messages.Payment.CreateWalletCommand
{
    ReferenceId = 1004,
    ReferenceType = "Branch",
    ReferenceWalletType = ReferenceWalletType.BranchMain,
    ReferenceTitle = "لئون شعبه الهیه"
});
```

## موجودی کیف پول (GetWalletBalanceCommand)

این command برای به دست آوردن موجودی کیف پول های یک شخص (کاربر،شعبه ...) استفاده می شود و به شرح زیر می باشد:

ورودی:

```cs
public class GetWalletBalanceCommand : IRequestMessage<GetWalletBalanceCommandResponse>
{
    public Guid ReferenceId { get; set; }
    public string ReferenceType { get; set; }
}
```

ReferenceId: شناسه یکتای موجودیت  
ReferenceType: نوع موجودیت

خروجی:

```cs
public class GetWalletBalanceCommandResponse : IResponseMessage
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
var response = await _messagingClient.Request<GetWalletBalanceCommand, GetWalletBalanceCommandResponse>(
    new GetWalletBalanceCommand
        {
            ReferenceId = new Guid("68700794-F63B-46CA-B10B-D4B54C6081F8"),
            ReferenceType = "Branch"
        });
```

## شارژ کیف پول کاربر (UserWalletChargeCommand)

این command برای شارژ کیف پول یک کاربر استفاده می شود و به شرح زیر می باشد:

ورودی:

```cs
public class UserWalletChargeCommand : IRequestMessage<UserWalletChargeCommandResponse>
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
public class UserWalletChargeCommandResponse : IResponseMessage
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
var response = await _messagingClient.Request<UserWalletChargeCommand, UserWalletChargeCommandResponse>(
    new UserWalletChargeCommand
        {
            UserId=15,
            UserGuid=new Guid("54700794-F63B-46CA-B10B-D4B54C6081F2"),
            UserName="mohsen",
            Guid=new Guid("15700794-F63B-46CA-B10B-D4B54C6025c3"),
            Amount=50000,
            CallbackUrl="marketplace.ir/wallet"
        });
```

## ایجاد دستور پرداخت (CreatePaymentCommand)

این command برای ایجاد یک دستور پرداخت استفاده می شود و به شرح زیر است:

ورودی:

```cs
public class CreatePaymentCommand : IRequestMessage<CreatePaymentCommandResponse>
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
    public IEnumerable<MetaData> MetaData { get; set; }
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
public class CreatePaymentCommandResponse : PaymentCommandResponse
{
    public long PaymentId { get; set; }
    public Guid PaymentGuid { get; set; }
    public string RedirectUrl { get; set; }
}
```

PaymentId: شناسه عددی پرداخت  
PaymentGuid: شناسه یکتای پرداخت  
RedirectUrl: آدرسی که برای پرداخت آنلاین باید به آن ریدایرکت کرد

نمونه درخواست:

```cs
var response = await _messagingClient.Request<CreatePaymentCommand, CreatePaymentCommandResponse>(
    new CreatePaymentCommand
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
                    ReferenceType nameof(User),
                    ReferenceWalletType = ReferenceWalletType.CustomerMain
                },
            DestinationWallets = new Common.Messaging.Messages.Payment.PaymentDtos.TransactionWalletAmount[]
                {
                    new Common.Messaging.Messages.Payment.PaymentDtos.TransactionWalletAmount
                    {
                        ReferenceId= ReservationAcceptPaymentCommand.Branch.Guid,
                        ReferenceType = nameof(Branch),
                        ReferenceWalletType = ReferenceWalletType.BranchReservation,
                        Amount = 100000
                    }
                },
            CallbackUrl = "https://restora.keepapp.ir/api/payment/callback",
            Metadata = new MetaData[]
                {
                    new MetaData{Name="meta name",Value="meta value"}
                }
        });
```

## پرداخت رزرو (ReservationPaymentCommand)

این command برای پیش پرداخت یک رزرو استفاده می شود و مبلغ را از حساب مشتری به حساب رزرو شعبه انتقال می دهد که به شرح زیر است:

ورودی:

```cs
public class ReservationPaymentCommand : IRequestMessage<CreatePaymentCommandResponse>
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
    public IEnumerable<MetaData> MetaData { get; set; }
    public string CallbackUrl { get; set; }
}
```

Name: نام  
UserGuid: شناسه یکتای کاربر  
UserName: نام کاربری  
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
public class CreatePaymentCommandResponse : PaymentCommandResponse
{
    public long PaymentId { get; set; }
    public Guid PaymentGuid { get; set; }
    public string RedirectUrl { get; set; }
}
```

PaymentId: شناسه عددی پرداخت  
PaymentGuid: شناسه یکتای پرداخت  
RedirectUrl: آدرسی که برای پرداخت آنلاین باید به آن ریدایرکت کرد

نمونه درخواست:

```cs
var response = await _messagingClient.Request<ReservationPaymentCommand, CreatePaymentCommandResponse>(
    new ReservationPaymentCommand
        {
            Name = "reserve sample"
            UserGuid = new Guid("54700794-F63B-46CA-B10B-D4B54C6081F2"),
            UserName = "ali",
            CustomerUserGuid = new Guid("36700794-F63B-46CA-B10B-D4B54C6082G6"),
            BusinessGuid = new Guid("12700794-F63B-46CA-B10B-D4B54C6081B5"),
            BranchGuid = new Guid("74700794-F63B-46CA-B10B-D4B54C6083A6"),
            ReservationGuid = new Guid("85400794-F63B-46CA-B10B-D4B54C6084C1"),
            Amount = 100000,
            UseWalletCredit = true,
            CallbackUrl = "https://restora.keepapp.ir/api/payment/callback",
            Metadata = new MetaData[]
                {
                    new MetaData{Name="meta name",Value="meta value"}
                }
        });
```

## پذیرش رزرو (ReservationAcceptPaymentCommand)

این command بعد از پذیرش رزرو صدا زده می شود و میزان سود ها را حساب کرده و به حساب های مخصوص آن ها و سهم شعبه را به حساب اصلی شعبه انتقال می دهد که به شرح زیر است:

ورودی:

```cs
public class ReservationAcceptPaymentCommand : IRequestMessage<PaymentCommandResponse>
{
    public Guid UserGuid { get; set; }
    public string UserName { get; set; }
    public Guid BranchGuid { get; set; }
    public Guid CustomerGuid { get; set; }
    public Guid ReservationGuid { get; set; }
    public long PrepaymentAmount { get; set; }
}
```

UserGuid: شناسه یکتای کاربر  
UserName: نام کاربری  
BranchGuid: شناسه یکتای شعبه  
CustomerGuid: شناسه یکتای مشتری  
ReservationGuid: شناسه یکتای رزرو  
PrepaymentAmount: مبلغ پیش پرداخت

خروجی:

```cs
public class PaymentCommandResponse : IResponseMessage
{
    public string Error { get; set; }
}
```

Error: اگر خطایی وجود داشته باشد در این فیلد برگرداننده می شود  
در صورتی که خطایی بر نگردد یعنی عملیات با موفقیت انجام شده

نمونه درخواست:

```cs
var response = await _messagingClient.Request<ReservationAcceptPaymentCommand, PaymentCommandResponse>(
    new ReservationAcceptPaymentCommand
        {
            UserGuid = new Guid("54700794-F63B-46CA-B10B-D4B54C6081F2"),
            UserName = "ali",
            BranchGuid = new Guid("74700794-F63B-46CA-B10B-D4B54C6083A6"),
            CustomerGuid = new Guid("36700794-F63B-46CA-B10B-D4B54C6082G6"),
            ReservationGuid = new Guid("85400794-F63B-46CA-B10B-D4B54C6084C1"),
            PrepaymentAmount = 120000,
        });
```

## لغو رزرو (ReservationCancelPaymentCommand)

این command برای کنسل کردن رزرو استفاده می شود
این دستور جریمه کنسلی و سودهای مربوطه را محاسبه کرده و از حساب رزرو شعبه به حساب های مربوطه انتقال می دهد و مابقی پول مشتری را به حساب مشتری باز می گرداند که به شرح زیر می باشد:

ورودی:

```cs
public class ReservationCancelPaymentCommand : IRequestMessage<PaymentCommandResponse>
{
    public Guid UserGuid { get; set; }
    public string UserName { get; set; }
    public Guid BranchGuid { get; set; }
    public Guid CustomerGuid { get; set; }
    public Guid ReservationGuid { get; set; }
    public long PrepaymentAmount { get; set; }
    public long PenaltyAmount { get; set; }
}
```

UserGuid: شناسه یکتای کاربر  
UserName: نام کاربری  
BranchGuid: شناسه یکتای شعبه  
CustomerGuid: شناسه یکتای مشتری  
ReservationGuid: شناسه یکتای رزرو  
PrepaymentAmount: مبلغ پیش پرداخت  
PenaltyAmount: مبلغ جریمه

خروجی:

```cs
public class PaymentCommandResponse : IResponseMessage
{
    public string Error { get; set; }
}
```

Error: اگر خطایی وجود داشته باشد در این فیلد برگرداننده می شود  
در صورتی که خطایی بر نگردد یعنی عملیات با موفقیت انجام شده

نمونه درخواست:

```cs
var response = await _messagingClient.Request<ReservationAcceptPaymentCommand, PaymentCommandResponse>(
    new ReservationAcceptPaymentCommand
        {
            UserGuid = new Guid("54700794-F63B-46CA-B10B-D4B54C6081F2"),
            UserName = "ali",
            BranchGuid = new Guid("74700794-F63B-46CA-B10B-D4B54C6083A6"),
            CustomerGuid = new Guid("36700794-F63B-46CA-B10B-D4B54C6082G6"),
            ReservationGuid = new Guid("85400794-F63B-46CA-B10B-D4B54C6084C1"),
            PrepaymentAmount = 120000,
            PenaltyAmount = 2400
        });
```

## تسویه پرداخت (SettlePaymentCommand)

این command برای تسویه یک پرداخت که شامل پرداخت آنلاین بوده استفاده می شود و پس از رفتن به درگاه بانکی و انجام شدن عملیات بانکی باید صدا زده شود که به شرح زیر می باشد:

ورودی:

```cs
public class SettlePaymentCommand : IRequestMessage<PaymentCommandResponse>
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
public class PaymentCommandResponse : IResponseMessage
{
    public string Error { get; set; }
}
```

Error: اگر خطایی وجود داشته باشد در این فیلد برگرداننده می شود  
در صورتی که خطایی بر نگردد یعنی عملیات با موفقیت انجام شده

نمونه درخواست:

```cs
var response = await _messagingClient.Request<SettlePaymentCommand, PaymentCommandResponse>(
    new ReservationAcceptPaymentCommand
        {
            Guid = new Guid("54700794-F63B-46CA-B10B-D4B54C6081F2"),
            BankAuthority = "1f41a9c5-57b3-4712-89c4-f90eae422f19",
            BankStatus = "OK"
        });
```

## لغو دستور پرداخت (RejectPaymentCommand)

این command برای لغو دستور پرداخت استفاده می شود و تمام اجزای یک دستور پرداخت (تراکنش ها ، واریز آنلاین ها ) و خود دستور پرداخت را منقضی می نماید که به شرح زیر می باشد:

ورودی:

```cs
public class RejectPaymentCommand : ICommandMessage
{
    public Guid ReferenceId { get; set; }
    public string ReferenceType { get; set; }
}
```

ReferenceId: شناسه موجودیت  
ReferenceType: نوع موجودیت

این command خروجی ندارد

نمونه درخواست:

```cs
await _messagingClient.Send(
    new RejectPaymentCommand
        {
         ReferenceId = new Guid("1f41a9c5-57b3-4712-89c4-f90eae422f19"),
         ReferenceType = "Branch"
        });
```

## لغو واریز آنلاین (RejectOnlineDepositCommand)

این command برای منقضی کردن یک واریز آنلاین به کار می رود و به شرح زیر می باشد:

ورودی:

```cs
public class RejectOnlineDepositCommand : ICommandMessage
{
    public Guid Guid { get; set; }
}
```

Guid: شناسه یکتای واریز آنلاین

این command خروجی ندارد

نمونه درخواست:

```cs
await _messagingClient.Send(
    new RejectOnlineDepositCommand
        {
         Guid = new Guid("1f41a9c5-57b3-4712-89c4-f90eae422f19"),
        });
```

