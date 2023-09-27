# توسعه یک Event Store

<p style="text-align:justify;">
به منظور توسعه اپلیکیشن هایی بر پایه مکانیزم Event sourcing شما میتوانید یا از Event Store های از پیش آماده استفاده کنید و یا اینکه خودتان یک Event Store توسعه بدهید.
توسعه یک Event Store بدین معنا می باشد که از ابزارهای موجود فعلی بایستی به شکلی نوین استفاده کنیم.
در این قسمت شما یاد خواهید گرفت که چطور با استفاده از RavenDB و SQL Server یک Event Store طراحی کنید.
لازم به ذکر است توسعه یک Event Store باعث میشود شما مفاهیم اساسی Event Sourcing را عمیق تر متوجه شوید.
</p>

<p style="text-align:justify;">
در مثال های قسمت های قبل مشاهده کردید که اینترفیس IEventStore توسط PayAsYouGoAccoutRepository مورد استفاده قرار می گرفت. این اینترفیس همان چیزی است که این Event Store سفارشی و دست ساز خودمان در این قسمت پیاده سازی میکند.
</p>

```csharp
public interface IEventStore
{
	 void CreateNewStream(string streamName, IEnumerable<DomainEvent> domainEvents);

	 void AppendEventsToStream(string streamName, IEnumerable<DomainEvent> 
	domainEvents, int? expectedVersion);

	 IEnumerable<DomainEvent> GetStream(string streamName, int fromVersion, int 
	toVersion);

	 void AddSnapshot<T>(string streamName, T snapshot);

	 T GetLatestSnapshot<T>(string streamName) where T: class;
}
```

## 🔸 طراحی یک فرمت ذخیره سازی

<p style="text-align:justify;">
طراحی یا انتخاب یک فرمت ذخیره سازی بزرگترین چالش به هنگام ساخت یک Event Store می باشد. 
شما بایستی با استفاده از تکنولوژی انتخابی خودتان حداقل قابلیت‌های ایجاد استریم رویدادها، Append کردن رویدادها به یک استریم و بازیابی آنها به همان ترتیب ذخیره شده را فراهم نمایید.
همچنین ممکن است بخواهید قابلیت پشتیبانی از Snapshot ها را نیز فراهم نمایید.
</p>

<p style="text-align:justify;">
در این مثال فرمت ذخیره سازی بر پایه سه نوع داکیومنت می باشد: EventStream, EventWrapper و SnapshotWrapper.
EventStream نماینده یک استریم از رویدادها می باشد اما تنها داخل آن متادیتا ذخیره میشود. مانند شناسه و ورژن. 
EventWrapper رویداد را wrap میکند و نماینده یک رویداد دامین می باشد. این داکیومنت همه دیتای رویداد را شامل میشود. همچنین متادیتایی مبنی بر اینکه این رویداد متعلق به کدام استریم است نیز ذخیره میشود.
SnapshotWrapper نماینده یک Snapshot می باشد.
یک طراحی جایگزین میتوانست این باشد که برای هر سه این مورد از ی داکیومنت استفاده کنید و آن ها را با متادیتا تمییز دهید. که البته این دلبخواه شماست  میتوانید این مثال را از نو طراحی کنید.
</p>

<p style="text-align:justify;">
RavenDB یک دیتابیس داکیومنت محور می باشد که هر داکیومنت را با فرمت JSON  ذخیره می نماید.
کتابخانه سی شارپ این دیتابیس خود میتواند از روی ساختار POCP کلاس ها داکیومنت JSON را بسازد. پس تنها کافیست کلاس های مربوط به این سه تایپ داکیومنت تعریف شوند:
</p>

```csharp
public class EventStream
{
	 public string Id { get; private set; } //aggregate type + id
	 public int Version {get; private set;}
	 private EventStream() { }
	 public EventStream(string id)
	 {
		 Id = id;
		 Version = 0;
	 }
	 public EventWrapper RegisterEvent(DomainEvent @event)
	 {
		 Version ++;
		 return new EventWrapper(@event, Version, Id);
	 }
}
```

