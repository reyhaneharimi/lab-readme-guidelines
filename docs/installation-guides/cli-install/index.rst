
.. _cli-based-installation:

نصب مبتنی بر CLI
=================

از میان دو روش موجود برای نصب Educates روی یک Kubernetes cluster موجود،
در این بخش نصب با استفاده از Educates CLI توضیح داده می‌شود.

فرض بر این است که شما قبلاً یک فایل configuration مناسب آماده کرده‌اید.

--------------------------------------------------------------------

.. _deploying-the-platform:

استقرار پلتفرم
---------------

پس از ایجاد فایل configuration مناسب، می‌توانید Educates را روی یک Kubernetes cluster موجود با اجرای دستور زیر نصب کنید:

.. raw:: html

   <div dir="ltr">

::

  educates deploy-platform --config config.yaml

.. raw:: html

   </div>

گزینه ``--config`` باید مسیر فایل configuration ایجادشده را مشخص کند.

در فایل configuration باید مقدار ``clusterInfrastructure.provider`` تنظیم شده باشد.

فرآیند نصب علاوه بر Educates، سرویس‌ها و Kubernetes operatorهای موردنیاز یا مفید برای provider مشخص‌شده را نیز نصب می‌کند.

اگر نیاز به debug فرآیند نصب دارید، می‌توانید از گزینه ``--verbose`` استفاده کنید:

.. raw:: html

   <div dir="ltr">

::

  educates deploy-platform --config config.yaml --verbose

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _kubeconfig-and-context:

Kubeconfig و Context
--------------------

به صورت پیش‌فرض، Educates CLI از فایل استاندارد ``kubeconfig`` استفاده می‌کند که معمولاً در مسیر زیر قرار دارد:

.. raw:: html

   <div dir="ltr">

::

  $HOME/.kube/config

.. raw:: html

   </div>

اگر بخواهید از فایل ``kubeconfig`` دیگری استفاده کنید، از گزینه ``--kubeconfig`` استفاده نمایید:

.. raw:: html

   <div dir="ltr">

::

  educates deploy-platform --config config.yaml --kubeconfig kubeconfig.yaml

.. raw:: html

   </div>

چه از kubeconfig پیش‌فرض استفاده شود و چه فایل جایگزین،
Context فعلی مشخص‌شده در آن استفاده خواهد شد.

اگر بخواهید Context متفاوتی مشخص کنید، از گزینه ``--context`` استفاده نمایید:

.. raw:: html

   <div dir="ltr">

::

  educates deploy-platform --config config.yaml --context educates-cluster

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _updating-configuration:

به‌روزرسانی configuration
--------------------------

پس از انجام نصب، در بسیاری از موارد می‌توان configuration را بدون حذف و نصب مجدد، به‌روزرسانی کرد.

برای این کار:

1. تغییرات لازم را در فایل configuration اعمال کنید.
2. همان دستور قبلی نصب را مجدداً اجرا نمایید.

مثال:

.. raw:: html

   <div dir="ltr">

::

  educates deploy-platform --config config.yaml

.. raw:: html

   </div>

توجه داشته باشید که این تغییرات لزوماً روی Training Portal یا Workshop Environmentهایی که قبلاً ایجاد شده‌اند تأثیر نخواهد گذاشت،
و فقط روی مواردی که بعد از آن ایجاد شوند اعمال خواهد شد.

--------------------------------------------------------------------

.. _deleting-the-installation:

حذف نصب
--------

برای حذف Educates و سرویس‌ها یا Kubernetes operatorهایی که در زمان نصب ایجاد شده‌اند، دستور زیر را اجرا کنید:

.. raw:: html

   <div dir="ltr">

::

  educates delete-platform

.. raw:: html

   </div>
