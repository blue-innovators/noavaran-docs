# وضعیت های سیستم پرداخت

در این سیستم وضعیت پرداخت در مراحل مختلف تعریف میشود :
    

```cs
    1 - BranchSettlementStatus
    2 - OnlineDepositSettlementStatus
    3 - OnlineDepositStatus
    4 - TransactionGroupStatus
    5 - TransactionStatus
    6 - PaymentStatus
```

## BranchSettlementStatus
 
 این وضعیت در زمانی معنی پیدا میکند و استفاده میشود که مرحله تسویه با کسب و کار فراخوانی میشود :

```cs
    public enum BranchSettlementStatus
    {
        New = 0,
        Succeeded = 1,
        Failed = 2,
    }
```

1 - New 
    
    در تعریف اولیه و ایجاد اولیه درخواست تسویه وضعیت در حالت جدید میباشد

2 - Succeeded 

    وقتی درخواست تراکنش ارسال شد و جواب با موفقیت برگشت، وضعیت این درخواست به حالت موفق در می آید

3 - Failed 

    وقتی درخواست تراکنش ارسال شد و جواب با خطا برگشت، وضعیت این درخواست به حالت ناموفق در می آید


## OnlineDepositStatus

 این وضعیت در زمانی معنی پیدا میکند و استفاده میشود که مرحله اولیه ثبت درخواست یک پرداخت آنلاین است  :

```cs
    public enum OnlineDepositStatus
    {
        New = 0,
        Succeeded = 1,
        Failed = 2,
    }
```

1 - New 
    
    در تعریف اولیه و ایجاد اولیه درخواست ثبت وضعیت در حالت جدید میباشد

2 - Succeeded 

    وقتی درخواست تراکنش ارسال شد و همه مراحل اعم از پرداخت و دریافت تایید بانک به درستی انجام شد وضعیت موفق شده میشود

3 - Failed 

   این تراکنش به حالت ناموفق در میاید. در هر کدام از مراحل پرداخت آنللان هم به خطا برسیم وضعیت ناموفق میشود. این وضعیت در مراحل مختلف بررسی میشود. برای پرداخت های قدیمی که وضعیتشان در حالت جدید  مانده است ، بعد گذشت مدت زمان تعیین شده 


## OnlineDepositSettlementStatus

  این وضعیت زمانی استفاده میشود که یک درخواست پرداخت آنلاین صادر شده است. وضعیت های مختلف به شرح زیر است :


```cs
    public enum OnlineDepositSettlementStatus
    {
        Succeeded = 1,
        Failed = 2,
        Exception = 3
    }
``` 

1 - Succeeded 

        وقتی درخواست مرحله ثبت تراکنش و پرداخت را سپری کرد و وارد مرحله settlment بانک شد و با موفقیت این مرحله را به اتمام رساند وضعیت درخواست به حالت موفق در میاید.

2 - Failed 

        وقتی درخواست مرحله ثبت تراکنش و پرداخت را سپری کرد و وارد مرحله settlment بانک شد و این مرحله موفقیت آمیز انجام نشد وضعیت این درخواست به حالت ناموفق در میاید.

3 - Exception 

        اگر در هر کدام از مراحل بالا و همچمنینی مرحله خواندن درخواست از دیتابیس خطای ناشناخته ای اتفاق افتاد وضعیت درخواست به این حالت در میاد.


## TransactionGroupStatus 

    این وضعیت در جایی مطرح میشود که یک دستور پرداخت شامل چندی تراکنش است یعنی این وضعیت یک سطخ بالاتر از وضعیت تراکنش هاست و به طور کلی شرایط همه تراکنشها در یک دسته برای یک دستورپرداخت  برررسی میشود


```cs
    public enum TransactionGroupStatus
    {
        New = 0,
        WaitForVerify = 1,
        Succeeded = 2,
        Refunded = 3,
        Rejected = 4,
        Failed = 5,
        SemiSuccessful = 6,
        FailedRefund = 7,
    }
```

1 - New 
 
    در مرجله اول تعریف یک دسته تراکنش وضعیت آن درخواست در حالت جدید است .

2 - WaitForVerify

    در صورتی که دسته بندی تراکنش ها به صورت دو مرحله ای تغریف شود بعد از اینکه دسته تراکنشها به حالت جدید درامد بعد از گدشتن از یک مرحله ای این دو مرحله وضعیت این درخواست به صورت منتظر برای تایید در می آید.

3 - Succeeded

    بعد از اتمام مرحبه دوم یا به عبارتی کامیت شدن تراکنش ها هم در حالت تک مرحله ایی و هم حالت دو مرحله ایی اگر با موفقت انجام شود. وضعیت درخواست موفق ثبت میشود.

