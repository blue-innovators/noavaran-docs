# جمع بندی

* Event Sourcing مدل ذخیره سازی تنها Snapshot را ذخیره تاریخچه کامل رویدادهای شکل دهنده استیت جاری جایگزین میکند.
* شما میتوانید استیت اپلیکیشن را به واسطه Event Sourcing به هر نقطه ای از گذشته برگردانید.
* ذخیره سازی تاریخچه به شما توانایی اجرای کوئری های قدرتمند که حول زمان میچرخند را میدهد که به آنها کوئری های زمانی میگوییم.
* کوئری های زمانی میتوانند یک مزیت رقابتی متحول کننده باشند که آنالیز های قدرتمند تری را امکان پذیر میکنند.
* دامین مدل ها باید شامل اگریگیت های رویداد محور باشند که امکان Event Sourcing را فراهم میکنند.
* اگریگیت های بر پایه Event Sourcing به ما اجازه میدهند قوانی بیزینس بطور شفافی نمود پیدا کنند و وابستگی کمتری به تکنولوژی های زیرساخت Persistance داشته باشیم.
* شما میتوانید یک Event Store سفارشی خودتان را بر پایه SQL Server یا دیتابیس های داکیومنت محجور ایجاد  کنید.
* Event Store یک ابزار از پیش آماده می باشد که بصورت از پیش ساخته شده از مفاهیم اساریم ها پشتیبانی میکند و مفاهیم پیشرفته ای مانند ساخت Projection را به ما میدهد.
* CQRS و Event Sourcing سینرژی خوبی با هم دارند که به پروژه ها این امکان را میدهند که view cache درست کنند.
* Event Sourcing میتواند هزینه و زحمت زیادی را بطلبد بدون اینکه ارزشی در قبال این سرمایه گذاری دریافت کنید. پس بدون ملاحظات دقیق استفاده نکنید.