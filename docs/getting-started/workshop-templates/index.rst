.. _workshop-templates:

قالب‌های ورکشاپ
================

قالب Workshop در Educates یک نقطه شروع برای ایجاد Workshopهای جدید فراهم می‌کند.
این قالب برای کار با محیط محلی Educates طراحی شده است، اما می‌تواند برای هر نوع Deployment سفارشی‌سازی شود.
هنگام ایجاد یک Workshop جدید می‌توان گزینه‌هایی برای تنظیم جزئیات آن مشخص کرد.

--------------------------------------------------------------------

.. _customizing-workshop-details:

سفارشی‌سازی جزئیات ورکشاپ
--------------------------

برای ایجاد یک Workshop جدید با استفاده از CLI، دستور زیر را اجرا کنید:

.. raw:: html

   <div dir="ltr">

::

  educates new-workshop lab-new-workshop

.. raw:: html

   </div>

آخرین بخش مسیر وارد شده به عنوان نام Workshop استفاده می‌شود.

نام Workshop باید مطابق استاندارد RFC 1035 برای Kubernetes object name باشد.
اگرچه Kubernetes حداکثر 63 کاراکتر را مجاز می‌داند، توصیه می‌شود طول نام بیش از 25 کاراکتر نباشد،
زیرا Educates در شرایط مختلف ممکن است prefix یا suffix اضافه کند.

به صورت پیش‌فرض، دستور ``educates new-workshop`` فایل‌ها را برای استفاده از renderer مبتنی بر ``hugo`` ایجاد می‌کند.

برای استفاده از renderer قدیمی ``classic``:

.. raw:: html

   <div dir="ltr">

::

  educates new-workshop lab-new-workshop --template classic

.. raw:: html

   </div>

renderer نوع ``classic`` منسوخ شده و در آینده حذف خواهد شد.
بنابراین توصیه می‌شود از ``hugo`` استفاده شود.

گزینه‌های CLI برای سفارشی‌سازی:

- ``--title`` : عنوان کوتاه Workshop
- ``--description`` : توضیح کامل‌تر Workshop
- ``--image`` : تعیین base image جایگزین

Base imageهای ارائه‌شده با Educates:

- jdk8-environment:*
- jdk11-environment:*
- jdk17-environment:*
- jdk21-environment:*
- conda-environment:*

--------------------------------------------------------------------

.. _custom-workshop-base-image:

ساخت image پایه سفارشی
-----------------------

قالب پیش‌فرض از یک OCI image artifact برای بسته‌بندی فایل‌های محتوای Workshop استفاده می‌کند.
این artifact روی base image استاندارد یا یکی از imageهای جایگزین ارائه‌شده قرار می‌گیرد.

نمونه پیکربندی در فایل ``resources/workshop.yaml``:

.. raw:: html

   <div dir="ltr">

::

  spec:
    workshop:
      files:
      - image:
          url: $(image_repository)/{name}-files:$(workshop_version)
        includePaths:
        - /workshop/**
        - /exercises/**
        - /README.md

.. raw:: html

   </div>

در این مثال مقدار ``{name}`` همان ``metadata.name`` است.

متغیر ``$(image_repository)`` باعث می‌شود OCI image artifact از registry محلی Kubernetes دریافت شود.
نیازی به مشخص کردن صریح host نیست، زیرا Educates مقدار مناسب را جایگزین می‌کند.

متغیر ``$(workshop_version)`` هنگام توسعه با مقدار ``latest`` جایگزین می‌شود
و هنگام Publish با مقدار واقعی نسخه جایگزین خواهد شد.

اگر بخواهید از image سفارشی استفاده کنید:

.. raw:: html

   <div dir="ltr">

::

  spec:
    workshop:
      image: custom-environment:latest
      files:
      - image:
          url: $(image_repository)/{name}-files:$(workshop_version)
        includePaths:
        - /workshop/**
        - /exercises/**
        - /README.md

.. raw:: html

   </div>

اگر image اختصاصی مربوط به همان Workshop باشد و جداگانه Publish نشود،
می‌توانید یک ``Dockerfile`` اضافه کنید و مقدار image را به صورت زیر تنظیم کنید:

.. raw:: html

   <div dir="ltr">

::

  spec:
    workshop:
      image: $(image_repository)/{name}-image:$(workshop_version)
      files:
      - image:
          url: $(image_repository)/{name}-files:$(workshop_version)
        includePaths:
        - /workshop/**
        - /exercises/**
        - /README.md

.. raw:: html

   </div>

برای Build و Push image سفارشی به registry محلی:

.. raw:: html

   <div dir="ltr">

::

  docker build -t localhost:5001/{name}-image:latest .

.. raw:: html

   </div>

image سفارشی سپس برای هر Workshop session از registry محلی دریافت خواهد شد.

توصیه می‌شود در صورت امکان از image سفارشی استفاده نشود و به جای آن از extension package استفاده شود.
با استفاده از base imageهای استاندارد اطمینان حاصل می‌کنید که نسخه image
با نسخه Educates سازگار است.

--------------------------------------------------------------------

.. _hosting-workshops-github:

میزبانی ورکشاپ در GitHub
-------------------------

برای Publish خودکار نسخه‌های tag شده، می‌توانید از GitHub Action استفاده کنید.

فایل ``.github/workflows/publish-workshop.yaml`` را به repository اضافه کنید
و محتوای زیر را در آن قرار دهید:

.. raw:: html

   <div dir="ltr">

::

  name: Publish Workshop
  on:
    push:
      tags:
        - "[0-9]+.[0-9]+"
        - "[0-9]+.[0-9]+-alpha.[0-9]+"
        - "[0-9]+.[0-9]+-beta.[0-9]+"
        - "[0-9]+.[0-9]+-rc.[0-9]+"
  jobs:
    publish-workshop:
      runs-on: ubuntu-latest
      steps:
        - name: Checkout code
          uses: actions/checkout@v3
        - name: Create release
          uses: educates/educates-github-actions/publish-workshop@v6
          with:
            token: ${{secrets.GITHUB_TOKEN}}

.. raw:: html

   </div>

پس از افزودن workflow، برای انتشار نسخه پایدار یک tag با فرمت ``X.Y`` ایجاد کنید،
مثلاً ``1.0`` و آن را Push کنید:

.. raw:: html

   <div dir="ltr">

::

  git tag 1.0
  git push origin 1.0

.. raw:: html

   </div>

با Push شدن tag اقدامات زیر انجام می‌شود:

- ایجاد OCI image artifact شامل فایل‌های محتوای Workshop
- Push به GitHub Container Registry
- ایجاد Release در GitHub
- افزودن فایل‌های Kubernetes resource به عنوان asset

این عملیات با استفاده از دستور ``educates publish-workshop`` انجام می‌شود.
در تعریف Workshop باید بخش Publish به شکل زیر تنظیم شده باشد:

.. raw:: html

   <div dir="ltr">

::

  spec:
    publish:
      image: $(image_repository)/{name}-files:$(workshop_version)

.. raw:: html

   </div>

اگر repository خصوصی باشد،
باید visibility imageهای منتشرشده در GitHub Container Registry را به public تغییر دهید.

برای Deploy از Release:

.. raw:: html

   <div dir="ltr">

::

  educates deploy-workshop -f https://github.com/educates/lab-k8s-fundamentals/releases/latest/download/workshop.yaml

.. raw:: html

   </div>
