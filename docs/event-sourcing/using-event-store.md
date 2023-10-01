# استفاده از Event Store آماده

<p style="text-align:justify;">
تصمیم برای استفاده از یک Event Store آماده میزان کاری که بایستی انجام دهید تا مکانیزم Event Sourcing را راه بندازید به صورت مستقیمی کاهش می دهد.
همچنین با انتخاب ابزاری به مانند Event Store توسعه داده شده توسط Greg Young شما فیچر های بسیار دیگری به مانند Projection های پیشرفته و multinode clustering را برای محیط های بسیار مقیاس پذیر خواهید داشت.
به منظور نمایش استفاده از این Event Store در این قسمت مثال هایمان همچنان حول دامین اپراتور فرضی تلفن همراه می باشد.
در ادامه خواهید دید که چطور اینترفیس IEventStore با استفاده از کتابخانه سی شارپ کلاینت آن پیاده سازی نمایید. و در ادامه این موارد اجرای کوئری های زمانی و ساخت Projection ها را از طریق رابط کاربری پنل ادمین آن خواهید دید. برای اینکه بتوانید مثال های زیر را پیاده سازی کرده و استفاده نمایید بایستی Event Store را بر روی سیستم خود نصب داشته باشید.
</p>

## 🔸 نصب Greg Young Event Store

<p style="text-align:justify;">
به منظور استفاده از Event Store لازم است که ابتدا نسخه سرور آن را بر روی کامپیوترتان نصب نمایید. جزئیات نصب و نحوه دریافت فایل نصبی مخصوص Event Store را میتوانید از سایتش دریافت نمایید.
</p>

## 🔸 استفاده از کتابخانه C# کلاینت

<p style="text-align:justify;">
به منظور استفاده از Event Store بایستی یک لایه persistance بنویسید که در آن کتابخانه سی شارپ نسخه کلاینت این دیتابیس استفاده خواهید کرد.
در این لایه persistance ما به دنبال پیاده سازی اینترفیس IEventStore می باشیم که هر زمان که بخواهید میتوانید آن را پیاده سازی سفارشی خودتان بر روی RavenDB جایگزین نمایید. پیاده سازی مثال‌های بعدی از دو پکیج نوگت زیر استفاده می نماید : 
</p>

* EventStore.Client
* Newtonsoft.Json

<p style="text-align:justify;">
EventStore.Client کتابخانه رسمی Event Store می باشد که به منظور ارتباط کلاینت با سرور توسعه داده شده است. دیگر پکیج ذکر شده همانطور که مشخص است به منظور کار کردن با فرمت JSON می باشد.
در مثال زیر کلاس GetEventStore را مشاهده میکنید که پیاده سازی IEventStore با Event Store می باشد:
</p>

