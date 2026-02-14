کار با workshop content
=======================

Workshop content می‌تواند به‌صورت یک OCI image artifact بسته‌بندی شود، در یک Git repository میزبانی شود، یا روی یک web server قرار گیرد و هنگام ایجاد workshop session دانلود شود. همچنین می‌تواند در یک custom workshop image نیز build شود.

برای سریع‌تر کردن چرخه تکرار ویرایش و تست هنگام توسعه workshop content، ابزار `educates` CLI دستورات مختلفی برای تسهیل توسعه در یک local environment فراهم می‌کند. هنگام استفاده از `educates` CLI بسیاری از جزئیات پیکربندی به‌صورت مخفی مدیریت می‌شوند.

این سند برخی از جزئیات استفاده از `educates` CLI را پوشش می‌دهد و همچنین تلاش می‌کند بخشی از configuration زیرساختی را توضیح دهد، در صورتی که نیاز باشد روی workshop content در یک hosted cluster کار کنید و امکان استفاده از `educates` CLI وجود نداشته باشد.

--------------------------------------------------------------------

انتشار workshop content
------------------------

هنگامی که با استفاده از `educates` CLI یک Kubernetes cluster محلی با Kind ایجاد می‌کنید، یک image registry نیز به‌صورت خودکار deploy شده و به cluster متصل می‌شود. این registry می‌تواند برای نگهداری workshopهای منتشرشده استفاده شود.

در این حالت، تعریف workshop باید به شکل زیر باشد:

```yaml
spec:
  publish:
    image: $(image_repository)/{name}-files:$(workshop_version)
  workshop:
    files:
    - image:
        url: $(image_repository)/{name}-files:$(workshop_version)
      includePaths:
      - /workshop/**
      - /exercises/**
      - /README.md
```

در اینجا `{name}` نام workshop است.

هنگام اجرای دستور:

```
educates publish-workshop
```

مقدار `$(image_repository)` در مقصد انتشار که در `publish.image` مشخص شده است با `localhost:5001` جایگزین می‌شود و `$(workshop_version)` با `latest` جایگزین خواهد شد.

اگر پس از آن اجرا کنید:

```
educates deploy-workshop
```

در زمان ارزیابی مقدار `workshop.files.image.url`، مقدار `$(image_repository)` با `registry.default.svc.cluster.local` جایگزین می‌شود که hostname داخلی registry هنگام دسترسی از داخل Kubernetes cluster است.

در صورت دسترسی به registry از داخل cluster با ابزارهایی مانند `imgpkg`، به دلیل داشتن آدرس `.local` دسترسی insecure مجاز خواهد بود. برخی ابزارهای دیگر ممکن است نیاز داشته باشند که دسترسی insecure به‌صورت دستی فعال شود. همین موضوع ممکن است هنگام دسترسی به registry از طریق `localhost:5001` نیز صادق باشد.

در صورت کار با یک remote Kubernetes cluster که به local registry دسترسی ندارد، می‌توانید از یک remote registry برای میزبانی workshopهای عمومی استفاده کنید.

برای انتشار به یک remote registry:

```
educates publish-workshop --image-repository docker.io/username
```

بسته به نوع registry، باید با `docker login` وارد شده باشید یا credentials را در خط فرمان به `educates publish-workshop` بدهید.

برای اینکه فرد دیگری بتواند workshop را deploy کند:

```
educates publish-workshop --image-repository docker.io/username --workshop-version 1.0 --export-workshop published-workshop.yaml
```

در این حالت، نسخه‌ای اصلاح‌شده از workshop definition در فایل `published-workshop.yaml` نوشته می‌شود که تمامی ارجاعات `$(image_repository)` با آدرس remote registry جایگزین شده‌اند.

اگر فقط بخواهید definition اصلاح‌شده را تولید کنید:

```
educates export-workshop --image-repository docker.io/username --workshop-version 1.0 > published-workshop.yaml
```

دریافت‌کننده می‌تواند با اجرای:

```
educates deploy-workshop -f published-workshop.yaml
```

یا با استفاده از `kubectl` آن را deploy کند.

--------------------------------------------------------------------

ساخت local workshop image
--------------------------