```csharp
public class EventWrapper
{
	 public string Id { get; private set; }
	 public DomainEvent Event { get; private set; }
	 public string EventStreamId { get; private set; }
	 public int EventNumber { get; private set; }
	 public EventWrapper(DomainEvent @event, int eventNumber, string streamStateId)
	 {
		 Event = @event;
		 EventNumber = eventNumber;
		 EventStreamId = streamStateId;
		 Id = string.Format("{0}-{1}", streamStateId, EventNumber);
	 }
}
```

```csharp
public class SnapshotWrapper
{
	 public string StreamName { get; set; }
	 public Object Snapshot { get; set; }
	 public DateTime Created { get; set; }
}
```

## 🔸 ساخت استریم رویدادها

<p style="text-align:justify;">
حال که یک نوع داکیومنت را برای استریم‌ها مشخص کرده و در نظر گرفتیم میتوانیم با ایجاد یک instance از این کلاس یک استریم درست کنیم.
مثال زیر پیاده سازی ساخت یک استریم را در Event Store نشان میدهد:
</P>

```csharp
public class EventStore : IEventStore
{
	 private readonly IDocumentSession _documentSession;
	 public EventStore(IDocumentSession documentSession)
	 {
		_documentSession = documentSession;
	 }
	 public void CreateNewStream(string streamName,
	 IEnumerable<DomainEvent> domainEvents)
	 {
		 var eventStream = new EventStream(streamName);
		 _documentSession.Store(eventStream);
		 AppendEventsToStream(streamName, domainEvents); // see next listing
	 }
}
```
<p style="text-align:justify;">
تکنولوژی زیر ساخت RavenDB توسط اینترفیس IDocumentSession به کار گرفته می شود.
در مثال بالا پیاده سازی متد CreateNewStream() با استفاده از این اینترفیس instance ساخته شده از کلاس استریم را ذخیره می کند.
در این مرحله هنوز رویدادها ذخیره نشده اند. در مثال بعد ادامه کار را خواهید دید.
</p>

## 🔸 Append کردن به استریم‌ها

<p style="text-align:justify;">
حال که یک استریم از رویداد را ایجاد کردیم چالش بعدی append کردن رویدادها به آن استریم می باشد.
در این مثال هر رویداد با شی EventWrapper ساخته می شود که در آن متادیتای لازم برای وصل شدنش به استریم مناسب نیز گذاشته می شود. پیاده سازی Event Store با مثال زیر کامل تر میشود :
</p>

```csharp
public class EventStore : IEventStore
{
	 ...
	 public void AppendEventsToStream(string streamName,
	 IEnumerable<DomainEvent> domainEvents, int? expectedVersion = null)
	{
		 var stream = _documentSession.Load<EventStream>(streamName);
		 foreach (var @event in domainEvents)
		 {
		 _documentSession.Store(stream.RegisterEvent(@event));
		 }
	}
}
```

<p style="text-align:justify;">
پیاده سازی متد AppendEventsToStream هر کدام از EventWrapper ها را به صورت یک داکیومنت یکتا ذخیره می کند. اما این موضوع در حقیقت جزئیات پیاده سازی می باشد.
از دید مصرف کننده بیرونی یک سری رویداد به یک استریم مشخص اضافه می شوند.
اما حالا که هر رویداد یک داکیومنت مجزا می باشد بایستی راهکاری جهت بازیابی تمامی رویدادهای مربوط به یک استریم بیابیم که این موضوع با اطلاعاتی که در EventWrapper ذخیره میشود حل شده است.
در متد RegisterEvent که در داخل EventStream گذاشته شده است اطلاعات ورژن و شناسه Stream به EventWrapper داده میشود.
بنابراین یافتن رویدادهای استریم به سادگی اجرای یک کوئری بر روی رویدادهایی می باشد که EventStreamId مشخصی دارند.
</p>

## 🔸 اجرای کوئری بر روی استریم‌ها

