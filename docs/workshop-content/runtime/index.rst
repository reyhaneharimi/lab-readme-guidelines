.. _workshop-runtime:

Workshop Runtime
================

محتوای Workshop می‌تواند مراحلی را که کاربر باید برای اجرای یک Workshop انجام دهد script کند. در برخی موارد ممکن است لازم باشد این محتوا را با اطلاعاتی از runtime environment پارامترایز کنید. data variables در محتوای Workshop تا حدی این کار را انجام می‌دهند، اما در برخی موارد ممکن است بخواهید این کار را از طریق scriptهایی که داخل workshop container اجرا می‌شوند انجام دهید تا فایل‌های configuration آماده شوند.

این کار با ارائه setup scriptهایی که هنگام start شدن container اجرا می‌شوند امکان‌پذیر است. در صورت نیاز می‌توانید background processهای persistent نیز در container اجرا کنید که در طول اجرای Workshop کارهای اضافی انجام دهند.

--------------------------------------------------------------------

.. _pre-defined-environment-variables:

Pre-defined environment variables
---------------------------------

هنگام ایجاد محتوای Workshop می‌توانید از data variables استفاده کنید تا مقادیر مربوط به session یا environment جاری به‌صورت خودکار درج شوند. برای مثال نام workshop session، ingress domain هنگام ایجاد ingress route، و نام Kubernetes namespace مرتبط با workshop session.

این data variables می‌توانند برای نمایش فایل YAML/JSON resource در محتوای Workshop با مقادیر پرشده به‌صورت خودکار استفاده شوند. همچنین می‌توانند در executable commandها جایگزین شوند.

برای commandهایی که در shell environment اجرا می‌شوند، تعدادی pre-defined environment variable نیز در دسترس هستند که می‌توان مستقیماً به آن‌ها ارجاع داد.

Key environment variables عبارتند از:

* ``INGRESS_DOMAIN`` — دامنه‌ای که باید در hostnameهای تولیدشده برای ingress routes استفاده شود.
* ``INGRESS_PROTOCOL`` — protocol (http/https) مورد استفاده برای ingress routes.
* ``PLATFORM_ARCH`` — معماری CPU کانتینر (``amd64`` یا ``arm64``).
* ``SESSION_HOSTNAME`` — hostname مربوط به workshop session.
* ``SESSION_ID`` — شناسه کوتاه workshop session (فقط در context همان workshop environment یکتا است).
* ``SESSION_NAME`` — نام workshop session (در سطح Kubernetes cluster یکتا است).
* ``SESSION_NAMESPACE`` — نام namespace مربوط به workshop session در Kubernetes.
* ``SESSION_URL`` — URL کامل دسترسی به dashboard مربوط به workshop session.
* ``TRAINING_PORTAL`` — نام training portal میزبان Workshop.
* ``WORKSHOP_NAMESPACE`` — namespace مربوط به workshop environment.
* ``WORKSHOP_NAME`` — نام Workshop.

توجه داشته باشید که ``SESSION_NAME`` از نسخه 2.6.0 اضافه شده است. در نسخه‌های قبلی ``SESSION_NAMESPACE`` به‌عنوان شناسه session استفاده می‌شد. از این پس برای اشاره به نام session از ``SESSION_NAME`` استفاده کنید و فقط زمانی از ``SESSION_NAMESPACE`` استفاده کنید که نیاز به اشاره به namespace واقعی در Kubernetes دارید.

به‌جای استفاده از data variable مانند:

```execute
kubectl get all -n {{session_namespace}}
```

می‌توانید از environment variable استفاده کنید:

```execute
kubectl get all -n $SESSION_NAMESPACE
```

در این حالت مقدار environment variable توسط shell جایگزین می‌شود.

--------------------------------------------------------------------

.. _running-steps-on-container-start:

Running steps on container start
--------------------------------

برای اجرای script هنگام start شدن container، یک فایل shell executable با پسوند ``.sh`` در مسیر زیر قرار دهید:

```
workshop/setup.d
```

در صورت restart شدن container، setup script دوباره اجرا خواهد شد، بنابراین script باید idempotent باشد.

برای جایگزینی مقادیر در فایل‌ها، ابزار ``envsubst`` مفید است:

```shell
#!/bin/bash
envsubst < frontend/ingress.yaml.in > frontend/ingress.yaml
```

در فایل ورودی، عبارتی مانند ``${INGRESS_DOMAIN}`` با مقدار environment variable جایگزین می‌شود.

