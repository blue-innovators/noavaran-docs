# Aggregate ها با مکانیزم Event Sourcing

<p style="text-align:justify;">
به منظور سازگاری با Event sourcing بایستی Aggreagte ها رویداد محول باشند. و مهم است که بتوانند وضعیتشان را از روی event ها محاسبه کنند.
افرادی که با رویکرد DDD توسعه می دهند دریافته اند که یک اثر جانبی مثبت مکانیزم Event Sourcing رفتار محور بودن Aggregate ها می باشد که این موضوع سطح بیانگر بودن رویدادهای دامین را افزایش داده است.
یک مزیت رضایت بخش دیگر این است که persistance بیشتر loosely coupled و کمتر مشکل خیز می باشد.
</p>

## 🔸 ساختار

<p style="text-align:justify;">
زمانی که میخواهیم Aggregate هایی را در یک دامیت Event Sourced شده ایجاد کنیم چندین نکته کلیدی وجود دارد.
اولین نکته این است که آن Aggregate بتواند یک رویداد دامین را بر روی خودش با توجه به قوانین بیزینسی اعمال نماید.
دومین نکته این است که بایستی یک لیست از رویدادهای uncommited بایستی بر روی Aggreagte نگهداری شوند که بتوانیم در نهایت آنها را در Event Store ذخیره نماییم.
سومین نکته این است که هر Aggregate بایستی ورژن داشته باشد که بتوانیم از روی آن Snapshot بسازیم و از روی آن ها Aggreagte را بسازیم.
</p>

### 🔹 اضافه کردن قابلیت های Event Sourcing

<p style="text-align:justify;">
اغلب بهتر است که یک کلاس پایه برای Aggregate ها بسازیم که تمامی دیگر Aggregate ها از آن ارث ببرند. با اینکار مشترکات Aggreagte ها را کپسوله کرده ایم.
</p>

```csharp
public abstract class EventSourcedAggregate : Entity
{
	 public List<DomainEvent> Changes { get; private set; }
	 public int Version { get; protected set; }
	 public EventSourcedAggregate()
	 {
		Changes = new List<DomainEvent>();
	 }
	 public abstract void Apply(DomainEvent changes);
}
```
<p style="text-align:justify;">
در اینجا Changes یک لیست از رویدادهای Uncomitted می باشد.
همچنین Version ورژن Aggregate را نگهداری می کند.
متد Apply نیز یک رویداد دامین را میگیرد و Aggregate بایستی آن را با توجه به قوانین بیزینسی دامین اعمال نماید.
حال بیایید یک مثال از پیاده سازی این کلاس پایه در دامین اپراتور تلفن همراه به منظور مدل کردن Aggregate حساب مشتری ببینیم :
</p>

```csharp
public class PayAsYouGoAccount : EventSourcedAggregate
{
	 private FreeCallAllowance _freeCallAllowance;
	 private Money _credit;
	 private PayAsYouGoInclusiveMinutesOffer _inclusiveMinutesOffer =
	 new PayAsYouGoInclusiveMinutesOffer();
	 public PayAsYouGoAccount()
	 { }
	 public override void Apply(DomainEvent @event)
	 {
		 When((dynamic)@event);
		 Version = Version ++;
	 }
	 private void When(CreditAdded creditAdded)
	 {
		 _credit = _credit.Add(creditAdded.Credit);
	 }
	 private void When(PhoneCallCharged phoneCallCharged)
	 {
		 _credit = _credit.Subtract(phoneCallCharged.CostOfCall);
		 if (_freeCallAllowance != null)
		 _freeCallAllowance.Subtract(phoneCallCharged.CoveredByAllowance);
	 }
	 private void When(AccountCreated accountCreated)
	 {
		 Id = accountCreated.Id;
		 _credit = accountCreated.Credit;
	 }
	 ...
}
```

<p style="text-align:justify;">
پیاده سازی متد Apply از دیتا تایپ dynamic کمک میگیرد که این امکان را ایجاد میکند که Handler های بسیار گویا داشته باشیم.
این Handler ها همان متدهای When می باشند.
هر متد When انتظار میرود که برای خواننده کد شفاف باشد که به چه تایپ رویدادی واکنش نشان میدهد.
متد Apply همچنین ورزن Aggregate را آپدیت می نماید.این موضوع به ما کمک میکند که بفهمیم قبل از اینکه ادامه دهیم آیا عملیات مورد نظر اجرا شده است یا نه.
مقدار Version در Aggregate یک عدد ترتیبی است نسبت به ایتدای رشته رویدادهای آن Aggregate.
</p>

