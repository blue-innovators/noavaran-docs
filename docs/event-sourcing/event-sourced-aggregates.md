# Aggregate ها با مکانیزم Event Sourcing

<p style="text-align:justify;">
به منظور سازگاری با Event sourcing بایستی Aggreagte ها رویداد محول باشند. و مهم است که بتوانند وضعیتشان را از روی event ها محاسبه کنند.
افرادی که با رویکرد DDD توسعه می دهند دریافته اند که یک اثر جانبی مثبت مکانیزم Event Sourcing رفتار محور بودن Aggregate ها می باشد که این موضوع سطح بیانگر بودن رویدادهای دامین را افزایش داده است.
یک مزیت رضایت بخش دیگر این است که persistance بیشتر loosely coupled و کمتر مشکل خیز می باشد.
</p>

## 🔸 ساختار

<p style="text-align:justify;">
زمانی که میخواهیم Aggregate هایی را در یک دامین Event Sourced شده ایجاد کنیم چندین نکته کلیدی وجود دارد.
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
متد Apply همچنین ورژن Aggregate را آپدیت می نماید. این موضوع به ما کمک میکند که بفهمیم قبل از اینکه ادامه دهیم آیا عملیات مورد نظر اجرا شده است یا نه.
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

<p style="text-align:justify;">
آخرین موضوع باقیمانده قبل از آنکه سراغ موضوع Persistance برویم توانایی ساخت Snapshot ها می باشد.
خود Aggregate ها بایستی Snapshot ها را بسازند. زیرا که این Aggregate است که کل منطق دامین که میتواند رویدادها را بازبخش نماید درون خود دارد. در همین راستا یک متد به منظور ساخت Snapshot و یک متد به منظور Restore کردن Aggregate از روی Snapshot اضافه میکنیم :
</p>

```csharp
public class PayAsYouGoAccount : EventSourcedAggregate
{
	 private FreeCallAllowance _freeCallAllowance;
	 private Money _credit;
	 private PayAsYouGoInclusiveMinutesOffer _inclusiveMinutesOffer =
	 new PayAsYouGoInclusiveMinutesOffer();
	 ...
	 // constructor overload - restore aggregate from snapshot
	 public PayAsYouGoAccount(PayAsYouGoAccountSnapshot snapshot)
	 {
		 Version = snapshot.Version;
		 _credit = new Money(snapshot.Credit);
	 }
	 public PayAsYouGoAccountSnapshot GetPayAsYouGoAccountSnapshot()
	 {
		 return new PayAsYouGoAccountSnapshot
		 {
		 Version = Version,
		 Credit = _credit.Amount
		 };
	 }
	 ...
}
```
<p style="text-align:justify;">
در مثال بالا overload متد constroctor مربوط به Aggregate یک Snapshot را در ورودی دریافت میکند و وضعیت Aggregate را با Snapshot یکی میکند.
همانطور که عقب تر اشاره کردیم این یک میانبر به منظور افزایش پرفورمنس برنامه می باشد که معادل پازپخش تمامی رویدادهای قبل از ورژن آن Snapshot می باشد.
برای اینکه این راهکار به درستی جلو برود لازم است که Snapshot تمامی اطلاعات مورد نیاز Aggregate را درون خود داشته باشد که در زمان ساخت بتوانند مورد استفاده قرار بگیرند.
کلاس مربوط به Snapshot مثال بالا را ببینید :
</p>

```csharp
public class PayAsYouGoAccountSnapshot
{
	 public int Version { get; set; }
	 public decimal Credit { get; set; }
}
```

## 🔸 Persisting & Rehydrating

<p style="text-align:justify;">
Persist کردن Aggregate های بر پایه مکانیزم Event Sourcing به معنای ذخیره سازی رویدادهای Comitt نشده در Event Store می باشد.
همچین لود کردن Aggregate که به آن Rehydrating نیز گفته میشود که در طی اینکار بایستی تمامی رویدادهای قبلی را restore کرده و بازپخش کنید.
البته که اینکار بایستی این امکان را داشته باشد که از Snapshot ها به عنوان یک میانبر استفاده نماید.
در طی مثال های قبل دیدید که متدهای Apply و Change به منظور همین Persistance مورد استفاده قرار گرفتند.
</p>

### 🔹 ساخت یک Repository بر پایه Event Sourcing

