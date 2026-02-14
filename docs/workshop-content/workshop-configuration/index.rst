.. _workshop-configuration:

پیکربندی Workshop
=================

هر Workshop از دو بخش اصلی تشکیل شده است. بخش اول، Workshop definition است که نیازمندی‌های setup برای deploy کردن Workshop و نحوه پیکربندی محیط Educates برای آن Workshop را مشخص می‌کند. بخش دوم، Workshop files هستند که شامل دستورالعمل‌های Workshop، فایل‌های setup مربوط به Workshop و هرگونه فایل تمرینی موردنیاز برای Workshop می‌باشند.

--------------------------------------------------------------------

.. _workshop-setup-requirements:

نیازمندی‌های setup برای Workshop
--------------------------------

Workshop imageها را می‌توان مستقیماً روی یک container runtime deploy کرد. برای مدیریت deployment در یک Kubernetes cluster، Educates operator ارائه شده است. پیکربندی Educates operator از طریق یک ``Workshop`` custom resource definition انجام می‌شود. هنگام استفاده از workshop template برای ایجاد فایل‌های اولیه یک Workshop، این definition در فایل ``resources/workshop.yaml`` قرار دارد.

.. code-block:: yaml

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

در این نمونه، Workshop files از یک Git repository که روی GitHub میزبانی می‌شود دانلود می‌گردند. این موضوع در بخش ``workshop.files`` از Workshop definition مشخص شده است. فایل‌های Workshop روی standard workshop base image به‌صورت overlay اعمال می‌شوند.

علاوه بر standard workshop base image، Educates، workshop base imageهای دیگری برای کار با Java و Python نیز ارائه می‌دهد. برای انتخاب workshop base image با پشتیبانی از Java JDK 17 باید از تنظیم زیر استفاده کنید:

.. code-block:: yaml

   apiVersion: training.educates.dev/v1beta1
   kind: Workshop
   metadata:
     name: lab-markdown-sample
   spec:
     title: Markdown Sample
     description: A sample workshop using Markdown
     workshop:
       image: jdk17-environment:*
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

در این حالت، workshop image جایگزین با تنظیم property ``workshop.image`` مشخص شده است.

علاوه بر انتخاب یکی از workshop base imageهای ارائه‌شده توسط Educates، می‌توانید یک workshop image سفارشی خودتان را نیز مشخص کنید:

.. code-block:: yaml

   apiVersion: training.educates.dev/v1beta1
   kind: Workshop
   metadata:
     name: lab-markdown-sample
   spec:
     title: Markdown Sample
     description: A sample workshop using Markdown
     workshop:
       image: ghcr.io/educates/lab-markdown-sample:latest
     session:
       namespaces:
         budget: small
       applications:
         console:
           enabled: true
         editor:
           enabled: true

در صورت استفاده از custom workshop image، می‌توانید Workshop files را داخل همان image قرار دهید، یا همچنان آن‌ها را به‌صورت جداگانه دانلود کرده و روی image به‌صورت overlay اعمال کنید.

علاوه بر تعیین workshop base image و منبع Workshop files، ``Workshop`` definition برای پیکربندی workshop environment و workshop session نیز استفاده می‌شود. این تنظیمات شامل مواردی مانند فعال بودن embedded editor یا Kubernetes web console، منابع اضافی که همراه workshop environment یا session deploy می‌شوند، میزان memory و storage موردنیاز برای یک workshop session، و همچنین quota قابل استفاده هنگام deploy کردن workloadها در Kubernetes cluster می‌باشد.

برای جزئیات بیشتر به مستندات جداگانه مربوط به :ref:`workshop-definition` مراجعه کنید.

--------------------------------------------------------------------

.. _instructions-rendering-options:

گزینه‌های render دستورالعمل‌ها
------------------------------

در حال حاضر Educates از دو renderer مختلف برای Workshop instructions پشتیبانی می‌کند.

اولین و renderer اصلی که در Educates وجود دارد، ``classic`` renderer نام دارد. این یک dynamic web application سفارشی برای render کردن Workshop instructions است و از Markdown یا AsciiDoc پشتیبانی می‌کند.

renderer دوم، ``hugo`` renderer نام دارد. همان‌طور که از نام آن مشخص است، از Hugo برای تولید فایل‌های static HTML با استفاده از layoutهای سفارشی ارائه‌شده توسط Educates استفاده می‌کند. Hugo تنها از Markdown پشتیبانی می‌کند.

در هر دو حالت، صفحاتی که Workshop instructions را تشکیل می‌دهند در مسیر ``workshop/content`` قرار می‌گیرند. در صورت نیاز به پیکربندی navigation path بین صفحات، نحوه انجام آن بسته به renderer مورد استفاده متفاوت است.

--------------------------------------------------------------------

.. _classic-renderer-configuration:

پیکربندی classic renderer
--------------------------

در صورت استفاده از ``classic`` renderer برای Workshop instructions، روش‌های مختلفی برای پیکربندی ساختار محتوا وجود دارد. روشی که در sample workshopها استفاده شده، استفاده از فایل‌های YAML است.

فایل ``workshop/modules.yaml`` شامل لیست moduleهای در دسترس که Workshop شما را تشکیل می‌دهند و همچنین data variableهای قابل استفاده در محتوا است.

ممکن است همه moduleها استفاده نشوند. moduleهای فعال برای یک Workshop خاص در فایل جداگانه ``workshop/workshop.yaml`` مشخص می‌شوند.

نمونه تعریف moduleها در فایل ``workshop/modules.yaml``:

.. code-block:: yaml

   modules:
     00-workshop-overview:
       name: Workshop Overview
       exit_sign: Start Workshop
     01-workshop-instructions:
       name: Workshop Instructions
     99-workshop-summary:
       name: Workshop Summary
       exit_sign: Finish Workshop

نمونه فایل ``workshop/workshop.yaml``:

.. code-block:: yaml

   name: Workshop
   modules:
     activate:
     - 00-workshop-overview
     - 01-workshop-instructions
     - 99-workshop-summary

ترتیب پیمایش صفحات بر اساس ترتیب moduleها در ``modules.activate`` تعیین می‌شود.

در پایین هر صفحه یک دکمه Continue نمایش داده می‌شود. برچسب آن با ``exit_sign`` قابل تغییر است.

--------------------------------------------------------------------

.. _hugo-renderer-configuration:

پیکربندی hugo renderer
----------------------

در صورت استفاده از ``hugo`` renderer، تعیین navigation path از طریق یک configuration file جداگانه اختیاری است.

به‌صورت پیش‌فرض، تمام صفحات موجود در مسیر ``workshop/content`` در navigation path قرار می‌گیرند.

اگر بخواهید ترتیب یا pathwayهای مختلف تعریف کنید، فایل ``workshop/config.yaml`` را ایجاد کنید:

.. code-block:: yaml

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

در ``hugo`` renderer امکان override کردن دکمه‌های navigation مانند ``classic`` وجود ندارد.