``` csharp
public class GetEventStore : IEventStore
{
	 private IEventStoreConnection esConn;
	 private const string EventClrTypeHeader = "EventClrTypeName";
	 public GetEventStore(IEventStoreConnection esConn)
	 {
		this.esConn = esConn;
	 }
	 
	 public void CreateNewStream(string streamName, IEnumerable<DomainEvent> domainEvents)
	 {
		 // automatically creates a stream when events are added to it
		 AppendEventsToStream(streamName, domainEvents, null);
	 }
	 
	 public void AppendEventsToStream(string streamName, IEnumerable<DomainEvent> domainEvents, int? expectedVersion)
	 {
		 var commitId = Guid.NewGuid();
		 var eventsInStorageFormat = domainEvents.Select(
		 e => MapToEventStoreStorageFormat(e, commitId, e.Id)
		 );
		 esConn.AppendToStream(StreamName(streamName), expectedVersion ?? ExpectedVersion.Any, eventsInStorageFormat);
	 }
	 
	 private EventData MapToEventStoreStorageFormat(object evnt, Guid commitId, Guid eventId)
	 {
		 var headers = new Dictionary<string, object>
		 {
			 // each event in this operation will be associated with the same commit
			 {"CommitId", commitId},
			 // store type of class so event can be rebuilt when the event is loaded
			 {EventClrTypeHeader, evnt.GetType().AssemblyQualifiedName}
		 };
		 
		 // events and headers are stored at binary-encoded JSON
		 var data = Encoding.UTF8.GetBytes(JsonConvert.SerializeObject(evnt));
		 var metadata = Encoding.UTF8.GetBytes(
		 JsonConvert.SerializeObject(headers)
		 );
		 
		 // enahnced support in the admin web UI if Event Store knows events are JSON
		 var isJson = true;
		 return new EventData(eventId, evnt.GetType().Name, isJson, data, metadata);
	 }
	 
	 public IEnumerable<DomainEvent> GetStream(string streamName, int fromVersion, int toVersion)
	 {
		 // Event Store wants number of events to retrieve
		 // not highest version
		 var amount = (toVersion - fromVersion) + 1;
		 var events = esConn.ReadStreamEventsForward(StreamName(streamName), fromVersion, amount, false);
		 // map events back from JSON to DomainEvent. Header indicates the type
		 return events.Events.Select(e => (DomainEvent)RebuildEvent(e));
	 }
	 
	 private object RebuildEvent(ResolvedEvent eventStoreEvent)
	 {
		 var metadata = eventStoreEvent.OriginalEvent.Metadata;
		 var data = eventStoreEvent.OriginalEvent.Data;
		 var typeOfDomainEvent = JObject.Parse(Encoding.UTF8.GetString(metadata)).Property(EventClrTypeHeader).Value;
		 var rebuiltEvent = JsonConvert.DeserializeObject(Encoding.UTF8.GetString(data), Type.GetType((string)typeOfDomainEvent));
		 return rebuiltEvent;
	 }
	 
	 // snapshots in Event Store are just events in dedicated snapshot streams
	 // explained: http://stackoverflow.com/questions/16359330/is-snapshotsupported-from-greg-young-eventstore
	 public void AddSnapshot<T>(string streamName, T snapshot)
	 {
		 var stream = SnapshotStreamNameFor(streamName);
		 var snapshotAsEvent = MapToEventStoreStorageFormat(snapshot, Guid.NewGuid(), Guid.NewGuid());
		 esConn.AppendToStream(stream, ExpectedVersion.Any, snapshotAsEvent);
	 }
	 
	 public T GetLatestSnapshot<T>(string streamName) where T : class
	 {
		 var stream = SnapshotStreamNameFor(streamName);
		 var amountToFetch = 1; // just the latest one
		 var ev = esConn.ReadStreamEventsBackward(stream, StreamPosition.End, amountToFetch, false);
		 if (ev.Events.Any())
			return (T)RebuildEvent(ev.Events.Single());
		 else
			return null;
	 }
	 
	 private string SnapshotStreamNameFor(string streamName)
	 {
		 // snapshots are just events in separate streams
		 return StreamName(streamName) + "-snapshots";
	 }
	 
	 private string StreamName(string streamName)
	 {
		 // Event Store projections require only a single hypen ("-")
		 // see: https://groups.google.com/forum/#!msg/eventstore/D477bKLcdI8/62iFGhHdMMIJ
		 var sp = streamName.Split(new []{ '-' }, 2);
		 // remove all hyphens except the first
		 return sp[0] + "-" + sp[1].Replace("-", "");
	}
}
```
<p style="text-align:justify;">
خوب است که مثال بالا را با دقت نگاه کنید. همانطور که احتمالاً متوجه شده اید استفاده از یک Event store آماده کار پیاده سازی کمتری نسبت به رویکرد پیاده سازی به وسیله SQL Server و RavenDB دارد.
دو نکته قابل توجه در مثال بالا مسئله کوئری ها و مدیریت همزمانی می باشد.
زمانی که به واسه متد GetEventStream کوئری اجرا میکنید کافیست تنها نام استریم و ورژن ابتدایی و پایانی را به Event Store بدهید.
در پیاده سازی بر پایه RavenDB برای این موضوع پیچیدگی بیشتری داشت و این تفاوت به دلیل این است که Event Store بصورت درونی از مفاهیمی مانند استریم پشتیبانی میکند.
</p>

<p style="text-align:justify;">
پشتیبانی built-int  از استریم باعث میشود مدیریت Optimistic Concurrency زحمت کمتری داشته باشد. شما بایستی تنها ورژن مورد انتظار خود را از آخرین رویداد ذخیره شده  به متد IEventStoreConnection.AppendToStream() پاس دهید و خود Event Store تمامی چک کردن ها را انجام میدهد.
</p>

