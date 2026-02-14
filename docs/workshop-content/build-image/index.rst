.. _building-an-image:

ساخت image
===========

اگرچه توصیه می‌شود از extension package‌ها برای bundle کردن applicationهای اضافی موردنیاز در workshop استفاده شود و آن‌ها هنگام شروع یک workshop session روی base image استاندارد overlay شوند، اما امکان ساخت custom workshop base image نیز وجود دارد.

از آنجا که این کار وابستگی مستقیم به نسخه مشخصی از workshop base image ایجاد می‌کند، ممکن است image ساخته‌شده با نسخه‌های جدیدتر Educates سازگار نباشد. بنابراین تنها در صورتی از این قابلیت استفاده کنید که گزینه دیگری وجود نداشته باشد.

--------------------------------------------------------------------

.. _structure-of-the-dockerfile:

ساختار Dockerfile
-----------------

ساختار Dockerfile برای ساخت یک custom workshop base image باید به شکل زیر آغاز شود:

::

  FROM ghcr.io/educates/educates-base-environment:3.1
  COPY --chown=1001:0 . /home/eduk8s/
  RUN mv /home/eduk8s/workshop /opt/workshop
  RUN fix-permissions /home/eduk8s

custom workshop image باید بر پایه image زیر ساخته شود:

``ghcr.io/educates/educates-base-environment``

اگر image شامل فایل‌های مربوط به یک workshop خاص نیز باشد، باید تمام فایل‌ها به دایرکتوری ``/home/eduk8s`` کپی شوند.

گزینه ``--chown=1001:0`` تضمین می‌کند مالک فایل‌ها user و group مناسب باشند.

سپس زیرشاخه ``workshop`` به ``/opt/workshop`` منتقل می‌شود تا مستقیماً برای کاربر قابل مشاهده نباشد. این مسیر یک location ویژه است که علاوه بر ``/home/eduk8s/workshop`` برای یافتن محتوای workshop بررسی می‌شود.

برای نادیده گرفتن فایل‌ها یا دایرکتوری‌های اضافی، آن‌ها را در فایل ``.dockerignore`` لیست کنید.

می‌توان از دستورهای ``RUN`` برای اجرای مراحل build سفارشی استفاده کرد، اما user پیش‌فرض image برابر با user ID ``1001`` خواهد بود و ``root`` نیست.

--------------------------------------------------------------------

.. _base-images-and-version-tags:

base image و version tagها
--------------------------

در مثال بالا از image زیر استفاده شده است:

``ghcr.io/educates/educates-base-environment:3.1``

باید version آن را با نسخه Educates مورد استفاده هماهنگ و به‌روز نگه دارید.

برای مشاهده versionهای موجود:

https://github.com/educates/educates-training-platform/pkgs/container/educates-base-environment

--------------------------------------------------------------------

.. _custom-workshop-base-images:

custom workshop base imageها
----------------------------

imageهای ``base-environment`` شامل runtimeهای Node.js و Python هستند.

اگر به runtime دیگری یا نسخه متفاوتی نیاز دارید، باید custom workshop base image مخصوص خود را بسازید که بر پایه ``base-environment`` باشد و runtimeهای اضافی موردنیاز را اضافه کند.

برای زبان Java، پروژه Educates imageهای آماده برای JDK 8، 11، 17 و 21 ارائه می‌دهد که شامل Gradle و Maven نیز هستند.

JDK 8:

``ghcr.io/educates/educates-jdk8-environment:3.1``

JDK 11:

``ghcr.io/educates/educates-jdk11-environment:3.1``

JDK 17:

``ghcr.io/educates/educates-jdk17-environment:3.1``

JDK 21:

``ghcr.io/educates/educates-jdk21-environment:3.1``

برای مشاهده tagهای موجود هر image به صفحه package مربوطه در GitHub مراجعه کنید.

همچنین برای Anaconda Python یا Jupyter:

``ghcr.io/educates/educates-conda-environment:3.1``

--------------------------------------------------------------------

.. _container-run-as-random-user-id:

اجرای container با user ID تصادفی
----------------------------------

به‌طور پیش‌فرض container به‌صورت user ``1001`` اجرا می‌شود.

در برخی توزیع‌های Kubernetes مانند OpenShift ممکن است policy امنیتی باعث شود container با user ID متفاوتی اجرا شود.

اگر workshop نیاز به نوشتن در ``/home/eduk8s`` داشته باشد و فایل‌ها متعلق به user دیگری باشند، ممکن است مشکل permission ایجاد شود.

برای حل این موضوع، اسکریپت ``fix-permissions`` در base image وجود دارد که در پایان Dockerfile اجرا می‌شود و permission گروه را برابر permission کاربر می‌کند.

--------------------------------------------------------------------

.. _installing-extra-system-packages:

نصب system packageهای اضافی
----------------------------

برای نصب system packageها باید از user ``root`` استفاده کنید:

::

  USER root
  RUN ... commands to install system packages
  USER 1001

فقط برای نصب packageها از root استفاده کنید. برای فایل‌های داخل ``/home/eduk8s`` از root استفاده نکنید.

همچنین هنگام اجرای command به‌عنوان root، مقدار ``HOME`` را به ``/root`` تغییر دهید:

::

  USER root
  RUN HOME=/root && \
      ... commands to install system packages
  USER 1001

در غیر این صورت ممکن است فایل‌های config با permission اشتباه ایجاد شوند.

--------------------------------------------------------------------

.. _installing-third-party-packages:

نصب third party packageها
--------------------------

اگر packageها را دستی دانلود و build می‌کنید، بهتر است این کار را با user پیش‌فرض (نه root) انجام دهید.

برای build در مسیر موقت ``/tmp`` کار کنید و در همان دستور RUN آن را حذف کنید.

اگر فقط یک binary نصب می‌کنید، آن را در:

``/home/eduk8s/bin``

قرار دهید. این مسیر در متغیر ``PATH`` تعریف شده است.

اگر نیاز به نصب یک directory کامل دارید، آن را در زیرشاخه‌ای از ``/opt`` نصب کنید و سپس متغیرهای ``PATH`` و ``LD_LIBRARY_PATH`` را در Dockerfile تنظیم کنید.

اگر در یک دستور ``RUN`` فایلی در ``/home/eduk8s`` ایجاد می‌کنید، بهتر است در همان دستور ``fix-permissions`` را نیز اجرا کنید تا از ایجاد layer اضافی جلوگیری شود.