<p style="text-align:justify;">
حال بیایید یک ریپازیتوری نمونه برای Aggregate مثال قبلی که PayAsYouGoAccount نام داشت ببینی. البته توجه داشته باشید در این مثال موضوع Snapshot ها در نظر گرفته نشده است.
</p>

```csharp
public class PayAsYouGoAccountRepository : IPayAsYouGoAccountRepository
{
	 private readonly IEventStore _eventStore;
	 public PayAsYouGoAccountRepository(IEventStore eventStore)
	 {
		_eventStore = eventStore;
	 }
	 public void Add(PayAsYouGoAccount payAsYouGoAccount)
	 {
		 var streamName = StreamNameFor(payAsYouGoAccount.Id);
		 _eventStore.CreateNewStream(streamName, payAsYouGoAccount.Changes);
	 }
	 public void Save(PayAsYouGoAccount payAsYouGoAccount)
	 {
		 var streamName = StreamNameFor(payAsYouGoAccount.Id);
		 _eventStore.AppendEventsToStream(streamName, payAsYouGoAccount.Changes);
	 }
	 public PayAsYouGoAccount FindBy(Guid id)
	 {
		 var streamName = StreamNameFor(id);
		 var fromEventNumber = 0;
		 var toEventNumber = int.MaxValue ;
		 var stream = _eventStore.GetStream(
		 streamName, fromEventNumber, toEventNumber
		 );
		 var payAsYouGoAccount = new PayAsYouGoAccount();
		 foreach(var @event in stream)
		 {
			payAsYouGoAccount.Apply(@event);
		 }
		 return payAsYouGoAccount;
	 }
	 private string StreamNameFor(Guid id)
	 {
		 // stream per-aggregate: {AggregateType}-{AggregateId}
		 return string.Format("{0}-{1}", typeof(PayAsYouGoAccount).Name, id);
	 }
}

```
<p style="text-align:justify;">
در مثال بالا شما سه عملیات اصلی که بایستی پشتیبانی شوند را مشاهده می کنید: ساختن رشته یا همان استریم ها، اضافه کردن رویدادها به یک استریم و لود کردن استریم ها.
ساخت یک استریم شامل ایجاد یک نام برای آن استریم و اضافه کدن رویدادهای اولیه به ان می باشد.
در متد Save لیست رویدادهای Commmit نشده به یک استریم اضافه می شود.
و متد آخر یک intance از PayAsYouGoAccount را لود می نماید.
در لود کردن یک Aggregate بایستی مشخص کنید که رویدادها را از کجا تا کجا میخواهید.
سپس رویدادهایی که بدست آورده اید را با متد Apply داخل Aggregate بر روی آن اعمال میکنید.
این قسمت تنها جایی است که متد Apply داخل یک Aggregate از بیرون صدا زده می شود و این همان جایی است قبل تر در رابطه با public بودن این متد به آن اشاره کردیم.
</p>

<p style="text-align:justify;">
نکته قابل توجه دیگر نام استریم ها می باشد که به ازای هر Aggregate یکتا می باشد. و این موضوع باعث میشود به ازای هر Aggregate یک استریم جدا داشته باشیم که در نتیجه آن پرفومنس بهتری خواهیم داشت.
</p>

### 🔹 افزودن Persistance و Reload کردن Snapshot ها

<p style="text-align:justify;">
Snapshot ها یک راهکار به منظور افزایش پرفورمنس می باشند که از لود شدن کل تاریخچه یک استریم جلوگیری می نمایند.
Snapshot ها معمولاً توسط جاب های پس زمینه ای ایجاد میشوند.
مثال زیر یک جاب برای همین منظور را با استفاده از یک Event Store بر پایه RevenDB نشان میدهد :
</p>