<p style="text-align:justify;">
یک مزیت RavenDB نسبت به مثال فعلی این است که خودش  موضوع Serlialization  و Deserialization برای JSON ها را پشتیبانی میکند.
در مثال بالا اگر دقت کنید اسم کلاس در متد MapToEventStoreStorageFormat ذخیره میشود. این هدر به منظور بازسازی رویدادها وقتی از دیتابیس واکشی میشوند استفاده میشود.
در این حالت لازم است که منطق Serialization و Deserialization را خودتان مدیریت کنید.
</p>

<p style="text-align:justify;">
پشتیبانی از Snapshot در Event Store به واسطه ساخت یک استریم جدید برای Snapshot های هر Aggregate انجام میشود.
میتوانید ببینید که در متد GetLatestSnapshot که به واسطه یک کوئری که توسط متد IEventStore
.ReadStreamEventsBackwards() اجرا میشود تنها آخرین مورد Snapshot را دریافت کنید.
</p>

<p style="text-align:justify;">
یک نکته مهم که در مثال بالا نشان داده نشده بود ساخت کانکشن اولیه با Event Store می باشد. مثال زیر نشان میدهد چطور اینکار انجام میشود:
</p>

```csharp
IPEndPoint endpoint = new IPEndPoint(IPAddress.Loopback, 1113);
IEventStoreConnection con = EventStoreConnection.Create(endpoint)
con.Connect();
```
<p style="text-align:justify;">
در مثال بالا یک ارتباط TCP بر روی سیستم لوکال و بر روی پورت پیش فرض 1113 برقرار شده است. البته که ممکن است این تنظیمات در محیط پروداکشن متفاوت باشد.
</p>

<p style="text-align:justify;">
همچنین آخرین نکته در مورد مثال بالا این است که در اینجا صرفاً برای نمایش مثال از متدهای Sync استفاده شد که در محیط واقعی بایستی با جایگزین Async خود جا به جا شوند.
</p>

## 🔸 اجرای کوئری‌های زمانی

<p style="text-align:justify;">
Greg Young  و تیم Event Store تصمیم گرفتند که جاوا اسکریپت بهترین ابزار ایجاد کوئری و Projection می باشد. خواهید دید که چطور با استفاده از رابط کاربری راحت Event Store میتوانید کوئری های زمانی ایجاد کنید.
البته که شما ابتدا بایستی مقداری دیتای تستی وارد Event Store کنید که بتوانید کوئری های مورد نظر را اجرا نمایید.
در مثال های بعدی فرض میشود که برای اگریگیت با نام PayAsYouGoAccount که همان حساب کاربری در اپراتور فرضی تلفن همراه ما بود تعدادی رویداد تستی ذخیره شده است. 
با باز کردن آدرسی مشابه  http://localhost:2113/web شما میتوانید رابط کاربری Event Store را مشاهده کنید.
در این رابط کاربری به مانند شکل زیر خواهید دید که استریم هایی ایجاد شده است :
</p>

<p align="center">
  <img src="/assets/images/event-sourcing-figure-22-4.png" />
</p>

### 🔹 کوئری از یک استریم

<p style="text-align:justify;">
برخلاف اسم به ظاهر سخت کوئری های زمانی، ساخت این کوئری ها لازم نیست پیچیده و طولانی باشد.
یک کوئری کوتاه که میتواند مفید نیز باشد نمایش تعداد کل دقایقی است که یک مشتری به مکالمه درون شبکه اپراتور فرضی پرداخته است. حال در روز جاری یا هر روز دیگری در گذشته. 
این کوئری میتواند به مشتریان کمک کند که متوجه شوند آیا بیش از حد تماس برقرار میکنند یا خیر. 
مثال زیر نشان میدهد که چطور با استفاده از کد جاوا اسکریپت میتوان چنین کوری زمانی در Event Store ساخت.
</p>

```js
fromStream('PayAsYouGoAccount-5b3415c8d58a4dcf955eed9978bcd8b1')
.when({
	 // initialize the state
	 $init : function(s,e) {return {minutes : 0}},
	 
	 "PhoneCallCharged" : function(s,e) {
		 var dateOfCall = e.data.PhoneCall.StartTime;
		 var june4th = "2014-06-04";
		 if (dateOfCall.substring(0, 10) == june4th) {
			s.minutes += e.data.PhoneCall.Minutes.Number;
		 }
	 }
 });
```