در صورت استفاده از Kubernetes cluster محلی ایجادشده توسط `educates` CLI، همان local image registry می‌تواند برای نگهداری custom workshop base image استفاده شود.

تعریف workshop:

```yaml
spec:
  workshop:
    image: $(image_repository)/{name}-image:$(workshop_version)
```

مقدار `$(image_repository)` و `$(workshop_version)` هنگام deploy به‌صورت خودکار جایگزین خواهند شد.

آدرس registry در سیستم محلی `localhost:5001` است. دسترسی insecure خواهد بود اما ابزارهایی مانند `docker` آن را مجاز می‌دانند.

از داخل cluster، hostname برابر `registry.default.svc.cluster.local` است و port پیش‌فرض 80 استفاده می‌شود. فرآیند `containerd` برای اعتماد به registry پیکربندی شده است.

--------------------------------------------------------------------

غیرفعال کردن reserved sessions
------------------------------

برای توسعه content باید reserved sessions غیرفعال شوند:

```yaml
apiVersion: training.educates.dev/v1beta1
kind: TrainingPortal
metadata:
  name: lab-sample-workshop
spec:
  portal:
    sessions:
      maximum: 1
  workshops:
  - name: lab-sample-workshop
    reserved: 0
    expires: 120m
    orphaned: 15m
```

در غیر این صورت session رزرو شده ممکن است نسخه قدیمی content را داشته باشد.

--------------------------------------------------------------------

دسترسی به workshop error logs
------------------------------

در صورت بروز خطا هنگام دانلود content، اجرای setup script یا render دستورالعمل‌ها، یک dialog خطا نمایش داده می‌شود.

دو روش بررسی:

۱- استفاده از `kubectl logs`  
۲- بررسی فایل‌های لاگ در:

```
$HOME/.local/share/workshop
```

فایل‌های اصلی:

- `$HOME/.local/share/workshop/download-workshop.log`
- `$HOME/.local/share/workshop/setup-scripts.log`

--------------------------------------------------------------------

به‌روزرسانی زنده content
------------------------

برای به‌روزرسانی بدون restart session:

```
update-workshop
```

برای restart renderer در classic:

```
restart-workshop
```

برای hugo نیاز به restart نیست.

برای rebuild content در hugo:

```
rebuild-content
```

برای اجرای مجدد setup scripts:

```
rebuild-workshop
```

--------------------------------------------------------------------

تغییرات custom workshop image
-----------------------------

هنگام توسعه از tagهای زیر استفاده کنید:

```
main
master
develop
latest
```

نمونه definition:

```yaml
apiVersion: training.educates.dev/v1beta1
kind: Workshop
metadata:
  name: lab-sample-workshop
spec:
  title: Sample Workshop
  description: A sample workshop
  workshop:
    image: ghcr.io/educates/lab-sample-workshop:latest
```

--------------------------------------------------------------------

overlay برای custom workshop image
-----------------------------------

```yaml
apiVersion: training.educates.dev/v1beta1
kind: Workshop
metadata:
  name: lab-sample-workshop
spec:
  title: Sample Workshop
  description: A sample workshop
  workshop:
    image: ghcr.io/educates/lab-sample-workshop-image:latest
    files:
    - image:
        url: ghcr.io/educates/lab-sample-workshop-files:latest
```

--------------------------------------------------------------------

proxy به local workshop content
--------------------------------

برای سرو content محلی:

```
educates serve-workshop --patch-workshop
```

پیکربندی تزریق‌شده:

```yaml
spec:
  workshop:
    enabled: true
    proxy:
      changeOrigin: false
      headers:
      - name: X-Session-Name
        value: $(session_name)
      host: localhost.$(ingress_domain)
      port: 10081
      protocol: http
```

--------------------------------------------------------------------

تغییرات در workshop definition
------------------------------

```yaml
apiVersion: training.educates.dev/v1beta1
kind: TrainingPortal
metadata:
  name: lab-sample-workshop
spec:
  portal:
    sessions:
      maximum: 1
    updates:
      workshop: true
  workshops:
  - name: lab-sample-workshop
    expires: 120m
    orphaned: 15m
```

با فعال بودن این گزینه، در صورت تغییر definition، workshop environment جدید ایجاد خواهد شد.
