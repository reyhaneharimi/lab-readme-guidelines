.. _cluster-requirements:

نیازمندی‌های Cluster
====================

محیط local مربوط به Educates بهترین روش برای شروع کار و توسعه Workshopهای جدید است.
اما زمانی که قصد دارید Workshopها را برای استفاده دیگران منتشر کنید،
باید یک Kubernetes cluster جداگانه برای اجرای Educates و Workshopها آماده نمایید.

در محیط local، تنظیمات Kind cluster به صورت خودکار انجام می‌شود.
اما در یک Kubernetes cluster مستقل ممکن است لازم باشد برخی پیش‌نیازها را خودتان تنظیم کنید.

--------------------------------------------------------------------

.. _dedicated-kubernetes-cluster:

Cluster اختصاصی Kubernetes
---------------------------

به دلیل اینکه کاربران Workshop ممکن است کاربران غیرقابل اعتماد باشند،
به‌شدت توصیه می‌شود Educates در یک Kubernetes cluster اختصاصی اجرا شود.

از Deploy کردن Educates در clusterی که برای توسعه یا production استفاده می‌کنید خودداری کنید.

اگرچه Educates از RBAC، quota، limit range و security policy engine استفاده می‌کند،
همیشه این ریسک وجود دارد که یک کاربر Workshop
تأثیری روی کل cluster یا سایر workloadها بگذارد.

--------------------------------------------------------------------

.. _size-of-kubernetes-cluster:

اندازه Kubernetes cluster
--------------------------

پاسخ قطعی برای اندازه cluster وجود ندارد،
زیرا به موارد زیر بستگی دارد:

- نوع Workshopها
- نیازمندی‌های آن‌ها
- تعداد کاربران همزمان

برای شروع پیشنهاد می‌شود:

- ۳ worker node
- هر worker دارای 16 تا 32 گیگابایت RAM

از nodeهایی با بیش از 32GB RAM با احتیاط استفاده کنید،
زیرا برخی providerها محدودیت در تعداد persistent volume دارند.

بهتر است به جای nodeهای بزرگ،
load را روی nodeهای بیشتر توزیع کنید.

--------------------------------------------------------------------

.. _kubernetes-ingress-controller:

Ingress controller
------------------

Cluster باید دارای یک ingress controller باشد.

الزامات:

- استفاده از پورت‌های استاندارد 80 و 443
- نیازی به edge termination سراسری نیست
- Educates خود ingress resource و secret مربوطه را ایجاد می‌کند

به‌شدت توصیه می‌شود از Contour استفاده شود
به جای nginx ingress controller.

دلیل:

nginx هنگام ایجاد یا حذف ingress،
پیکربندی خود را reload می‌کند
که باعث قطع شدن websocket connection می‌شود.

Educates reconnect را مدیریت می‌کند،
اما همه applicationها این رفتار را ندارند.

اگر از providerهایی مانند AWS، GCP یا Azure استفاده می‌کنید،
timeout مربوط به load balancer را افزایش دهید
(پیشنهاد: 1 ساعت برای websocket).

اگر کاربران زیادی از Safari استفاده می‌کنند،
به دلیل مشکل HTTP/2 coalescing
ممکن است لازم باشد HTTP/2 را در ingress controller غیرفعال کنید.

--------------------------------------------------------------------

.. _kubernetes-persistent-volumes:

Persistent Volumeها
-------------------

Educates به persistent volume از نوع ``ReadWriteOnce (RWO)`` نیاز دارد.

باید یک default storage class در cluster تعریف شده باشد.

اگر storage مبتنی بر NFS باشد
ممکن است لازم باشد تنظیم user/group برای permission انجام شود.

توجه داشته باشید پشتیبانی از NFS به صورت کامل و مداوم تست نمی‌شود،
در صورت بروز مشکل گزارش دهید.

--------------------------------------------------------------------

.. _cluster-security-enforcement:

اجرای سیاست امنیتی در سطح Cluster
-----------------------------------

RBAC فقط نوع resource و عملیات مجاز را کنترل می‌کند.
اما برخی تنظیمات مانند:

- اجرای container به صورت root
- Linux capability
- privileged container

تحت RBAC قرار نمی‌گیرند.

مکانیزم‌های موجود:

- Pod Security Policies (<= 1.25)
- Pod Security Standards (>= 1.22)
- Security Context Constraints (OpenShift)

توصیه برای Kubernetes استاندارد:

- استفاده نکردن از Pod Security Policies
- استفاده نکردن از Pod Security Standards
- استفاده از Kyverno

Kyverno باید نصب شده باشد.
Educates policyها را برای آن تعریف می‌کند.

اگر هیچ policy engine فعال نباشد،
Workshopها نباید برای کاربران غیرقابل اعتماد در دسترس باشند.

در OpenShift باید از Security Context Constraints استفاده شود.

برای امنیت بیشتر می‌توان از Kata Containers و runtimeClass استفاده کرد.

--------------------------------------------------------------------

.. _workshop-security-enforcement:

اجرای سیاست امنیتی در سطح Workshop
------------------------------------

مکانیزم‌های cluster فقط محدودیت‌های پایه را اعمال می‌کنند.

برای جلوگیری از تداخل کاربران Workshop،
از Kyverno استفاده می‌شود.

حتی اگر برای cluster از Kyverno استفاده نکنید،
برای enforcement سطح Workshop به آن نیاز دارید.

در صورت نبود Kyverno،
نباید دسترسی Workshop به کاربران غیرقابل اعتماد داده شود.

--------------------------------------------------------------------

.. _carvel-package-installation:

نصب با Carvel
-------------

نصب Educates بر اساس سیستم packaging مربوط به Carvel انجام می‌شود.

دو روش نصب وجود دارد:

روش اول:
استفاده از CLI مربوط به Educates

در این حالت نیازی به نصب ابزار Carvel یا kapp-controller روی سیستم local نیست.

روش دوم:
نصب kapp-controller داخل cluster
و استفاده از آن برای نصب Educates

در Tanzu Kubernetes Grid یا Tanzu Mission Control،
kapp-controller به صورت پیش‌فرض نصب است.

در سایر distributionها باید آن را دستی نصب کنید.