<p style="text-align:justify;">
زمانی که در حال اضافه کردن قابلیت های Event Store می باشد یکی از موارد لازم و ضروری امکان کوئری گرفتن از رویدادهای داخل یک استریم می باشد همچنین اگر بخواهید پشتیبانی از Snapshot ها را نیز اضافه نمایید بایستی بتوانید این کوئری را با پارامتر شناسه یا ورژن یک رویداد مشخص نیز اجرا کنید.
این ویژگی این امکان را می دهد که بتوانید رویدادها را از یک نقطه مشخص به بعد بازیابی نمایید.
در مثال زیر قابلیت کوئری زدن تمامی رویدادهای یک استریم به Event Store اضافه شده است :
</p>

```csharp
public class EventStore : IEventStore
{
	 ...
	 public IEnumerable<DomainEvent> GetStream(string streamName,
	 int fromVersion, int toVersion)
	 {
		 // get events from a specific version
		 var eventWrappers = (from stream in _documentSession.Query<EventWrapper>()
		 .Customize(x => x.WaitForNonStaleResultsAsOfNow())
		 where stream.EventStreamId.Equals(streamName)
		 && stream.EventNumber <= toVersion
		 && stream.EventNumber >= fromVersion
		 orderby stream.EventNumber
		 select stream).ToList();
		 
		 if (eventWrappers.Count() == 0) return null;
		 var events = new List<DomainEvent>();
		 
		 foreach (var @event in eventWrappers)
		 {
			events.Add(@event.Event);
		 }
		 return events;
	 }
	 ...
}
```

<p style="text-align:justify;">
همانطور که قبلاً به آن اشاره کردیم تعدادی داکیومنت یکتا بر پایه کلاس EventWrapper که اطلاعات رویدادها در آنها ذخیره می شوند نماینده یک stream می باشند.
در شرط Where این کوئری این موضوع مشهود می باشد که ما شناسه مشخصی از استریم را میخواهیم.
همچنین موضوع ورژن که همان شماره رویدادها به صورت ترتیبی می باشد نیز جزو شروط این کوئری می باشد که میتوانیم نقطه شروع و پایان دلخواه را مشخص کنیم.
در ادامه خواهید دید که این ویژگی به منظور افزودن قابلیت پشتیبانی از Snapshot ها کابردی می باشد.
</p>

## 🔸 پشتیبانی از Snapshot ها

<p style="text-align:justify;">
برای پشتیبانی از Snapshot ها شما بایستی امکانی را برای ذخیره سازی آنها فراهم کنید. در این مثال snapshotWrapper به منظور wrap کردن اطلاعات Snapshot ها بعلاوه متادیتایی که مشخص میکند به کدام استریم است و در چه تاریخی اضافه شده است، می باشد.
مثال زیر نشان میدهد که به Event Store چطور قابلیت ذخیره سازی Snapshot ها را اضافه کنیم:
</p>

```csharp
public class EventStore : IEventStore
{
	 ...
	 public void AddSnapshot<T>(string streamName, T snapshot)
	 {
		 var wrapper = new SnapshotWrapper
		 {
			 StreamName = streamName,
			 Snapshot = snapshot,
			 Created = DateTime.Now
		 };
		 _documentSession.Store(snapshot);
	 }
	 ...
}
```

<p style="text-align:justify;">
در متد AddSnapshot به مصرف کننده این امکان داده میشود  که برای یک استریم خاص یک Snapshot ذخیره کند.
نام استریم در حقیقت متادیتای Snashot می باشد که به واسطه آن میتوانیم آخرین Snapshot یک استریم را بدست آوریم.
</p>

