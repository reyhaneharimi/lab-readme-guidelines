.. _configuration-settings:

تنظیمات پیکربندی
================

در زمان نصب Educates می‌توان تنظیمات مختلفی را مشخص کرد.
برخی از این تنظیمات ضروری هستند و برخی اختیاری.
بعضی تنظیمات نیز هنگام ایجاد TrainingPortal قابل Override هستند،
اما تنظیمات اصلی باید هنگام نصب مشخص شوند.

--------------------------------------------------------------------

.. _defining-configuration-for-ingress:

تعریف پیکربندی Ingress
-----------------------

اگر دامنه سفارشی برای Ingress مشخص نشود،
دامنه پیش‌فرض ``educates-local-dev.xyz`` استفاده خواهد شد.

برای Override دامنه:

.. raw:: html

   <div dir="ltr">

::

  clusterIngress:
    domain: "workshops.example.com"

.. raw:: html

   </div>

اگر TLS با Secret جداگانه تعریف شود:

.. raw:: html

   <div dir="ltr">

::

  clusterIngress:
    domain: "workshops.example.com"
    tlsCertificateRef:
      namespace: "default"
      name: "workshops.example.com-tls"

.. raw:: html

   </div>

TLS به صورت inline:

.. raw:: html

   <div dir="ltr">

::

  clusterIngress:
    domain: "workshops.example.com"
    tlsCertificate:
      tls.crt: |
        ...
      tls.key: |
        ...

.. raw:: html

   </div>

Override کردن protocol:

.. raw:: html

   <div dir="ltr">

::

  clusterIngress:
    domain: "workshops.example.com"
    protocol: "https"

.. raw:: html

   </div>

Override کردن ingress class:

.. raw:: html

   <div dir="ltr">

::

  clusterIngress:
    domain: "workshops.example.com"
    class: "nginx"

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _defining-cluster-policy-engine:

تعریف موتور سیاست امنیتی کلاستر
--------------------------------

پیشنهادی برای Kubernetes معمولی:

.. raw:: html

   <div dir="ltr">

::

  clusterSecurity:
    policyEngine: "kyverno"

.. raw:: html

   </div>

برای Pod Security Policies:

.. raw:: html

   <div dir="ltr">

::

  clusterSecurity:
    policyEngine: "pod-security-policies"

.. raw:: html

   </div>

برای OpenShift:

.. raw:: html

   <div dir="ltr">

::

  clusterSecurity:
    policyEngine: "security-context-constraints"

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _defining-workshop-policy-engine:

تعریف موتور سیاست امنیتی ورکشاپ
--------------------------------

.. raw:: html

   <div dir="ltr">

::

  workshopSecurity:
    policyEngine: "kyverno"

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _overriding-container-runtime-class:

Override کردن container runtime class
--------------------------------------

.. raw:: html

   <div dir="ltr">

::

  clusterRuntime:
    class: kata-qemu

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _defining-image-registry-pull-secrets:

تعریف image pull secret
-----------------------

.. raw:: html

   <div dir="ltr">

::

  clusterSecrets:
    pullSecretRefs:
      - namespace: "default"
        name: "registry.example.com-pull"

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _defining-storage-class:

تعریف storage class
-------------------

.. raw:: html

   <div dir="ltr">

::

  clusterStorage:
    class: "default"

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _defining-storage-group:

تعریف storage group
-------------------

.. raw:: html

   <div dir="ltr">

::

  clusterStorage:
    group: 1

.. raw:: html

   </div>

یا:

.. raw:: html

   <div dir="ltr">

::

  clusterStorage:
    user: 1
    group: 1

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _restricting-network-access:

محدودسازی دسترسی شبکه
----------------------

.. raw:: html

   <div dir="ltr">

::

  clusterNetwork:
    blockCIDRs:
      - "169.254.169.254/32"
      - "fd00:ec2::254/128"

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _overriding-network-mtu:

Override کردن MTU شبکه
----------------------

.. raw:: html

   <div dir="ltr">

::

  dockerDaemon:
    networkMTU: 1400

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _image-registry-pull-through-cache:

فعال‌سازی registry pull-through cache
--------------------------------------

.. raw:: html

   <div dir="ltr">

::

  dockerDaemon:
    proxyCache:
      remoteURL: "https://registry-1.docker.io"

.. raw:: html

   </div>

با احراز هویت:

.. raw:: html

   <div dir="ltr">

::

  dockerDaemon:
    proxyCache:
      remoteURL: "https://registry-1.docker.io"
      username: "username"
      password: "access-token"

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _setting-default-access-credentials:

تنظیم credential پیش‌فرض
------------------------

.. raw:: html

   <div dir="ltr">

::

  trainingPortal:
    credentials:
      admin:
        username: "educates"
        password: "admin-password"
      robot:
        username: "robot@educates"
        password: "robot-password"

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _tracking-using-workshop-events:

ردیابی با webhook
-----------------

.. raw:: html

   <div dir="ltr">

::

  workshopAnalytics:
    webhook:
      url: "https://metrics.educates.dev/?client=name&token=password"

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _tracking-using-google-analytics:

ردیابی با Google Analytics
--------------------------

.. raw:: html

   <div dir="ltr">

::

  workshopAnalytics:
    google:
      trackingId: "G-XXXXXXXXXX"

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _tracking-using-microsoft-clarity:

ردیابی با Microsoft Clarity
---------------------------

.. raw:: html

   <div dir="ltr">

::

  workshopAnalytics:
    clarity:
      trackingId: "XXXXXXXXXX"

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _tracking-using-amplitude:

ردیابی با Amplitude
-------------------

.. raw:: html

   <div dir="ltr">

::

  workshopAnalytics:
    amplitude:
      trackingId: "XXXXXXXXXX"

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _overriding-workshop-styling:

Override کردن استایل ورکشاپ
----------------------------

.. raw:: html

   <div dir="ltr">

::

  websiteStyling:
    workshopDashboard:
      style: |
        body {
          font-family: "Comic Sans MS", cursive, sans-serif;
        }

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _allowing-sites-to-embed-workshops:

اجازه embed کردن ورکشاپ
-----------------------

.. raw:: html

   <div dir="ltr">

::

  websiteStyling:
    frameAncestors:
      - example.com

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _overriding-session-cookie-domain:

Override کردن دامنه Cookie
--------------------------

.. raw:: html

   <div dir="ltr">

::

  sessionCookies:
    domain: "example.com"

.. raw:: html

   </div>
