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

## 🔸 اجرای کوئری‌های زمانی

### 🔹 کوئری از یک استریم

### 🔹 کوئری از چند استریم

## 🔸 ساخت Projection ها