```csharp
public class EventStore : IEventStore
{
	 ...
	 public void AddSnapshot<T>(string streamName, T snapshot)
	 {
		 var wrapper = new SnapshotWrapper
		 {
			 StreamName = streamName,
			 Snapshot = snapshot,
			 Created = DateTime.Now
		 };
		 _documentSession.Store(snapshot);
	 }
	 public T GetLatestSnapshot<T>(string streamName) where T: class
	 {
		 var latestSnapshot = _documentSession.Query<SnapshotWrapper>()
		 .Customize(x => x.WaitForNonStaleResultsAsOfNow())
		 .Where(x => x.StreamName == streamName)
		 .OrderByDescending(x => x.Created)
		 .FirstOrDefault();
		 
		 if (latestSnapshot == null)
		 {
			return null;
		 }
		 else
		 {
			 // unwrap snapshot - give user what he passed to AddSnapshot
			 return (T)latestSnapshot.Snapshot;
		 }
	 }
}
```

<p style="text-align:justify;">
آنچه در مثال بالا مبینید کاربرد متادیتای ذخیره شده بر روی Snapshot می باشد. به هنگام کوئری زدن برای یک Snapshot از نام استریم و تاریخ به منظور فیلتر کردن و سورت کردن استفاده میشود.
</p>

## 🔸 مدیریت Concurrency

<p style="text-align:justify;">
بسیاری از اپلیکیشن های مدرن چند کاربره هستند. بنابراین بسیار محتمل است که دو کاربر بخواهند همزمان یک دیتایی را مشاهده کرده و آپدیت کنند. اگر یکی از کاربران بخواهد قسمتی از دیتا را آپدیت کند بدون علم به اینکه کاربر دیگری قبل از اون اینکار را کرده است بایستی عملیات آپدیت با خطا مواجه شود. همانطور که قبلاً اشاره شد این Optimistic Concurrency می باشد.
یک راه حل کلی برای هندل کردن همزمانی در Event Store ها بررسی کردن آخرین شماره ورژن ذخیره شده و ورژنی که ما میخواهیم آپدیتش کنیم می باشد.
</p>

<p style="text-align:justify;">
در مثال های قبلی دیدیم که چطور با نگه داشتن پراپرتی InitialVersion میتوانیم ورژن اولیه Aggregate خود را نگهداری کنیم.
در مثال زیر می بینید که چطور متد AppendEventsToStream در پیاده سازی Event Store را تغییر می دهیم تا با استفاده از مقدار این پراپرتی فرآیند آپدیت را در صورت مغایرت آخرین ورژن با مقدار در دست لغو کند.
لغو این فرآیند و برگرداندن خطا به این معناست که کاربر از آپدیت دیتای در دست احتمالاً آگاه نبوده است.
</p>

```csharp
public void AppendEventsToStream(string streamName,
 IEnumerable<DomainEvent> domainEvents, int? expectedVersion = null)
{
	 var stream = _documentSession.Load<EventStream>(streamName);
	 if (expectedVersion != null)
	 {
		CheckForConcurrencyError(expectedVersion, stream);
	 }
	 foreach (var @event in domainEvents)
	 {
		_documentSession.Store(stream.RegisterEvent(@event));
	 }
	}
	
	private static void CheckForConcurrencyError(int? expectedVersion,
	 EventStream stream)
	{
	 var lastUpdatedVersion = stream.Version;
	 if (lastUpdatedVersion != expectedVersion)
	 {
		 var error = string.Format("Expected: {0}. Found: {1}", expectedVersion, 
		lastUpdatedVersion);
		 throw new OptimsticConcurrencyException(error);
	 }
}
```

<p style="text-align:justify;">
در مثال بالا چنانچه به متد AppendEventsToStream مقدار ورژن پاس داده شود با استفاده از متد CheckForConcurrencyError قبل از ذخیره کردن رویدادها بررسی می شود که آیا مقدار ورژن در دست با آخرین ورژن ذخیره شده یکسان می باشد یا نه.
در صورتی که این دو مقدار یکسان نباشند با ایجاد خطای OptimsticConcurrencyException فرآیند آپدیت لغو میشود.
</p>

