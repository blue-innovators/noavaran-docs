# وضعیت های سیستم پرداخت

:در این سیستم وضعیت پرداخت در مراحل مختلف تعریف میشود
    

```cs
    1 - BranchSettlementStatus
    2 - OnlineDepositSettlementStatus
    3 - OnlineDepositStatus
    4 - TransactionGroupStatus
    5 - TransactionStatus
    6 - PaymentStatus
```

## BranchSettlementStatus
 
 :این وضعیت در زمانی معنی پیدا میکند و استفاده میشود که مرحله تسویه با کسب و کار فراخوانی میشود 

```cs
    public enum BranchSettlementStatus : short
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