setup scriptها هنگام اجرا، دایرکتوری خانگی کاربر Workshop را به‌عنوان working directory دارند.

اگر نیاز دارید environment variableهایی تعریف کنید که در interactive shell نیز در دسترس باشند، می‌توانید آن‌ها را در فایل مشخص‌شده توسط ``WORKSHOP_ENV`` بنویسید:

```shell
#!/bin/bash
echo "NAME=VALUE" >> $WORKSHOP_ENV
```

محتوای فایل باید با فرمت ``.env`` باشد.

--------------------------------------------------------------------

.. _running-background-applications:

Running background applications
-------------------------------

setup scriptها فقط یک‌بار هنگام start شدن container اجرا می‌شوند. اگر نیاز دارید background application در طول اجرای Workshop فعال بماند، بهتر است آن را با supervisor daemon مدیریت کنید.

برای این کار یک فایل configuration با پسوند ``.conf`` در مسیر زیر ایجاد کنید:

```
workshop/supervisor
```

نمونه configuration:

```text
[program:myapplication]
process_name=myapplication
command=/opt/myapplication/sbin/start-myapplication
stdout_logfile=/proc/1/fd/1
stdout_logfile_maxbytes=0
redirect_stderr=true
```

log خروجی باید به ``stdout`` یا ``stderr`` ارسال شود و سپس به ``/proc/1/fd/1`` هدایت شود.

برای restart یا shutdown برنامه می‌توانید از ``supervisorctl`` استفاده کنید.

--------------------------------------------------------------------

.. _terminal-user-shell-environment:

Terminal user shell environment
-------------------------------

environment variableهایی که در setup script یا background application تعریف می‌شوند مستقیماً بر shell کاربر تأثیر نمی‌گذارند.

shell environment از ``bash`` استفاده می‌کند و فایل:

```
$HOME/.bash_profile
```

خوانده می‌شود.

برای سفارشی‌سازی interactive shell می‌توانید فایل زیر را ایجاد کنید:

```
workshop/profile
```

این فایل هنگام ایجاد هر terminal session به‌صورت خودکار source می‌شود.

این فایل فقط برای سفارشی‌سازی shell interactive استفاده شود (مانند تغییر prompt یا command completion).

اگر نیاز به logic پیچیده‌تر دارید (مانند query کردن Kubernetes REST API)، از ``workshop/setup.d`` استفاده کنید و environment variableها را در فایل ``WORKSHOP_ENV`` بنویسید.

همچنین می‌توانید از مسیر زیر استفاده کنید:

```
workshop/profile.d
```

هر فایل ``.sh`` در این مسیر یک‌بار هنگام initialize شدن container اجرا می‌شود. توجه داشته باشید که این قابلیت ممکن است در آینده deprecated شود و توصیه می‌شود از ``WORKSHOP_ENV`` استفاده شود.

--------------------------------------------------------------------

.. _overriding-terminal-shell-command:

Overriding terminal shell command
---------------------------------

به‌صورت پیش‌فرض هر terminal session با ``bash`` اجرا می‌شود.

برای override کردن command مربوط به یک session خاص، یک فایل executable با نام:

```
workshop/terminal/<session>.sh
```

ایجاد کنید. نام sessionهای پیش‌فرض: ``1``، ``2`` و ``3``.

مثال اجرای ``k9s``:

```shell
#!/bin/bash
exec k9s
```

اگر همچنان می‌خواهید یک interactive shell اجرا شود، اما در ابتدای session یک banner حاوی اطلاعات خاص برای کاربر نمایش داده شود، می‌توانید از یک script استفاده کنید که ابتدا banner را چاپ کند و سپس interactive shell را اجرا کند.

```shell
#!/bin/bash
echo
echo "Your session namespace is "$SESSION_NAMESPACE"."
echo
exec bash
```

در صورتی که نیاز داشته باشید یک command پیش‌فرض برای تمام terminal sessionها (صرف‌نظر از نام آن‌ها) تعیین کنید — برای مثال جهت redirect کردن همه sessionها به یک virtual machine یا container — می‌توانید یک فایل shell executable با نام ``terminal.sh`` در مسیر ``workshop`` ایجاد کنید.

توجه داشته باشید اگر برای یک terminal session خاص فایل script جداگانه‌ای تعریف شده باشد، آن فایل نسبت به ``terminal.sh`` اولویت خواهد داشت.