### 🔹 ارائه api های Domain-Focused

<p style="text-align:justify;">
متد Apply جزئیات پیاده سازی داخلی یک Aggregate می باشد و نباید بتوان آن را از بیرون صدا زد. تنها دلیلی که این متد public می باشد برای این است که بتوانیم Aggregate را Rehydrate نماییم.
موضوع Rehydrate کردن جلوتر توضیح داده می شود. در بیرون یک Aggregate سرویس ها همچنان از طریق Api های گویا با Aggregate ارتباط برقرار می کنند.
یعنی سرویس ها از رویدا محور بودن aggregate ها بی خبر می باشند.
مثال این موضوع متد Topup در Aggregate حساب مشتری که نامش PayAsYouGoAccount می باشد :
</p>

```csharp
public class PayAsYouGoAccount : EventSourcedAggregate
{
	 private FreeCallAllowance _freeCallAllowance;
	 private Money _credit;
	 private PayAsYouGoInclusiveMinutesOffer _inclusiveMinutesOffer =
	 new PayAsYouGoInclusiveMinutesOffer();
	 public PayAsYouGoAccount()
	 { }
	 ...
	 public void TopUp(Money credit, IClock clock)
	 {
		 if (_InclusiveMinutesOffer.IsSatisfiedBy(credit))
		 Causes(new CreditSatisfiesFreeCallAllowanceOffer(
		 this.Id, clock.Time(), _inclusiveMinutesOffer.FreeMinutes)
		 );
		 Causes(new CreditAdded(this.Id, credit));
	 }
	 private void Causes(DomainEvent @event)
	 {
		 Changes.Add(@event);
		 Apply(@event);
	 }
	 ...
}
```

<p style="text-align:justify;">
در مثال بالا نمایی از آنچه سرویس های دامین و اپلیکیشن بیرونی میبینند مشاهده میکنید. این سرویس ها نمی دانند که این Aggregate از مکانیزم Event Sourcing استفاده میکند.
این سرویس ها تنها متدهای سطح بالایی مانند Topup را میبینند که که مفاهیم دامین را بیان میکند.
از این زاویه نگاه این Aggregate ها با Aggregate هایی که بر پایه مکانیزم Event sourcing  کار نمیکنند فرقی ندارند.
برای شفاف تر شدن این موضوع بیایید استفاده از این aggregate را در سطح سرویس اپلیکیشن ببینیم :
</p>

```csharp
// application service
public class TopUpCredit
{
	 private IPayAsYouGoAccountRepository _payAsYouGoAccountRepository;
	 private IDocumentSession _unitOfWork;
	 private IClock _clock;
	 public TopUpCredit(IPayAsYouGoAccountRepository payAsYouGoAccountRepository,
	 IClock clock)
	 {
		 _payAsYouGoAccountRepository = payAsYouGoAccountRepository;
		 _clock = clock;
	 }
	 public void Execute(Guid id, decimal amount)
	 {
		 ...
		 var account = _payAsYouGoAccountRepository.FindBy(id);
		 var credit = new Money(amount);
		 // .. expressive domain-focused API - no hint of event sourcing
		 account.TopUp(credit, _clock);
		 _payAsYouGoAccountRepository.Save(account);
		 ...
	 }
}
```

<p style="text-align:justify;">
وقتی متد Topup صدا زده میشود یا رویداد
CreditSatisfiedFreeCallAllowanceOffer
و یا رویداد
CreditAdded
ایجاد می شود که نوع رویداد بر پایه قوانین InclusiveMinutesOfferBusiness مشخص می گردد.
این متدها پس از پاس داده شدن به متد Apply وضعیت Aggregate را تغییر می دهند. 
اما این رویدادها همچنین باید persist شوند تا هر زمان که Aggregate لود شد بازپخش شوند.
متد Causes این دغدغه ها را مدیریت میکند و رویداد را به فیلد changes اضافه می کند که مجموعه ای از رویدادهای uncomitted می باشد و سپس رویداد را به متد Apply پاس می دهد.
</p>

### 🔹 افزودن پشتیبانی از Snapshot ها

## 🔸 Persisting & Rehydrating

### 🔹 ساخت یک Repository بر پایه Event Sourcing

### 🔹 افزودن Persistance و Reload کردن Snapshot ها

## 🔸 مدیریت Concurrency

## 🔸 اجرای تست