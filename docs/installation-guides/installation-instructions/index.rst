.. _installation-instructions:

دستورالعمل نصب
===============

دستورالعمل‌های این بخش فقط زمانی لازم هستند که بخواهید Educates را در یک Kubernetes cluster اختصاصی نصب کنید و از local Educates environment استفاده نکنید.

قبل از شروع، حتماً مستندات مربوط به cluster requirements را مطالعه کنید.

--------------------------------------------------------------------

.. _cli-vs-kapp-controller:

CLI در مقابل kapp-controller
-----------------------------

برای نصب Educates روی یک Kubernetes cluster موجود، دو گزینه دارید.

گزینه اول استفاده از ``educates`` CLI است.
این روش یک راهکار self-contained است و نیاز به نصب operator خاصی داخل Kubernetes cluster ندارد و همچنین به ابزار third-party خاصی روی ماشین نصب‌کننده نیاز ندارد، به جز خود CLI.

گزینه دوم استفاده از ``kapp-controller`` از پروژه Carvel است که باید از قبل داخل Kubernetes cluster نصب شده باشد.
در این روش تنها نیاز است که ``kubectl`` روی سیستم نصب‌کننده موجود باشد.

استفاده از ``educates`` CLI معمولاً تجربه ساده‌تری برای نصب فراهم می‌کند،
اما استفاده از Carvel packages همراه با ``kapp-controller`` ممکن است در سناریوهای GitOps مناسب‌تر باشد.

--------------------------------------------------------------------

.. _opinionated-cluster-install:

نصب Opinionated روی cluster
----------------------------

چه از CLI استفاده کنید و چه از ``kapp-controller``،
مکانیزم نصب Educates از یک configuration opinionated پشتیبانی می‌کند.

به این معنی که می‌توانید فقط infrastructure provider مربوط به Kubernetes cluster را مشخص کنید
و Educates به صورت خودکار از یک configuration آماده (pre-canned configuration) برای آن provider استفاده خواهد کرد.

در این حالت علاوه بر نصب Educates training platform،
سرویس‌ها و Kubernetes operatorهای مورد نیاز نیز نصب می‌شوند.

Infrastructure providerهای پشتیبانی‌شده:

- ``eks`` — Amazon Elastic Kubernetes Service
- ``gke`` — Google Kubernetes Engine
- ``kind`` — Kubernetes in Docker
- ``minikube`` — Minikube
- ``openshift`` — OpenShift
- ``vcluster`` — Virtual Kubernetes Cluster

اگر provider شما در لیست نیست و یک Kubernetes cluster عمومی با ingress controller از قبل نصب‌شده دارید،
می‌توانید از provider با نام ``generic`` استفاده کنید.

اگر بخواهید configuration را کاملاً از ابتدا خودتان تعریف کنید،
باید از provider با نام ``custom`` استفاده کنید
و تمام تنظیمات مورد نیاز را به صورت کامل مشخص نمایید.

--------------------------------------------------------------------

.. _additional-installed-services:

سرویس‌های نصب‌شده اضافی
------------------------

هنگام نصب Educates، فقط خود platform نصب نمی‌شود،
بلکه سرویس‌ها و Kubernetes operatorهای دیگری نیز که مورد نیاز هستند یا مفیدند نصب می‌شوند.

سرویس‌هایی که می‌توانند به صورت خودکار نصب شوند:

- ``cert-manager`` — مدیریت Certificate برای Kubernetes
- ``contour`` — Ingress controller
- ``external-dns`` — مدیریت DNS خارجی
- ``kapp-controller`` — operator نصب packageهای Carvel
- ``kyverno`` — engine اعمال policy امنیتی

معمولاً Kyverno همیشه نصب می‌شود زیرا برای اعمال policyهای امنیتی مورد استفاده قرار می‌گیرد.

در برخی infrastructure providerها سرویس‌های اضافی دیگری نیز به صورت خودکار نصب می‌شوند.

--------------------------------------------------------------------

.. _package-configuration-file:

فایل configuration package
--------------------------

برای نصب Educates باید یک فایل configuration با فرمت YAML ارائه شود.

حداقل configuration مورد نیاز بسته به infrastructure provider متفاوت است.

نمونه حداقل configuration برای یک Kubernetes cluster ساخته‌شده با Kind:

.. raw:: html

   <div dir="ltr">

::

  clusterInfrastructure:
    provider: kind

  clusterIngress:
    domain: educates-local-dev.test

.. raw:: html

   </div>

در این مثال:

- ``clusterInfrastructure.provider`` مشخص می‌کند Educates روی چه providerای نصب می‌شود.
- ``clusterIngress.domain`` دامنه‌ای است که Educates زیر آن در دسترس خواهد بود.

در صورت ارائه تنظیمات اضافی، این مقادیر جایگزین defaultهای global یا provider-specific خواهند شد.

برای جزئیات بیشتر درباره تنظیمات قابل سفارشی‌سازی،
مستندات مربوط به Configuration Settings را مطالعه کنید.

همچنین برای مشاهده نیازمندی‌های خاص هر provider،
به مستندات Infrastructure Providers مراجعه کنید.

--------------------------------------------------------------------

.. _performing-the-installation:

اجرای فرآیند نصب
-----------------

برای انجام نصب، بسته به روشی که انتخاب کرده‌اید به مستندات مربوطه مراجعه کنید:

- CLI Based Installation — نصب با استفاده از ``educates`` CLI
- Carvel Based Installation — نصب با استفاده از ``kapp-controller``

هر دو روش مربوط به نصب روی Kubernetes cluster موجود هستند.

اگر برای اولین بار Educates را امتحان می‌کنید،
توصیه می‌شود از cluster موجود استفاده نکنید،
بلکه از CLI برای ایجاد یک local environment استفاده کنید.

راهنماهای مرتبط:

- Quick Start Guide
- Local Environment
