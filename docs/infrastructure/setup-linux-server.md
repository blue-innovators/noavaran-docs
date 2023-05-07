# نصب سرور لینوکس
در این راهنما نحوه راه اندازی یک سرور لینوکس که بتواند نرم افزارها را اجرا نماید توضیح داده شده است:

## آماده سازی سیستم عامل
قبل از هر چیز باید سیستم عامل را آماده نماییم:

```
sudo apt update && sudo apt upgrade
```

## نصب docker
نصب داکر طبق راهنمای موجود در وبسایت رسمی داکر در [این آدرس](https://docs.docker.com/engine/install/ubuntu/) انجام خواهد شد.
جهت توشیحات بیشتر یا درصورت بروز مشکل به داکیومنت ذکر شده مراجعه نمایید.

```sh
sudo apt-get install ca-certificates curl gnupg lsb-release

sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo docker run hello-world
```

درصورتی که دستور آخر که برای گرفتن ایمیج داکر از سایت داکرهاب است دچار مشکل بود باید از ابزارهای رفع تحریم مثل [شکن](https://shecan.ir/) استفاده نمایید

در ادامه میتوانید جهت بی نیازی از استفاده از sudo در ابتدای تمام دستورات داکر از روش زیر استفاده نمایید:

```sh
sudo groupadd docker
sudo usermod -aG docker $USER

# logout and login again

newgrp docker
docker images
```

درصورتی که کاربر دیگری بخواهد در این سرور از دستورات داکر استفاده نماید کافی است یکی از کاربرهایی که توانایی اجرای دستورات `sudo` دارد، دستور زیر را اجرا نماید. در این صورت کاربر جدید بدون نیازی به دسترسی `sudo` در سرور، میتواند دستورات داکر را براحتی اجرا نماید.
```sh
sudo usermod -aG docker <new-user-name>
```

## نصب docker بصورت ساده تر
این روش از اسگریپ نصب که بصورت رسمی در وبسایت داکر قرار داده شده استفاده میکند:
```sh
sudo curl https://get.docker.com | sh
```

توجه: درصورت بروز خطا در اجرای دستورات به احتمال زیاد مربوط به تحریم از سوی شرکت داکر بوده که با تنظیم سرور روی تحریم شکن هایی مانند shecan این مشکل برطرف خواهد شد.

## نصب docker-compose
برای نصب docker-compose از دستورات زیر استفاده میکنیم:

```sh
sudo curl -L "https://github.com/docker/compose/releases/download/v2.16.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

اگر پیام زیر را دریافت کردید پس این کار را با موفقیت انجام داده اید:
```sh
Docker Compose version v2.16.0
```

در زمان نگارش این مقاله آخرین نسخه docker-compose برابر v2.16.0 میباشد.
جهت نصب آخرین نسخه میتوانید به صفحه [compose releases in github](https://github.com/docker/compose/releases) رجوع کنید و شماره نسخه مورد نظر خود را در لینک بالا جایگذاری نمایید.

## رفع مشکل اشتباه بودن Primary Group برای کاربران Active Directory
```sh
$ id
uid=[user] gid=[invalid-group] groups=xxx(domain users@domain.local),[other-groups]

# fix
$ sudo realm permit -g "domain users@domain.local" $USER

$ id
uid=[user] gid=xxx(domain users@domain.local) groups=[other-groups]
```
