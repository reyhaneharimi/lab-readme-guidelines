.. _carvel-based-installation:

نصب مبتنی بر Carvel
====================

از میان دو روش موجود برای نصب Educates روی یک Kubernetes cluster موجود،
در این بخش نصب از طریق سیستم بسته‌بندی Carvel و operator با نام ``kapp-controller`` توضیح داده می‌شود.

فرض بر این است که شما قبلاً یک فایل configuration مناسب آماده کرده‌اید.

--------------------------------------------------------------------

.. _carvel-command-line-tools:

ابزارهای command line مربوط به Carvel
--------------------------------------

پروژه Carvel مجموعه‌ای از ابزارهای command line ارائه می‌دهد که می‌توان آن‌ها را به صورت local اجرا کرد،
همچنین تعدادی operator برای نصب در Kubernetes cluster جهت مدیریت package و secret ارائه می‌کند.

برای نصب Educates الزاماً نیازی به نصب ابزارهای Carvel به صورت local ندارید،
اما برای آشنایی بیشتر می‌توانید به وب‌سایت رسمی Carvel مراجعه کنید:

https://carvel.dev/

--------------------------------------------------------------------

.. _installing-kapp-controller:

نصب kapp-controller
--------------------

برای نصب Educates با استفاده از سیستم بسته‌بندی Carvel،
باید ``kapp-controller`` در Kubernetes cluster نصب شده باشد.

اگر از Kubernetes cluster ایجاد شده با Tanzu Kubernetes Grid (TKG)
یا Tanzu Mission Control (TMC) استفاده می‌کنید،
``kapp-controller`` به صورت پیش‌فرض نصب شده است.

در غیر این صورت می‌توانید آن را از طریق لینک زیر نصب کنید:

https://carvel.dev/kapp-controller/docs/develop/install/

در اکثر موارد کافی است دستور زیر را اجرا کنید:

.. raw:: html

   <div dir="ltr">

::

  kubectl apply -f https://github.com/vmware-tanzu/carvel-kapp-controller/releases/latest/download/release.yml

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _installer-service-account:

Service Account نصب‌کننده
--------------------------

هنگام استفاده از ``kapp-controller`` برای نصب یک package،
باید یک Service Account در Kubernetes cluster تعریف شود
که دسترسی لازم برای ایجاد منابع مورد نیاز را داشته باشد.

از آنجا که Educates ممکن است برای اجرای برخی Workshopها
نیاز به ایجاد انواع مختلف Kubernetes resource داشته باشد،
باید دسترسی کامل ``cluster-admin`` داشته باشد.

برای ایجاد Service Account و Role Binding مورد نیاز،
فایل YAML مربوطه در هر Release از Educates ارائه شده است.

برای اعمال آن در آخرین نسخه:

.. raw:: html

   <div dir="ltr">

::

  kubectl apply -f https://github.com/educates/educates-training-platform/releases/latest/download/educates-installer-app-rbac.yaml

.. raw:: html

   </div>

یا می‌توانید از صفحه Releases نسخه مشخصی را انتخاب کنید:

https://github.com/educates/educates-training-platform/releases

Namespace با نام ``educates-installer`` ایجاد خواهد شد
که Service Account در آن قرار می‌گیرد.

--------------------------------------------------------------------

.. _applying-package-values:

اعمال مقادیر configuration
---------------------------

در این مرحله باید یک Secret در Kubernetes cluster ایجاد شود
که configuration مربوط به Deploy کردن Educates را نگهداری کند.

فرض بر این است که configuration در فایل ``config.yaml`` قرار دارد:

.. raw:: html

   <div dir="ltr">

::

  kubectl create secret generic educates-installer \
    -n educates-installer \
    --from-file config.yaml \
    --save-config

.. raw:: html

   </div>

Secret باید در Namespace با نام ``educates-installer`` ایجاد شود.

--------------------------------------------------------------------

.. _installing-educates-package:

نصب package مربوط به Educates
------------------------------

اکنون آماده نصب Educates و سرویس‌های مورد نیاز بر اساس configuration هستید.

برای نصب آخرین نسخه:

.. raw:: html

   <div dir="ltr">

::

  kubectl apply -f https://github.com/educates/educates-training-platform/releases/latest/download/educates-installer-app.yaml

.. raw:: html

   </div>

یا می‌توانید نسخه مشخصی را از صفحه Releases دریافت و استفاده کنید.

در این مرحله نیز از Namespace با نام ``educates-installer`` استفاده خواهد شد.

--------------------------------------------------------------------

.. _updating-package-configuration:

به‌روزرسانی configuration package
----------------------------------

برای به‌روزرسانی configuration نصب‌شده،
مقدار Secret با نام ``educates-installer`` را به‌روزرسانی کنید:

.. raw:: html

   <div dir="ltr">

::

  kubectl create secret generic educates-installer \
    -n educates-installer \
    --from-file config.yaml \
    --dry-run=client -o yaml | kubectl apply -f -

.. raw:: html

   </div>

در اجرای بعدی reconcile توسط ``kapp-controller``,
configuration جدید اعمال خواهد شد.

برای اجرای دستی reconcile:

.. raw:: html

   <div dir="ltr">

::

  kctrl app kick -a installer.educates.dev -n educates-installer -y

.. raw:: html

   </div>

دستور ``kctrl`` مربوط به ابزارهای Carvel است.

توجه داشته باشید این تغییرات لزوماً روی Training Portal
یا Workshop Environmentهای قبلی تأثیر نمی‌گذارد.

--------------------------------------------------------------------

.. _deleting-installed-package:

حذف package نصب‌شده
--------------------

برای حذف کامل آنچه توسط Educates نصب شده است:

.. raw:: html

   <div dir="ltr">

::

  kubectl delete -n educates-installer app/installer.educates.dev

.. raw:: html

   </div>

این دستور Namespace، Service Account و Secret را حذف نمی‌کند.

برای حذف دستی آن‌ها:

.. raw:: html

   <div dir="ltr">

::

  kubectl delete namespace/educates-installer
  kubectl delete clusterrolebinding/educates-installer

.. raw:: html

   </div>
