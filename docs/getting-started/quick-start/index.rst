.. _quick-start-guide:

راهنمای شروع سریع
==================

سریع‌ترین روش برای نصب و شروع کار با Educates این است که آن را روی سیستم محلی خود با استفاده از یک Kubernetes cluster که توسط Kind ایجاد می‌شود اجرا کنید.

برای ساده‌تر شدن این فرآیند، Educates یک ابزار Command Line به نام ``educates`` CLI ارائه می‌دهد که می‌توانید با آن:

- یک Kubernetes cluster ایجاد کنید
- Educates را Deploy کنید
- Workshopها را Deploy و مدیریت کنید

این محیط محلی همچنین بهترین گزینه برای توسعه محتوای Workshop شخصی شماست، زیرا شامل یک local image registry می‌باشد که برای نگهداری base imageهای سفارشی و Workshopهای منتشرشده استفاده می‌شود.

--------------------------------------------------------------------

.. _host-system-requirements:

نیازمندی‌های سیستم میزبان
--------------------------

برای Deploy کردن Educates روی سیستم محلی، موارد زیر لازم است:

- macOS یا Linux (در Windows باید از WSL استفاده شود)
- یک محیط فعال ``docker``
- حافظه و فضای دیسک کافی
- عدم اجرای Kubernetes cluster مبتنی بر Kind از قبل
- آزاد بودن پورت‌های 80 و 443
- در macOS آزاد بودن پورت 53
- آزاد بودن پورت 5001 برای local image registry

اگر از Docker Desktop استفاده می‌کنید:

- Allow the default Docker socket فعال باشد
- Allow privileged port mapping فعال باشد
- در برخی نسخه‌ها Use kernel networking for UDP تنظیم شود

اگر از Colima استفاده می‌کنید:

.. raw:: html

   <div dir="ltr">

::

  educates local config edit

  localKindCluster:
    listenAddress: 0.0.0.0

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _downloading-the-cli:

دانلود CLI
-----------

برای دانلود CLI به صفحه Releases مراجعه کنید:

https://github.com/educates/educates-training-platform/releases

پس از دانلود، فایل را executable کنید:

.. raw:: html

   <div dir="ltr">

::

  chmod +x educates

.. raw:: html

   </div>

دانلود مستقیم با curl:

.. raw:: html

   <div dir="ltr">

::

  curl -o educates -sL https://github.com/educates/educates-training-platform/releases/latest/download/educates-linux-amd64 && chmod +x educates

.. raw:: html

   </div>

دانلود به صورت OCI image:

.. raw:: html

   <div dir="ltr">

::

  imgpkg pull -i ghcr.io/educates/educates-client-programs:X.Y.Z -o educates-client-programs

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _default-ingress-domain:

دامنه پیش‌فرض Ingress
----------------------

Educates برای Kubernetes Ingress به یک FQDN معتبر نیاز دارد.

به صورت پیش‌فرض از ``nip.io`` استفاده می‌شود، مانند:

::

  192-168-1-10.nip.io

برخی قابلیت‌ها نیاز به wildcard TLS certificate دارند که برای ``nip.io`` قابل تولید با LetsEncrypt نیست.

--------------------------------------------------------------------

.. _local-kubernetes-cluster:

ایجاد Kubernetes cluster محلی
------------------------------

برای ایجاد cluster:

.. raw:: html

   <div dir="ltr">

::

  educates create-cluster

.. raw:: html

   </div>

این دستور:

- Kubernetes cluster را ایجاد می‌کند
- Contour ingress controller را نصب می‌کند
- local image registry را راه‌اندازی می‌کند
- Educates را Deploy می‌کند

نام context ایجاد شده:

::

  kind-educates

--------------------------------------------------------------------

.. _deploying-a-workshop:

Deploy کردن یک Workshop
------------------------

.. raw:: html

   <div dir="ltr">

::

  educates deploy-workshop -f https://github.com/educates/lab-k8s-fundamentals/releases/latest/download/workshop.yaml

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _accessing-the-workshop:

دسترسی به Workshop
-------------------

.. raw:: html

   <div dir="ltr">

::

  educates browse-workshops

.. raw:: html

   </div>

برای مشاهده URL:

.. raw:: html

   <div dir="ltr">

::

  educates list-portals

.. raw:: html

   </div>

برای مشاهده password:

.. raw:: html

   <div dir="ltr">

::

  educates view-credentials

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _deleting-the-workshop:

حذف Workshop
------------

.. raw:: html

   <div dir="ltr">

::

  educates delete-workshop -f https://github.com/educates/lab-k8s-fundamentals/releases/latest/download/workshop.yaml

.. raw:: html

   </div>

یا با نام:

.. raw:: html

   <div dir="ltr">

::

  educates delete-workshop -n workshop-name

.. raw:: html

   </div>