<p style="text-align:justify;">
متاسفانه راهکار بالا به هنگام استفاده از RavenDB به اندازه کافی قدرتمند نمی باشد. اگر با دقت نگاه کنید در کد فاصله کوتاهی میان متدی که همزمانی را چک میکند و جایی که ذخیره سازی رویدادها رخ می دهد وجود دارد.
اگر بعد از چک کردن همزمانی و قبل از ذخیره کردن رویدادها رویداد دیگری در دیتابیس ذخیره شود چکار میتوان کرد؟
خوشبختانه دیتابیس RavenDB این امکان را فراهم کرده که بتوان در سطح دیتابیس تنظیماتی انجام داد که از این همزمانی جلوگیری شود. مثال زیر این مورد را نشان می دهد:
</p>

```csharp
public static class Bootstrapper
{
	 public static void Startup()
	 {
		 var documentStore = new DocumentStore
		 {
			ConnectionStringName = "RavenDB"
		 }.Initialize();
		 
		 documentStore
		 .DatabaseCommands
		 .EnsureDatabaseExists("EventSourcingExample");
		 
		 ObjectFactory.Initialize(config =>
		 {
			 // register other dependencies here, too
			 config.For<IDocumentStore>().Use(documentStore);
			 config.For<IDocumentSession>()
			 .HybridHttpOrThreadLocalScoped()
			 .Use(x =>
			 {
				 var store = x.GetInstance<IDocumentStore>();
				 var session = store.OpenSession();
				 session.Advanced.UseOptimisticConcurrency = true;
				 return session;
			 });
		 });
	 }
}
```

<p style="text-align:justify;">
RavenDB خود از مدیریت Optimistic Concurrrency جلوگیری میکند.
در مثال بالا با ست کردن مقدار session.Advanced.UseOptimisticConcurrency به true اینکار انجام شده است.
نکته دیگری که مثال بالا نماینگر آن اس این است که این تنظیم در سطح تزریق وابستگی سرویس انجام شده است. بدین شکل که تامین کننده سرویس های مورد نیاز هر بخش از برنامه به ازای هر درخواست و هر thread به آن بخش یک instance جدید که در آن تنظیم همزمانی انجام شده است داده می شود.
</p>

<p style="text-align:justify;">
هرگاه در RavenDB همزمانی رخ بدهد خطای ConcurrencyExceptions رخ میدهد.
این خطا بدین معناست که به هنگام آپدیت توسط ما از طریق یک IDocumentSession دیگر دیتای مورد نیاز ما قبلاً آپدیت شده است.
مثال زیر نشان میدهد چطور این خطا در سطح Application Service هندل می شود :
</p>

```csharp
public class TopUpCredit
{
	 private IPayAsYouGoAccountRepository _payAsYouGoAccountRepository;
	 private IDocumentSession _unitOfWork;
	 private IClock _clock;
	 public TopUpCredit(IPayAsYouGoAccountRepository payAsYouGoAccountRepository,
	 IDocumentSession unitOfWork, IClock clock)
	 {
		 _payAsYouGoAccountRepository = payAsYouGoAccountRepository;
		 _unitOfWork = unitOfWork;
		 _clock = clock;
	 }
	 
	 public void Execute(Guid id, decimal amount)
	 {
		 try
		 {
			 var account = _payAsYouGoAccountRepository.FindBy(id);
			 var credit = new Money(amount);
			 account.TopUp(credit, _clock);
			 _payAsYouGoAccountRepository.Save(account);
			 _unitOfWork.SaveChanges();
		 }
		 catch (ConcurrencyException ex)
		 {
			 _unitOfWork.Advanced.Clear();
			 // any additional error handling - retry messages, error queue, etc.
			 throw ex;
		 }
	 }
}
```

<p style="text-align:justify;">
در مثال بالا می بینید که برای این سرویس یک نمونه از IDocumentSession پاس داده می شود.
سپس در این سرویس account مورد نظر جستجو میشود. و بر روی آن متد TopUp صدا زده می شود.
صدا زدن این متد باعث میشود که رویدادی به لیست رویدادهای Uncomitted اضافه شود. سپس در متد Save این تغییرات برای ذخیره سازی صف می شوند.
در ادامه وقتی SaveChanges صدا زده میشود اطلاعات بر روی دیتابیس ذخیره میشوند.
اگر در طی این فرآیند ذخیره کردن RavenDB متوجه شود که یک instance دیگر از IDocumentSession بر روی استریم فعلی رویدادی ذخیره کرده است خطای ConcurrencyException ایجاد میشود که همانطور میبینید در این مثال هندل شده است.
</p>