4 - Refunded

    برای بازگشت یک تراکنش موفق از این نوع وضعیت استفاده میکنند.

5 - FailedRefund

    اگر مرحله قبل یعنی بازگشت به هردلیلی به خطا بخورد وضعیت تراکنش به بازگشت با خطا تغییر پیدا میکیند

6 - Rejected 

    برای لغو تراکنش دو مرحله و بازگشت پول به حساب مبدا از نوع وضهیت بالا استفاده میکنیم     

7 - Failed
     
     وقتی یک دستور پرداخت تاریخش اتمام میابد به عبارتی اکسپایرد میشود تمام درخواست های دسته ای تراکنش ها که در وضعیت جدید و انتظار برای تایید هستن به وضعیت ناموق تغییر میابند.

8 - SemiSuccessful

    در تراکنش های تک مرحله ای وفتی تایید تراکنش به خطای نامشخص برخورد کرد وضعیا این درخواست را به وضعیت بالا تغییر پیدا میکند. 


## TransactionStatus 

    این وضعیت در جایی مطرح میشود که یک دستور پرداخت بعد از ایجاد شامل چندین تراکنش هست که هرکادم ار تراکنشها میتوانند شرایط پایین را داشته باشند .


```cs
    public enum TransactionStatus
    {
        New = 0,
        WaitForVerify = 1,
        Succeeded = 2,
        Refunded = 3,
        Rejected = 4,
        Failed = 5,
    }
```

1 - New 
 
    در مرجله اول تعریف یک دسته تراکنش وضعیت آن درخواست در حالت جدید است .

2 - WaitForVerify

    در صورتی که دسته بندی تراکنش ها به صورت دو مرحله ای تغریف شود بعد از اینکه دسته تراکنشها به حالت جدید درامد بعد از گدشتن از یک مرحله ای این دو مرحله وضعیت این درخواست به صورت منتظر برای تایید در می آید.

3 - Succeeded

    بعد از اتمام مرحبه دوم یا به عبارتی کامیت شدن تراکنش ها هم در حالت تک مرحله ایی و هم حالت دو مرحله ایی اگر با موفقت انجام شود. وضعیت درخواست موفق ثبت میشود.

4 - Refunded

    برای بازگشت یک تراکنش موفق از این نوع وضعیت استفاده میکنند.

5 - Rejected 

    برای لغو تراکنش دو مرحله و بازگشت پول به حساب مبدا از نوع وضهیت بالا استفاده میکنیم     

6 - Failed
     
     وقتی یک دستور پرداخت تاریخش اتمام میابد به عبارتی اکسپایرد میشود تمام درخواست های دسته ای تراکنش ها که در وضعیت جدید و انتظار برای تایید هستن به وضعیت ناموق تغییر میابند.


## PaymentStatus

    وقتی یک پرداخت انجام میشود در ابتدا یک دستور پرداخت برای ان زده میشود که این دستور پرداخت میتواند شامل یک تراکنش دسته ایی و چندین تراکنش زیر مجموعه باشد . این دستور پرداخت بر اساس وضعیت های به وجود آمده شامل وضعیت های زیر است.


```cs
    public enum PaymentStatus
    {
        New = 0,
        WaitingForOnlineDeposit = 1,
        Succeeded = 2,
        Failed = 3,
        Expired = 4,
    }  
```

1 - New 
 
    در مرحله اول هم=نگام ایجاد یک دستور پرداخت وضعیت اولیه این درخواست در حالت جدید است .

2 - WaitingForOnlineDeposit

   اگر دستور پرداختی شامل پرداخت انلاین باشد بعد از ایجاد دستور پرداخت تا زمانی که مراحل پرداخت آنلاین انجام شود، وضعیت این دستور پرداخت به وضعیت منتظر برای پرداخت آنلاین تغییر چیدا میکند

3 - Succeeded

    بعد از اتمام انجام موفق همه تراکنشهای یک دستور پرداخت وضعیت این دستورپرداخت موفق میشود.

4 - Failed
     
   در صورتی که یک دستور پرداخت در حالت لغو قرار بگیرد برای مثال وقتی کامند لغو فراخوانی شود و یا دستور پرداخت های قدیمی بررسی شود وضعیت به صورت ناموفق تفییر میکند

5 - Expired

    وقتی یک دستور پرداخت تاریخش اتمام میابد به عبارتی اکسپایرد میشود تمام درخواست های دسته ای تراکنش ها که در وضعیت جدید و انتظار برای تایید هستن به وضعیت ناموق تغییر میابند.

