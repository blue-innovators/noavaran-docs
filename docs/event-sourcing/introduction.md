# مقدمه‌ای بر  Event Sourcing و مشکلاتی را که حل میکند

<p align="center">
  <img src="/assets/images/event-sourcing.jpg" />
</p>

<p style="text-align:justify;">
رویکرد event sourcing در سال های اخیر به دلیل ارائه مزیت‌های رقابتی در کسب و کارها و چالش‌های فنی جذاب محبوبیت زیادی را کسب کرده است.
از آنجا که تاریخچه کامل فعالیت ها ذخیره میشود، این مکانیزم به کسب و کارها اجازه میدهد که جنبه‌های زیادی از داده هایشان را به صورت عمیقی درک نمایند.
این جنبه‌ها شامل درکی با جزئیات از رفتار مشتریانشان به همراه تاریخچه اطلاعات ایشان و اجرای کوئری‌های نوین  در راستای دریافت اطلاعات به منظور توسعه محصول، استراتژی‌های مارکتینگ و دیگر تصمیمات حوزه کسب و کار می باشد.
با استفاده از event soucing میتوانید بفهمید که در هر نقطه ای از زمان سیستم به چه شکلی بوده و چگونه به آن وضعیت رسیده است.
در بسیاری از دامین ها این قابلیتی متحول کننده است.
</p>

<p style="text-align:justify;">
 بسیاری از سیستم های امروزی تنها آخرین وضعیت داده ها را نگهداری میکنند و به همین دلیل امکان بررسی تاریخچه اطلاعات از دست می رود. اما مکانیزم event sourcing یک نگاه تازه به ذخیره سازی دیتا دارد.
</p>

<p style="text-align:justify;">
همچنین افرادی که با رویکرد DDD توسعه می دهند اغلب علاقمند به ترکیب event sourcing و cqrs می باشند تا به مقیاس پذیری و بهره وری بالا دست پیدا کنند.
cqrs اساساً درباره denormalize کردن داده ها می باشد اما این الگو سینرژی بسیار خوبی با event sourcing دارد بطوری که یک کامیونیتی حول این دو موضوع با هم شکل گرفته است.
</p>

<p style="text-align:justify;">
در ادامه با مثال‌هایی درباره اهمیت مکانیزم event sourcing و مقایسه آن با روش‌های سنتی صحبت خواهیم کرد. در تمامی مثال‌هایی که جلوتر بررسی خواهیم کرد یک دامین فرضی در مورد یک سرویس در شبکه موبایل که بصورت پیش پرداخت کار میکند دارند. در این دامین مشتریان اعتبار خود را با کارت شارژها یا بن کارت‌ها میتوانند افزایش دهند.
</p>