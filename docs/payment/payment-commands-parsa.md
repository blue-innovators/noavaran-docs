# کامندهای پرداخت سیستم پرسا

## خرید غذا - نسخه یک (OrderPaymentCommandV1)

**این کامند برای خرید یک سفارش منو در سیستم پرسا به کار می رود و به شرح زیر می باشد:**  
این نوع تراکنش هر سه حالت پرداخت(نقدی، آنلاین، کیف پول) و همین طور ترکیبی آن ها را ساپورت میکند

**اگر مبلغ نقدی داشته باشد تراکنش های آن به شرح زیر اتفاق می افتد:**  
ابتدا یک تراکنش از حساب ورودی نقدی رایان پی به کیف پول کاربر زده می شود و کیف پول کاربر به میزان مبلغ نقدی شارژ می شود
سپس یک تراکنش دوتایی زده می شود که در آن یک تراکنش به مبلغ کل خرید از کیف پول کاربر به حساب اصلی شعبه زده می شود و یک تراکنش دیگر به مبلغ خرید نقدی از حساب اصلی شعبه به حساب خروجی نقدی زده می شود

** ورودی و خروجی این کامند به شرح زیر می باشد **:

ورودی:

```cs
public class OrderPaymentCommandV1 
    : IRequestMessage<PaymentCreateCommandResponseV1>
{
    public string Name { get; set; }
    public Guid UserGuid { get; set; }
    public string UserName { get; set; }
    public Guid CustomerUserGuid { get; set; }
    public Guid BusinessGuid { get; set; }
    public Guid BranchGuid { get; set; }
    public Guid OrderGuid { get; set; }
    public long Amount { get; set; }
    public long CashAmount { get; set; }
    public bool UseWalletCredit { get; set; }
    public IEnumerable<PaymentMetaDataV1> MetaData { get; set; }
    public string CallbackUrl { get; set; }
}
```

Name: عنوان پرداخت  
UserGuid: شناسه یکتای کاربر ثبت کننده درخواست  
UserName: نام کاربری ثبت کننده درخواست  
OrderGuid: شناسه یکتای سفارش  
CustomerUserGuid: شناسه یکتای مشتری  
BusinessGuid: شناسه یکتای کسب و کار  
BranchGuid: شناسه یکتای شعبه  
OrderGuid: شناسه یکتای سفارش  
Amount: مبلغ کل خرید  
CashAmount: مبلغ نقدی  
UseWalletCredit: این پارامتر مشخص می کند که آیا از موجودی حساب کاربر استفاده بشود یا نه  
MetaData: متا دیتا که میتواند با توجه به نیاز لیستی را در آ ذخیره کرد  
CallbackUrl: آدرسی که در صورت داشتن درگاه پرداخت به آن مراجعه می شود  

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

PaymentId: شناسه یکتای عددی پرداخت  
PaymentGuid: شناسه یکتای پرداخت  
RedirectUrl: اگر سفارش منجر به پرداخت آنلاین شود درخواست دهنده باید برای پرداخت آنلاین به این آدرس مراجعه کند تا کاربر مراحل پرداخت را انجام دهد
Error: اگر خطایی وجود داشته باشد در این فیلد برگرداننده می شود

نمونه درخواست:

```cs
var response = await _messagingClient.Request<OrderPaymentCommandV1, PaymentCreateCommandResponseV1>(
    new OrderPaymentCommandV1
    {
        Name = "سفارش خرید غذای ایمان",
        UserGuid = new Guid("60E8708C-6B49-4F83-B727-09BBC4CCC4C7"),
        UserName = "iman",
        OrderGuid = new Guid("53E8708C-6B49-4F83-B727-09BBC4CCC4a4"),
        BusinessGuid = new Guid("E98B3227-C3DC-43D8-B879-D78F7C111DAF"),
        BranchGuid = new Guid("E98B3227-C3DC-43D8-B879-D78F7C111DAF"),
        CustomerUserGuid = new Guid("60E8708C-6B49-4F83-B727-09BBC4CCC4C7"),
        Amount = 100000,
        CashAmount = 80000,
        UseWalletCredit = true,
        CallbackUrl = "leon.ir/order",
        MetaData = new List<PaymentMetaDataV1>()
        {
            new PaymentMetaDataV1 { Name = "label", Value = "burger"},
            new PaymentMetaDataV1 { Name = "table", Value = "M2"}
        }
    });
```