```csharp
public class PasAsYouGoAccountSnapshotJob
{
	 private IDocumentStore _documentStore;
	 public PasAsYouGoAccountSnapshotJob(IDocumentStore documentStore)
	 {
		this._documentStore = documentStore;
	 }
	 public void Run()
	 {
		 while(true)
		 {
			 foreach (var id in GetIds())
			 {
				 using (var session = _documentStore.OpenSession())
				 {
					 var repository = new PayAsYouGoAccountRepository(
					 new EventStore(session)
					 );
					 var account = repository.FindBy(Guid.Parse(id));
					 var snapshot = account.GetPayAsYouGoAccountSnapshot();
					 repository.SaveSnapshot(snapshot, account);
				 }
			 }
			 // create a new snapshot for each aggregate every 12 hours
			 Thread.Sleep(TimeSpan.FromHours(12));
		 }
	 }
	 private IEnumerable<string> GetIds()
	 {
		 using (var session = _documentStore.OpenSession())
		 {
			 return session.Query<EventStream>()
			 .Select(x => x.Id)
			 .ToList();
		 }
	 }
}
```
<p style="text-align:justify;">
در این جاب هر 12 ساعت یکبار برای تمامی aggreagte ها یک Snapshot ایجاد و ذخیره می شود.
در ورژن های پروداکشن ممکن است بخواهید متفاوت تر عمل کنید.
احتمالاً بخواهید که لاگ ها سیستمی جهت ردیابی عملکرد این جاب اضافه نمایید.
یا اینکه یکباره کل Aggregate ها را برایشان Snapshot نسازید و تکه تکه این کار را انجام دهید.
و حتی ممکن است بخواهید بعد از اینکه تعداد رویدادهای مشخصی بعد از آخرین Snapshot رخ داد یک Snapshot جدید بسازید.
حال که Snapshot ها را می سازیم بیایید ریپازیتوری مثال گذشته را بهینه تر کنیم :
</p>

```csharp
public PayAsYouGoAccount FindBy(Guid id)
{
	 var streamName = StreamNameFor(id);
	 var fromEventNumber = 0;
	 var toEventNumber = int.MaxValue ;
	 var snapshot = _eventStore.GetLatestSnapshot<PayAsYouGoAccountSnapshot>(
	 streamName
	 );
	 if (snapshot != null)
	 {
		fromEventNumber = snapshot.Version + 1; // load only events after snapshot
	 }
	 var stream = _eventStore.GetStream(streamName, fromEventNumber, toEventNumber);
	 PayAsYouGoAccount payAsYouGoAccount = null;
	 if (snapshot != null)
	 {
		payAsYouGoAccount = new PayAsYouGoAccount(snapshot);
	 }
	 else
	 {
		payAsYouGoAccount = new PayAsYouGoAccount();
	 }
	 foreach(var @event in stream)
	 {
		payAsYouGoAccount.Apply(@event);
	 }
	 return payAsYouGoAccount;
}
```
<p style="text-align:justify;">
در اینجا میبینید که ابتدا تلاش می شود که آخرین Snapshot از Event Store گرفته شود.
اگر این snapshot وجود داشت Aggregate ابتدا از روی آن ساخته می شود و رویدادهایی که بعد از ورژن آخرین Snapshot رخ داده اند restore می شوند و Apply می شوند.
در صورتی که Snapshot ای وجود نداشت کارها به روال سابق انجام می شود.
</p>

## 🔸 مدیریت Concurrency

<p style="text-align:justify;">
گاهی اوقات شما میخواهید که عملیات اضافه کردن یک رویداد به یک استریم انجام نشود. این موضوع معمولا زمانی به وجود می‌آید که چندین کاربر یعی دارند وضعیت یک Aggregate را بصورت همزمان آپدیت نمایند.
در این حالت ممکن است شما بخواهید در حالتی که Aggregate پیش از آنکه شما تغییر خود را اعمال کنید توسط کاربر دیگری بروز شده و نسخه به روز آن در دستتان نیست، از اعمال تغییرات جلوگیری کنید.
به این تکنیک Optimistic Concurrency Control یا به اختصار OCC می گویند.
در اینجا جا دارد که درباره موضوع جستجو و مطالعه فرمایید.
</p>

<p style="text-align:justify;">
در یک اپلیکیشن بر پایه مکانیزم Event sourcing شما میتوانید با اضافه کردن دو مورد از تکنیک OCC برای حل مشکل همزمانی استفاده نمایید.
در ابتدا به Aggregate یک فیلد دیگر اضافه میکنیم که به هنگام لود شدن آن را با مقدار ورژن اولیه پر میکنیم.
سپس به هنگام ذخیره سازی تغییرات چک میکنیم که آیا آن ورژنی که ما در ایندا دریافت کردیم و میخواهیم تغییرش دهیم همان آخرین ورژن دیتای ذخیره شده می باشد یا نه.
دو تکه کد زیر این موضوع را نشان می دهند:
</p>