## 🔸 یک Event Store بر پایه SQl Server

<p style="text-align:justify;">
SQL Server یک گزینه رایج دیگر برای استفاده به عنوان زیرساخت ذخیره سازی در یک Event Store می باشد. پروژه های اوپن سورس زیادی وجود دارد که راهنما و رفرنس پیاده سازی Event Store بر پایه SQL Server می باشند.
پروژه های Ncqrs و NEventStore از معروف ترین این پروژه ها هستند. در مثال های بعدی ما از Ncqrs به عنوان یک کیس استادی استفاده خواهیم کرد.
</p>

### 🔹 انتخاب یک Schema

<p style="text-align:justify;">
زمانی که تصمیم میگیرید از Sql برای ذخیره سازی رویدادها و استریم ها استفاده نمایید مهم ترین موضوع Schema یا همان ساختار جداول می باشد.
کسانی که قبلاً با Sql یک Event Store توسعه داده اند توصیه میکنند که با ذخیره سازی دیتاهای پیچیده و متغیل بصورت XML یا Json دیزاین ساختار جداول را مینیمال نگه داریم.
ساختار پیشنهادی ما یک ساختار مستقل از دامین و عمومی به شکل زیر می باشد:
</p>

```sql
-- Table Events:

 Id [uniqueidentifier] NOT NULL,
 TimeStamp [datetime] NOT NULL,
 Name [varchar](max) NOT NULL,
 Version [varchar](max) NOT NULL,
 EventSourceId [uniqueidentifier] NOT NULL,
 Sequence [bigint],
 Data [nvarchar](max) NOT NULL
 
-- Table EventSources:

 Id [uniqueidentifier] NOT NULL,
 Type [nvarchar](255) NOT NULL,
 Version [int] NOT NULL
 
--- Table Snapshots:

 EventSourceId [uniqueidentifier] NOT NULL,
 Version [bigint] NULL,
 TimeStamp [datetime] NOT NULL,
 Type [varchar](255) NOT NULL,
 Data [varbinary](max) NOT NULL
```
<p style="text-align:justify;">
یک تفاوت عمده این طراحی با Event Store بر پایه RavenDB استفاده از uniqueidentifier به عنوان شناسه می باشد. در اپلیکیشن خودتان شما میتوانید البته این موضوع را به دلخواه خود تغییر دهید.
</p>


### 🔹 ساخت یک استریم

<p style="text-align:justify;">
بیشتر پیاده سازی Ncqrs evet store متاثر از schema می باشد. به عنوان مثال ساخت یک استریم تنها به سادگی اضافه کردن یک رکورد به جدول EventSources می باشد.
توجه کنید که در Ncqrs عبارت event source همان معادل مفهوم event stream می باشد.
</p>

```csharp
private static void AddEventSource(Guid eventSourceId, Type eventSourceType,
 long initialVersion, SqlTransaction transaction)
{
		using (var command =
	 new SqlCommand(Queries.InsertNewProviderQuery,
	 transaction.Connection))
		{
			command.Transaction = transaction;
			command.Parameters.AddWithValue("Id", eventSourceId);
			command.Parameters.AddWithValue("Type", eventSourceType.ToString());
			command.Parameters.AddWithValue("Version", initialVersion);
			command.ExecuteNonQuery();
		}
}

internal static class Queries
{
	 ...
		public const String InsertNewProviderQuery =
	 "INSERT INTO [EventSources](Id, Type, Version) VALUES (@Id, @Type, @Version)";
	 ...
}
```

### 🔹 ذخیره سازی رویدادها

<p style="text-align:justify;">
schema همچنین بر روی ذخیره کردن رویدادها نیز تاثیر میگذارد.
مثال زیر نشان میدهد که چطور هر رویداد با رفرنس به استریم مربوطه به صورت یک رکورد در جدول Events ذخیره می شود.
</p>

