.. _workshop-configuration:

پیکربندی ورکشاپ
================

یک Workshop از دو بخش اصلی تشکیل شده است:

1. تعریف Workshop (Workshop definition) که الزامات راه‌اندازی و نحوه پیکربندی محیط Educates را مشخص می‌کند.
2. فایل‌های Workshop شامل دستورالعمل‌ها، فایل‌های setup و فایل‌های تمرین.

--------------------------------------------------------------------

.. _workshop-setup-requirements:

الزامات راه‌اندازی ورکشاپ
--------------------------

Workshop imageها می‌توانند مستقیماً در یک container runtime اجرا شوند.
برای مدیریت استقرار در Kubernetes cluster، از Educates operator استفاده می‌شود.

پیکربندی Operator از طریق Custom Resource با نوع ``Workshop`` انجام می‌شود.
این تعریف در فایل ``resources/workshop.yaml`` قرار دارد.

نمونه تعریف:

.. raw:: html

   <div dir="ltr">

::

  apiVersion: training.educates.dev/v1beta1
  kind: Workshop
  metadata:
    name: lab-markdown-sample
  spec:
    title: Markdown Sample
    description: A sample workshop using Markdown
    workshop:
      files:
      - git:
          url: https://github.com/educates/lab-markdown-sample
          ref: origin/main
        includePaths:
        - /workshop/**
        - /exercises/**
        - /README.md
    session:
      namespaces:
        budget: small
      applications:
        console:
          enabled: true
        editor:
          enabled: true

.. raw:: html

   </div>

در این مثال فایل‌های Workshop از یک Git repository دریافت شده و روی base image استاندارد overlay می‌شوند.

برای استفاده از base image شامل Java JDK 17:

.. raw:: html

   <div dir="ltr">

::

  workshop:
    image: jdk17-environment:*

.. raw:: html

   </div>

همچنین می‌توان image سفارشی تعیین کرد:

.. raw:: html

   <div dir="ltr">

::

  workshop:
    image: ghcr.io/educates/lab-markdown-sample:latest

.. raw:: html

   </div>

تعریف ``Workshop`` علاوه بر image و فایل‌ها، شامل تنظیمات session، فعال‌سازی editor یا console،
تخصیص منابع، quota و سایر تنظیمات محیطی نیز می‌باشد.

--------------------------------------------------------------------

.. _instructions-rendering-options:

گزینه‌های رندر دستورالعمل‌ها
-----------------------------

Educates از دو نوع renderer برای دستورالعمل‌های Workshop پشتیبانی می‌کند:

- ``classic`` renderer
- ``hugo`` renderer

renderer نوع ``classic`` یک برنامه وب پویا برای نمایش Markdown یا AsciiDoc است.

renderer نوع ``hugo`` با استفاده از Hugo فایل‌های Markdown را به HTML ایستا تبدیل می‌کند.

در هر دو حالت فایل‌های محتوا در مسیر ``workshop/content`` قرار می‌گیرند.

--------------------------------------------------------------------

.. _classic-renderer-configuration:

پیکربندی classic renderer
--------------------------

در حالت ``classic`` ساختار محتوا با فایل‌های YAML تعریف می‌شود.

فایل ``workshop/modules.yaml`` لیست تمام moduleهای موجود را مشخص می‌کند:

.. raw:: html

   <div dir="ltr">

::

  modules:
    00-workshop-overview:
      name: Workshop Overview
      exit_sign: Start Workshop
    01-workshop-instructions:
      name: Workshop Instructions
    99-workshop-summary:
      name: Workshop Summary
      exit_sign: Finish Workshop

.. raw:: html

   </div>

فایل ``workshop/workshop.yaml`` تعیین می‌کند کدام moduleها فعال باشند:

.. raw:: html

   <div dir="ltr">

::

  name: Workshop
  modules:
    activate:
    - 00-workshop-overview
    - 01-workshop-instructions
    - 99-workshop-summary

.. raw:: html

   </div>

ترتیب نمایش صفحات بر اساس ترتیب ``modules.activate`` تعیین می‌شود.

دکمه Continue در پایین صفحه نمایش داده می‌شود.
متن آن با فیلد ``exit_sign`` قابل تنظیم است.

برای تغییر مقصد صفحه پایانی:

- استفاده از ``exit_link``
- تنظیم متغیر محیطی ``RESTART_URL``

در صورت استفاده از Training Portal، مقصد پایانی به صورت خودکار override می‌شود.

توصیه می‌شود در صفحه پایانی فقط ``exit_sign`` تنظیم شود و ``exit_link`` مشخص نشود.

--------------------------------------------------------------------

.. _hugo-renderer-configuration:

پیکربندی hugo renderer
----------------------

در حالت ``hugo`` مشخص کردن مسیر navigation به صورت جداگانه اختیاری است.

به طور پیش‌فرض همه صفحات موجود در ``workshop/content`` در navigation قرار می‌گیرند
و ترتیب آن‌ها بر اساس نام فایل محاسبه می‌شود.

می‌توان از metadata هر صفحه برای تعیین ترتیب با استفاده از ``weight`` استفاده کرد.

در صورت نیاز به کنترل کامل navigation می‌توان فایل ``workshop/config.yaml`` ایجاد کرد:

.. raw:: html

   <div dir="ltr">

::

  pathways:
    default: workshop
    paths:
      workshop:
        title: "Workshop"
        steps:
        - 00-workshop-overview
        - 01-workshop-instructions
        - 99-workshop-summary
  modules:
  - name: 00-workshop-overview
    title: Workshop Overview
  - name: 01-workshop-instructions
    title: Workshop Instructions
  - name: 99-workshop-summary
    title: Workshop Summary

.. raw:: html

   </div>

بخش ``modules`` در حالت hugo اختیاری است.

برای تعریف چند pathway می‌توان چند بخش در ``pathways.paths`` ایجاد کرد.

برای انتخاب pathway خاص هنگام اجرای Workshop، متغیر محیطی ``PATHWAY_NAME`` را تنظیم کنید.

در حالت ``hugo`` امکان تغییر متن دکمه navigation وجود ندارد،
برخلاف ``classic`` renderer.
