# سنجش معایب Event Sourcing

<p style="text-align:justify;">
مهم است که درباره جنبه های منفی Event Sourcing واقع گرا باشید.
توسعه دهندگان Event Store ها نیز به این امر توصیه میکنند.
شما لازم است که زمان زیادی را برای این مکانیزم سرمایه گذاری کنید و بازخورد آن ممکن است ارزش آن را نداشته باشد.
مخصوصاً که بایستی یک سری موارد که در ادامه آن ها را بیان خواهیم کرد در نظر بگیرید.
</p>

## 🔸 ورژنینگ

<p style="text-align:justify;">
همانطور که درباره دامین اپلیکیشن خود بیشتر یاد خواهید گرفت و محصولی که توسعه میدهید را ارتقا میدهید ممکن است اطلاعاتی به دامین مدل شما اضافه شوند.
در نتیجه ممکن است بخواهید رویدادها را تغییر نام دهید یا دیتای آن را تغییر دهید.
این موضوع یک مشکل بزرگ است زیرا که شما استریم هایی خواهید داشت که رویدادها در آن با فرمت قبلی خواهند بود.
راهکارهایی برای این موضوع وجود خواهد داشت و افورت زیادی نیاز ندارد.
اگر چه اگر به این موضوع توجه کافی نشوند در طولانی مدت ممکن است مشکل زا باشد.
</p>

## 🔸 مفاهیم و مهارت های جدید برای یادگیری

<p style="text-align:justify;">
Projection ها، کوئری های زمانی و Snapshot ها مفاهیم جدید هستند که تیم های درگیر مکانیزم Event Sourcing با آن درگیر خواهند شد.
به اضافه بسیاری موارد دیگر بایستی آزمایشات عملی را با تئوری ترکیب کرد تا بتوان در این زمینه حرفه ای شد.
همچنین بایستی آمادگی این موضوع را داشته باشید که که نرخ سرعت توسعه تان در کوتاه مدت کند شود. برای این مضوع بایستی زمان یادگیری تخصیص داده شود.
همچنین این موضوع ممکن است تایم آنبودرینگ نیروهای تازه کاری که با این مفاهیم آشنا نیستند را افزایش دهد.
</p>

## 🔸 تکنولوژی های جدید

<p style="text-align:justify;">
شما ممکن است از تکنولوژی های فعلی تیم مانند SQL Server برای مقاصد Event Store استفاده کنید اما خیلی محتمل است که بخواهید از ابزار آماده Event Store استفاده کنید.
هزینه اینکار زمان مورد نیاز برای نه فقط یادگیری این تکنولوژی بلکه اجرای آن بر روی سرورهای پروداکشن، مانیتورینگ و بررسی استفاده از منابع می باشد.
سخت است که تعیین کنیم چقدر زمان مورد نیاز خواهد بود ولی برای استفاده از Event Store در پروداکشن قطعاً لازم است که زمانی برای حرفه ای شدن افراد در نظر گرفت.
</p>

## 🔸 نیازمندی به ابزار ذخیره سازی بیشتر

<p style="text-align:justify;">
پرواضح است که ذخیره سازی تغییرات دیتا بجای آخرین وضعیت آن حجم بیشتری از اطلاعات را شامل میشود. 
بنابریان نیاز به فضای ذخیره سازی بیشتری خواهد بود.
البته که این روزها تجهیزات ذخیره سازی بسیار ارزان می باشند و بعید است که برای کسی این موضوع دغدغه جدی بشود ولی خب باید در نظر گرفتش.
</p>