```csharp
private void SaveEvents(IEnumerable<UncommittedEvent> uncommittedEvents,
 SqlTransaction transaction)
{
    foreach (var sourcedEvent in uncommittedEvents)
    {
        SaveEvent(sourcedEvent, transaction);
    }
}

private void SaveEvent(UncommittedEvent uncommittedEvent, SqlTransaction transaction)
{
    string eventName;
    var document = _formatter.Serialize(uncommittedEvent.Payload, out eventName);
    var storedEvent = new StoredEvent<JObject>(
	 uncommittedEvent.EventIdentifier,
	 uncommittedEvent.EventTimeStamp, eventName,
	 uncommittedEvent.EventVersion,
	 uncommittedEvent.EventSourceId, 
	 uncommittedEvent.EventSequence, document
	 );
	 
    var raw = _translator.TranslateToRaw(storedEvent);
    using (var command = new SqlCommand(
	Queries.InsertNewEventQuery, transaction.Connection))
    {
        command.Transaction = transaction;
        command.Parameters.AddWithValue("EventId", raw.EventIdentifier);
        command.Parameters.AddWithValue("TimeStamp", raw.EventTimeStamp);
        command.Parameters.AddWithValue("EventSourceId", raw.EventSourceId);
        command.Parameters.AddWithValue("Name", raw.EventName);
        command.Parameters.AddWithValue("Version", raw.EventVersion.ToString());
        command.Parameters.AddWithValue("Sequence", raw.EventSequence);
        command.Parameters.AddWithValue("Data", raw.Data);
        command.ExecuteNonQuery();
    }
}

internal static class Queries
{
	 ...
			public const String InsertNewEventQuery = "INSERT INTO [Events]([Id],
	[EventSourceId], [Name], [Version], [Data], [Sequence], [TimeStamp]) VALUES
	(@EventId, @EventSourceId, @Name, @Version, @Data, @Sequence, @TimeStamp)";
	 ...
}
```

### 🔹 لود کردن رویدادها از یک استریم

<p style="text-align:justify;">
مثال زیر یک سمپل از Ncqrs می باشد که رویدادها را از یک استریم میخواند. اینکار مشابه IEventStore.GetStream() می باشد که پیشتر در این فصل نشان داده شده است. همه رویدادهایی که به یک استریم خاص به واسه شناسه ان رفرنس دارند اگر در محدوده ورژن های پاس داده شده به این متد باشند برگردانده میشوند.
</p>

```csharp
public CommittedEventStream ReadFrom(Guid id, long minVersion, long maxVersion)
{
    var events = new List<CommittedEvent>();
    using (var connection = new SqlConnection(_connectionString))
    {
        using (var command = new SqlCommand(Queries.SelectAllEventsQuery, connection))
        {
            command.Parameters.AddWithValue("EventSourceId", id);
            command.Parameters.AddWithValue("EventSourceMinVersion", minVersion);
            command.Parameters.AddWithValue("EventSourceMaxVersion", maxVersion);
            connection.Open();
            using (SqlDataReader reader = command.ExecuteReader())
            {
                while (reader.Read())
                {
                    var evnt = ReadEventFromDbReader(reader);
                    events.Add(evnt);
                }
            }
        }
    }
    return new CommittedEventStream(id, events);
}

internal static class Queries
{
	 ...
	 public const String SelectAllEventsQuery =
	 "SELECT [Id], [EventSourceId],[Name], [Version], [TimeStamp], [Data], Sequence] 
	 FROM [Events] WHERE [EventSourceId] = @EventSourceId
	 AND [Sequence] >= @EventSourceMinVersion AND
	 [Sequence] <= @EventSourceMaxVersion
	 ORDER BY [Sequence]";
	 ...
}
```

<p style="text-align:justify;">
شما همچنین میتوانید پیاده سازی کامل ReadEventFromDbReader() را در ریپازیتوری گیت هاب Ncqrs ببینید.
</p>

### 🔹 Snapshot ها

