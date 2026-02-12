.. _local-environment:

محیط توسعه محلی
================

برای توسعه محلی محتوای Workshop، ابزار CLI مربوط به Educates امکان ایجاد یک Kubernetes cluster محلی با استفاده از Kind را فراهم می‌کند. این محیط شامل یک image registry محلی برای ذخیره فایل‌های محتوای Workshop و imageهای پایه سفارشی است.

به صورت پیش‌فرض این محیط از دامنه ``nip.io`` برای دسترسی استفاده می‌کند، اما می‌توان آن را برای استفاده از یک ingress domain سفارشی و TLS certificate مناسب تنظیم کرد.

--------------------------------------------------------------------

.. _create-local-cluster:

ایجاد کلاستر
------------

برای ایجاد Kubernetes cluster محلی و Deploy کردن Educates:

.. raw:: html

   <div dir="ltr">

::

  educates create-cluster

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _delete-local-cluster:

حذف کلاستر
-----------

برای حذف کلاستر:

.. raw:: html

   <div dir="ltr">

::

  educates delete-cluster

.. raw:: html

   </div>

برای حذف کامل شامل registry و DNS:

.. raw:: html

   <div dir="ltr">

::

  educates delete-cluster --all

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _custom-local-config:

 ----------------

برای ویرایش تنظیمات پیش‌فرض YAML:

.. raw:: html

   <div dir="ltr">

::

  educates local config edit

.. raw:: html

   </div>

برای مشاهده مقادیر نهایی YAML:

.. raw:: html

   <div dir="ltr">

::

  educates local config view

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _local-image-registry:

--------------------

در هنگام اجرای دستور ایجاد کلاستر، یک image registry محلی نیز Deploy می‌شود که برای ذخیره فایل‌های Workshop و imageهای سفارشی استفاده می‌شود.

برای Push کردن image به registry محلی باید از آدرس زیر استفاده کنید:

.. raw:: html

   <div dir="ltr">

::

  localhost:5001

.. raw:: html

   </div>

اگر در داخل Kubernetes cluster نیاز به Pull image داشتید، باید از آدرس زیر استفاده شود:

.. raw:: html

   <div dir="ltr">

::

  registry.default.svc.cluster.local

.. raw:: html

   </div>

در داخل تعریف Workshop نیز استفاده از متغیر ``$(image_repository)`` توصیه می‌شود که به صورت خودکار به همین registry اشاره می‌کند.

برای حذف registry:

.. raw:: html

   <div dir="ltr">

::

  educates local registry delete

.. raw:: html

   </div>

برای Deploy مجدد:

.. raw:: html

   <div dir="ltr">

::

  educates local registry deploy

.. raw:: html

   </div>

برای پاکسازی لایه‌های بدون مرجع:

.. raw:: html

   <div dir="ltr">

::

  educates local registry prune

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _custom-ingress-domain:

دامنه Ingress سفارشی
---------------------

برای استفاده از دامنه سفارشی هنگام ایجاد کلاستر:

.. raw:: html

   <div dir="ltr">

::

  educates create-cluster --domain educates-local-dev.test

.. raw:: html

   </div>

یا از طریق تنظیمات YAML:

.. raw:: html

   <div dir="ltr">

::

  clusterIngress:
    domain: educates-local-dev.test

.. raw:: html

   </div>

برای اضافه کردن TLS certificate:

.. raw:: html

   <div dir="ltr">

::

  educates local secrets add tls ${INGRESS_DOMAIN}-tls \
   --cert $HOME/.letsencrypt/config/live/${INGRESS_DOMAIN}/fullchain.pem \
   --key $HOME/.letsencrypt/config/live/${INGRESS_DOMAIN}/privkey.pem \
   --domain ${INGRESS_DOMAIN}

.. raw:: html

   </div>

برای اضافه کردن CA:

.. raw:: html

   <div dir="ltr">

::

  educates local secrets add ca ${INGRESS_DOMAIN}-ca \
   --cert "`mkcert -CAROOT`/rootCA.pem" \
   --domain ${INGRESS_DOMAIN}

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _local-dns-resolver:

DNS محلی کلاستر
---------------

برای Deploy resolver در macOS:

.. raw:: html

   <div dir="ltr">

::

  educates local resolver deploy

.. raw:: html

   </div>

ساخت پوشه resolver:

.. raw:: html

   <div dir="ltr">

::

  sudo mkdir /etc/resolver

.. raw:: html

   </div>

ساخت فایل resolver:

.. raw:: html

   <div dir="ltr">

::

  sudo sh -c "cat > /etc/resolver/${INGRESS_DOMAIN} << EOF
  nameserver 127.0.0.1
  EOF"

.. raw:: html

   </div>

برای حذف resolver:

.. raw:: html

   <div dir="ltr">

::

  educates local resolver delete

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _local-image-mirrors:

-----------------

برای استفاده از mirror هنگام ایجاد cluster:

.. raw:: html

   <div dir="ltr">

::

  localKindCluster:
    listenAddress: 0.0.0.0
    registryMirrors:
      - mirror: ghcr.io
      - mirror: docker.io
        url: registry-1.docker.io

.. raw:: html

   </div>

سپس:

.. raw:: html

   <div dir="ltr">

::

  educates create-cluster

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _deploy-local-mirror:

-------------------

.. raw:: html

   <div dir="ltr">

::

  educates local mirror deploy ghcr.io

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _delete-local-mirror:

----------------

.. raw:: html

   <div dir="ltr">

::

  educates local mirror delete ghcr.io

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _custom-cidrs:

تنظیم CIDRهای کلاستر
--------------------

برای تغییر pod و service CIDR در تنظیمات:

.. raw:: html

   <div dir="ltr">

::

  localKindCluster:
    networking:
      podCIDR: 10.244.0.0/16
      serviceCIDR: 10.96.0.0/12

.. raw:: html

   </div>