<p style="text-align:justify;">
کوئری ها در Event Store با مشخص کردن اینکه کدام رویدادها را میخواهید کوئری بزنید شروع میشوند.
در مثال بالا در متد FromStream مشخص شده که این کوئری بایستی به رویدادهای استریم یک instance از یک Aggregate مربوط به حساب کاری اعمال شوند.
شناسه این استریم از همان رابط کاربری برداشته شده است.
تمامی رویدادهای مشخص شده به ترتیب از قدیمی به جدید داخل متد When می آیند.
در متد When مشخص میشود که کدام یک از رویدادها به چه شکلی هندل شوند.
در مثال بالا میبینید که تنها رویدادهای PhoneCallCharged هندل خواهند شد.
</p>

<p style="text-align:justify;">
نتیحه یک کوئری در Event Store بصورت یک آبجکت جاوا اسکریپتی که با عنوان state کوئری شناخته میشوند برگردانده میشوند.
این آبجکت هر بار به متد when پاس داده میشود و بنا به منطق داخل این متد مقدارش ممکن است بروز رسانی شود.
در مثال همچنان میتوانید ببینید که در هندلر init آبجکت جاوا اسکریپتی با یک پراپرتی minutes ایجاد شده است.
در این قسمت بنا به دلخواه خودتان میتوانید ساختار استیت کوئری را مشخص نمایید.
در مثال بالا مقدار پراپرتی minutes هر بار که یک رویداد PhoneCallCharged به متد when فرستاده میشود تغییر میکند و به مقدارش اضافه میشود. 
البته که شرط تاریخ نیز چک میشود.
بعد اجرای کوئری در آدرسی مانند http://localhost:2113/web/query.htm میتوانید خروجی کوئری را به شکل زیر مشاهده نمایید :
</p>

<p align="center">
  <img src="/assets/images/event-sourcing-figure-22-5.png" />
</p>

<p style="text-align:justify;">
در مثال بالا میتوانید ببینید که نتیجه کوئری یا همان استیت کوئری در سمت راست نمایش داده شده است.
همانطور که اشاره شد ساختار استیت کوئری کاملاً توسط  خومان قابل تغییری و سفارشی سازی می باشد.
به عنوان مثال میتوانیم استیت کوئری را به گونه ای تغییر دهیم که چندین عدد که نماینگر میزان دقایق مکالمه در چندین تاریخ می باشد را نشان دهد. مثال زیر را ببینید :
</p>

```js
fromStream('PayAsYouGoAccount-86e057b961624c6f92c9ab3c56e6e232')
.when({
	 // initialize the state
	 $init : function(s,e) {return { june3rd: 0, june4th: 0, june5th: 0}},
	 "PhoneCallCharged" : function(s,e) {
		 var dateOfCall = e.data.PhoneCall.StartTime;
		 var june3rd = "2014-06-03";
		 var june4th = "2014-06-04";
		 var june5th = "2014-06-05";
		 if (dateOfCall.substring(0, 10) == june3rd) {
			s.june3rd += e.data.PhoneCall.Minutes.Number;
		 }
		 if (dateOfCall.substring(0, 10) == june4th) {
			s.june4th += e.data.PhoneCall.Minutes.Number;
		 }
		 if (dateOfCall.substring(0, 10) == june5th) {
			s.june5th += e.data.PhoneCall.Minutes.Number;
		 }
	 }
	 // handle other types of event and update the state accordingly
 });
```

<p style="text-align:justify;">
در مثال بالا ما سه پراپرتی داریم که هر کدام مقدار دقایق مکالمه در یک تاریخ مشخص را نشان میدهد. در متد when با توجه به شروزی که برای تاریخ گذاشته شده است پراپرتی های مختلف استیت ممکن است تغییر کنند.
</p>

### 🔹 کوئری از چند استریم

<p style="text-align:justify;">
Event Store محدود به کوئری از تنها یک استریم نمی باشد. در بسیاری از سناریوها لازم است که دیتاهای چندین استریم را با یکدیگر ترکیب کنیم. اینکار معادل Join در SQL  می باشد. در مثال زیر کوئری ایجاد شده کل دقایق مکالمه مشتریان مختلف را در سه تاریخ مشخص محاسبه کرده است :
</p>