<p style="text-align:justify;">
یک جدول به خصوص به منظور نگهداری Snapshot ها در Ncqrs در نظر گرفته شده است. در این جدول یک رفرنس به استریم مربوطه از طریق نگهداری شناسه آن برقرار شده است.
در مثال زیر ذخیره سازی و لود شدن Snapshot ها را مشاهده میکنید:
</p>

```csharp
public void SaveSnapshot(Snapshot snapshot)
{
    using (var connection = new SqlConnection(_connectionString))
    {
        connection.Open();
        using (SqlTransaction transaction = connection.BeginTransaction())
        {
            try
            {
                using (var dataStream = new MemoryStream())
                {
                    var formatter = new BinaryFormatter();
                    formatter.Serialize(dataStream, snapshot.Payload);
                    byte[] data = dataStream.ToArray();
                    using (var command = new SqlCommand(Queries.InsertSnapshot, transaction.Connection))
                    {
                        command.Transaction = transaction;
                        command.Parameters.AddWithValue("EventSourceId", snapshot.EventSourceId);
                        command.Parameters.AddWithValue("Version", snapshot.Version);
                        command.Parameters.AddWithValue("Type", snapshot.GetType().AssemblyQualifiedName);
                        command.Parameters.AddWithValue("Data", data);
                        command.ExecuteNonQuery();
                    }
                }
                transaction.Commit();
            }
            catch
            {
                transaction.Rollback();
                throw;
            }
        }
    }
}

internal static class Queries
{
	 ...
		public const String InsertSnapshot =
		"DELETE FROM [Snapshots] WHERE [EventSourceId]=@EventSourceId;
		INSERT INTO [Snapshots]([EventSourceId], [Timestamp], [Version], [Type], [Data]) 
		VALUES (@EventSourceId, GETDATE(), @Version, @Type, @Data)";
	 ...
}
```

```csharp
public Snapshot GetSnapshot(Guid eventSourceId, long maxVersion)
{
    using (var connection = new SqlConnection(_connectionString))
    {
        connection.Open();
        using (var command = new SqlCommand(Queries.SelectLatestSnapshot, connection))
        {
            command.Parameters.AddWithValue("@EventSourceId", eventSourceId);
            using (var reader = command.ExecuteReader())
            {
                if (reader.Read())
                {
                    var snapshotData = (byte[])reader["Data"];
                    using (var buffer = new MemoryStream(snapshotData))
                    {
                        var formatter = new BinaryFormatter();
                        var payload = formatter.Deserialize(buffer);
                        var theSnapshot = new Snapshot(eventSourceId, (long)reader["Version"], payload);
                        return theSnapshot.Version > maxVersion ? null : theSnapshot;
                    }
                }
                return null;
            }
        }
    }
}

internal static class Queries
{
	 ...
		public const String SelectLatestSnapshot =
		"SELECT TOP 1 * FROM [Snapshots] WHERE [EventSourceId]=@EventSourceId
		ORDER BY Version DESC";
	 ...
}
```

## 🔸 آیا ساخت یک Event Store سفارشی معقول است؟

<p style="text-align:justify;">
در طی روزهای اولیه event sourcing هیچ ابزار تجاری وجود نداشت و توسعه دهندگان مجبور بودند که قابلیت های event sourcing را با استفاده از تکنولوژی‌های موجود مانند SQL یا دیتابیس های داکیومنت محور توسعه دهند. این بدین معناست که این کار صد در صد قابل دستیابی می باشد. اما اگر شما به سمت سناریوهای پیچیده تری مانند projection ها، کوئری‌های زمانی پیچیده و افزایش مقیاس پذیری بروید ممکن است به جای خلق ارزش برای کسب و کار بیشتر به توسعه امکانات زیرساخت مشغول خواهید شد.
به همین دلیل است که شاید شما بخواهید از ابزارهای تکنولوژیک آماده برای این منظور به مانند Event Store توسعه داده شده توسط Greg Young استفاده نمایید. این ابزار به نام Event Store نیز شناخته می شود.
</p>