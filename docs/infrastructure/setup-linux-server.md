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