```js
fromCategory('PayAsYouGoAccount')
.when({
	 // initialize the state
	 $init : function(s,e) {return { june3rd: 0, june4th: 0, june5th: 0}},
	 
	 "PhoneCallCharged" : function(s,e) {
		 var dateOfCall = e.data.PhoneCall.StartTime;
		 var june3rd = "2014-06-03";
		 var june4th = "2014-06-04";
		 var june5th = "2014-06-05";
		 if (dateOfCall.substring(0, 10) == june3rd) {
			s.june3rd += e.data.PhoneCall.Minutes.Number;
		 }
		 if (dateOfCall.substring(0, 10) == june4th) {
			s.june4th += e.data.PhoneCall.Minutes.Number;
		 }
		 if (dateOfCall.substring(0, 10) == june5th) {
			s.june5th += e.data.PhoneCall.Minutes.Number;
		 }
	 }
	 // handle other types of event and update the state accordingly
 });
```

<p style="text-align:justify;">
متد fromCategory در Event Store تمامی استریم هایی که با عبارت آورده شده در پارامتر ورودی آن شروع میشوند را با هم ترکیب میکند.
البته این متد انتظار دارد بعد از این عبارت یک Hyphen آمده باشد. به عمین دلیل است که در مثال پیاده سازی IEventStore بالاتر دیدید که متد StreamName تمامی Hyphen ها را به جز مورد اول پاک میکند.
</p>

## 🔸 ساخت Projection ها

<p style="text-align:justify;">
گاهی اوقات به جای محاسبه یک استیت شما میخواهید این توانایی را داشته باشید که زیرمجموعه ای از رویدادها را داشته باشید و یک استریم جدید از روی آنها بسازید. این قابلیت میتواند در سیستم های بزرگ که میلیون ها و یا شاید میلیاردها رویداد داشته باشند مهم باشد.
به عنوان مثال دامین اپراتور فرضی موبایل را تصور کنید. در این دامین ممکن است میلیون ها مشتری داشته باشید که هر کدام تعداد قابل توجهی فعالیت که منجر به ثبت رویداد میشود انجام میدهند.
حال اگر بخواهید یک کوئری بر روی دیتای تمام مشتریان انجام دهید این کوئری یک کوئری غیر بهینه خواهد بود.
Projection ها این مشکل را به این شکل حل میکنند که به شما اجازه میدهند تعداد رویداد مشخصی را به نحوی مشخص کنید و آنها را در یک استریم جدید ذخیره نمایید.
حال کوئری های خود را میتوانید بر روی این استریم جدید اجرا نمایید.
در مثال زیر میتوانید یک Projection را ببنید که تمامی عملیات های Topup مشتریان را در یک استریم جدید ذخیره میکند:
</p>

```js
fromCategory('PayAsYouGoAccount')
.when({
	 "CreditAdded": function(s, event) {
		linkTo('AllTopUps', event);
	 }
});
```

<p style="text-align:justify;">
اجرای Projection مثال بالا منجر به این میشود که تمامی رویدادهای CreditAdded که در استریم های مربوط به اگریگیت PayAsYouGoAccount هستند را به یک استریم جدید با عنوان AllTopUps اضافه نماید. این بدین معناست که حالا میتوانید تنها بر رویدادهای CreditAdded بدون اینکه لازم باشد دیگر رویدادهای اگریگیت ها را لود نمایید کوئری اجرا کنید.
متدی که این کار را انجام میدهد LinkTo می باشد که از تمامی استریم هایی که نامشان با ورودی متد FromCategory شروع میشود رویدادهای CreditAdded را به استریم AllTopUps وصل میکند.
برای اجرای پروجکشن به قسمت Projections در منوی رابط کاربری Event Store رفته و فرم ایجاد آن را مطابق شکل زیر پر نمایید:
</p>

<p align="center">
  <img src="/assets/images/event-sourcing-figure-22-6.png" />
</p>

<p style="text-align:justify;">
در مثال بالا اگر مقدار Select Mode را برابر با Continuous بگذارید رویدادهای Projection همزان با استریم های مبدا به روز میشوند.
این فیچر مخصوصاً زمانی که بخواهید CQRS اجرا کنید مفید واقع میشود. که در قسمت های بعدی به آن خواهیم پرداخت.
</p>