```csharp
public class PayAsYouGoAccount : EventSourcedAggregate
{
	 ...
	 // once set, does not change
	 public int InitialVersion { get; private set; }
	 public PayAsYouGoAccount(PayAsYouGoAccountSnapshot snapshot)
	 {
		 Version = snapshot.Version;
		 InitialVersion = snapshot.Version;
		 _credit = new Money(snapshot.Credit);
	 }
	 ...
}
```

```csharp
public class PayAsYouGoAccountRepository : IPayAsYouGoAccountRepository
{
	 ...
	 public void Save(PayAsYouGoAccount payAsYouGoAccount)
	 {
		 var streamName = StreamNameFor(payAsYouGoAccount.Id);
		 var expectedVersion = GetExpectedVersion(payAsYouGoAccount.InitialVersion);
		 _eventStore.AppendEventsToStream(streamName, payAsYouGoAccount.Changes,
		 expectedVersion);
	 }
	 private int? GetExpectedVersion(int expectedVersion)
	 {
		 if (expectedVersion == 0)
		 {
			 // first time the aggregate is stored, there is no expected version
			 return null;
		 }
		 else
		 {
			return expectedVersion;
		 }
	 }
	 ...
}
```
<p style="text-align:justify;">
در دو مثال بالا در PayAsYouGoAccount پراپرتری InitialVersion نماینده ورژن آخرین رویدادی می باشد که قبل از لود شدن Aggregate ذخیره شده بود. 
مقدار این پراپرتی به متد APpendEventsToStream پاس داده می شود و در پیاده سازی Event Store چک میشود که آیا این مقدار همچنان آخرین ورژن ذخیره شده می باشد یا خیر.
در ادامه نحوه پیاده سازی این فیچر را خواهید دید.
</p>

## 🔸 اجرای تست

<p style="text-align:justify;">
نوشتن Unit test برای Aggregate ها به معنای این می باشد که چک کنیم آیا رویداد مورد نظرمان اتفاق افتاده اند یا خیر. اینکار با چک کردن تغییرات Uncomitted متعلق به آن Aggregate صورت می گیرد.
مثال زیر یک نمونه تست برای متد PayAsYouGoAccount.TopUp() می باشد:
</p>

```csharp
[TestClass]
public class PayAsYouGoAccount_Tests
{
	 static PayAsYouGoAccount _account;
	 static Money _fiveDollars = new Money(5);
	 static PayAsYouGoInclusiveMinutesOffer _free90MinsWith10DollarTopUp =
	 new PayAsYouGoInclusiveMinutesOffer(
	 new Money(10), new Minutes(90)
	 );
	 
	 [ClassInitialize] // runs first
	 public static void When_applying_a_top_up_that_does_not_qualify_for_inclusive
	_minutes(TestContext ctx)
	 {
		 _account = new PayAsYouGoAccount(Guid.NewGuid(), new Money(0));
		 // remove the AccountCreated event that is not relevant to this test
		 _account.Changes.Clear();
		 _account.AddInclusiveMinutesOffer(_free90MinsWith10DollarTopUp);
		 // $5 top up does not meet $10 threshold for free minutes
		 _account.TopUp(_fiveDollars, new SystemClock());
	 }
	 [TestMethod]
	 public void The_account_will_be_credited_with_the_top_up_amount_but_no_free
	_minutes()
	 {
		 var lastEvent = _account.Changes.SingleOrDefault() as CreditAdded;
		 Assert.IsNotNull(lastEvent);
		 Assert.AreEqual(_fiveDollars, lastEvent.Credit);
		 Assert.AreEqual(5, _account.GetPayAsYouGoAccountSnapshot().Credit);
	 }
}
```

<p style="text-align:justify;">
بعد از اینکه به PayAsYouGoAccount، AddInclusiveMinutesOffer را اضافه کردیم
اگر مشتری بیشتر از میزان تعیین شده Topup انجام دهد میزان دقایق اضافه تری نیز دریافت خواهد کرد
در این تست بررسی میشود که اگر شرایط این آفر مورد تایید نبود بعد از انجام Topup بایستی رویداد CreditAdded را داشته باشیم و حساب مشتری تنها به مقداری که خریداری شده شارژ میشود.
</p>