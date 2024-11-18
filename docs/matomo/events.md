# رویدادها

رویدادها همه اتفاقاتی هستند که به صورت پیش فرض در ماتومو تعریف نشده اند و با برنامه نویسی اتفاق می افتند

مثلا زمانی که کاربر روی یک دکمه کلیک میکند، رویداد کلیک کاربر به سمت سرور ماتومو برمیگردد

رویدادها را میتوان در منوی جانبی، از بخش های زیر عنوان Behavior پیدا کرد

برای تعریف یک رویداد باید تکه کدی شبیه زیر به کد اپ اضافه شود

```html
<a href="mailto:name@example.com" title="Email Us" onclick="_paq.push(['trackEvent', 'Contact', 'Email Link Click', 'name@example.com']);">Email Us</a>
```

یا به صورت زیر

```js
_paq.push(['trackEvent', 'Contact', 'Email Link Click', 'name@example.com']);
```

کد بالا رویدادی را با داده های زیر رهگیری میکند:

Event Category: Contact
Event Action: Email Link Click
Event Name: name@example.com

سپس میتوانید در صفحه رویدادها این بخش ها را مجزا مشاهده کنید

