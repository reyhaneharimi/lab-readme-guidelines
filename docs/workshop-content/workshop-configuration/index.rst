پیکربندی Workshop
=================

هر Workshop از دو بخش اصلی تشکیل شده است. بخش اول، Workshop definition است که نیازمندی‌های setup برای deploy کردن Workshop و نحوه پیکربندی محیط Educates برای آن Workshop را مشخص می‌کند. بخش دوم، Workshop files هستند که شامل دستورالعمل‌های Workshop، فایل‌های setup مربوط به Workshop و هرگونه فایل تمرینی موردنیاز برای Workshop می‌باشند.

نیازمندی‌های setup برای Workshop
--------------------------------

Workshop imageها را می‌توان مستقیماً روی یک container runtime deploy کرد. برای مدیریت deployment در یک Kubernetes cluster، Educates operator ارائه شده است. پیکربندی Educates operator از طریق یک ``Workshop`` custom resource definition انجام می‌شود. هنگام استفاده از workshop template برای ایجاد فایل‌های اولیه یک Workshop، این definition در فایل ``resources/workshop.yaml`` قرار دارد.

```yaml
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
```

در این نمونه، Workshop files از یک Git repository که روی GitHub میزبانی می‌شود دانلود می‌گردند. این موضوع در بخش ``workshop.files`` از Workshop definition مشخص شده است. فایل‌های Workshop روی standard workshop base image به‌صورت overlay اعمال می‌شوند.

علاوه بر standard workshop base image، Educates، workshop base imageهای دیگری برای کار با Java و Python نیز ارائه می‌دهد. برای انتخاب workshop base image با پشتیبانی از Java JDK 17 باید از تنظیم زیر استفاده کنید:

```yaml
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
```

در این حالت، workshop image جایگزین با تنظیم property ``workshop.image`` مشخص شده است.

علاوه بر انتخاب یکی از workshop base imageهای ارائه‌شده توسط Educates، می‌توانید یک workshop image سفارشی خودتان را نیز مشخص کنید:

```yaml
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
```

در صورت استفاده از custom workshop image، می‌توانید Workshop files را داخل همان image قرار دهید، یا همچنان آن‌ها را به‌صورت جداگانه دانلود کرده و روی image به‌صورت overlay اعمال کنید.

علاوه بر تعیین workshop base image و منبع Workshop files، ``Workshop`` definition برای پیکربندی workshop environment و workshop session نیز استفاده می‌شود. این تنظیمات شامل مواردی مانند فعال بودن embedded editor یا Kubernetes web console، منابع اضافی که همراه workshop environment یا session deploy می‌شوند، میزان memory و storage موردنیاز برای یک workshop session، و همچنین quota قابل استفاده هنگام deploy کردن workloadها در Kubernetes cluster می‌باشد.

برای جزئیات بیشتر به مستندات جداگانه مربوط به [Workshop definition](workshop-definition) مراجعه کنید.

گزینه‌های render دستورالعمل‌ها
------------------------------

در حال حاضر Educates از دو renderer مختلف برای Workshop instructions پشتیبانی می‌کند.

اولین و renderer اصلی که در Educates وجود دارد، ``classic`` renderer نام دارد. این یک dynamic web application سفارشی برای render کردن Workshop instructions است و از Markdown یا AsciiDoc پشتیبانی می‌کند.

renderer دوم، ``hugo`` renderer نام دارد. همان‌طور که از نام آن مشخص است، از Hugo برای تولید فایل‌های static HTML با استفاده از layoutهای سفارشی ارائه‌شده توسط Educates استفاده می‌کند. Hugo تنها از Markdown پشتیبانی می‌کند.

در هر دو حالت، صفحاتی که Workshop instructions را تشکیل می‌دهند در مسیر ``workshop/content`` قرار می‌گیرند. در صورت نیاز به پیکربندی navigation path بین صفحات، نحوه انجام آن بسته به renderer مورد استفاده متفاوت است.

پیکربندی classic renderer
--------------------------

در صورت استفاده از ``classic`` renderer برای Workshop instructions، روش‌های مختلفی برای پیکربندی ساختار محتوا وجود دارد. روشی که در sample workshopها استفاده شده، استفاده از فایل‌های YAML است.

فایل ``workshop/modules.yaml`` شامل لیست moduleهای در دسترس که Workshop شما را تشکیل می‌دهند و همچنین data variableهای قابل استفاده در محتوا است.

در لیست moduleها، ممکن است همه moduleها مورد استفاده قرار نگیرند. این لیست نشان‌دهنده مجموعه کامل moduleهای موجود است که ممکن است از آن‌ها استفاده کنید. ممکن است بخواهید variationهای مختلفی از Workshop خود اجرا کنید، مثلاً برای زبان‌های برنامه‌نویسی متفاوت. بنابراین moduleهای فعال که برای یک Workshop خاص استفاده می‌شوند در فایل جداگانه ``workshop/workshop.yaml`` مشخص می‌شوند، به‌همراه نامی که برای آن variation از Workshop در نظر گرفته می‌شود.

به‌صورت پیش‌فرض فایل ``workshop.yaml`` تعیین می‌کند که چه moduleهایی استفاده شوند. اگر بخواهید variationهای متفاوتی ارائه دهید، می‌توانید چندین workshop file با نام‌های مختلف ایجاد کنید، مانند ``workshop-java.yaml`` و ``workshop-python.yaml``.

اگر چند workshop file داشته باشید و فایل پیش‌فرض ``workshop.yaml`` وجود نداشته باشد، می‌توانید با تنظیم environment variable به نام ``WORKSHOP_FILE`` در runtime configuration مربوط به Workshop، فایل موردنظر را مشخص کنید.

فرمت تعریف moduleهای در دسترس در فایل ``workshop/modules.yaml`` به شکل زیر است:

```yaml
modules:
  00-workshop-overview:
    name: Workshop Overview
    exit_sign: Start Workshop
  01-workshop-instructions:
    name: Workshop Instructions
  99-workshop-summary:
    name: Workshop Summary
    exit_sign: Finish Workshop
```

هر module در بخش ``modules`` لیست می‌شود. نام هر module باید با مسیر فایل محتوای مربوط به آن مطابقت داشته باشد، بدون در نظر گرفتن extension فایل.

برای هر module، فیلد ``name`` عنوان صفحه‌ای است که برای آن module نمایش داده می‌شود. اگر فیلدی مشخص نشود و ``name`` تنظیم نشده باشد، عنوان بر اساس نام فایل module محاسبه می‌شود.

فایل متناظر ``workshop/workshop.yaml`` که در آن همه moduleهای موجود استفاده شده‌اند، ساختاری مانند زیر دارد:

```yaml
name: Workshop
modules:
  activate:
  - 00-workshop-overview
  - 01-workshop-instructions
  - 99-workshop-summary
```

فیلد سطح بالای ``name`` در این فایل، نام این variation از محتوای Workshop است.

فیلد ``modules.activate`` لیستی از moduleهایی است که برای Workshop استفاده می‌شوند. نام‌ها باید دقیقاً با نام‌های تعریف‌شده در فایل modules مطابقت داشته باشند.

ترتیب پیمایش صفحات بر اساس ترتیب moduleها در بخش ``modules.activate`` از workshop configuration file تعیین می‌شود. ترتیب قرارگیری moduleها در فایل modules اهمیتی ندارد.

در پایین هر صفحه یک دکمه "Continue" برای رفتن به صفحه بعدی نمایش داده می‌شود. برچسب این دکمه را می‌توان با تنظیم فیلد ``exit_sign`` در تعریف module مربوطه تغییر داد.

برای آخرین module نیز یک دکمه نمایش داده می‌شود، اما مقصد آن می‌تواند متفاوت باشد.

اگر بخواهید پس از اتمام Workshop کاربر به یک وب‌سایت دیگر هدایت شود، می‌توانید فیلد ``exit_link`` مربوط به آخرین module را روی یک URL خارجی تنظیم کنید. همچنین می‌توان environment variable به نام ``RESTART_URL`` را در workshop environment تنظیم کرد تا مقصد نهایی کنترل شود.

اگر مقصدی برای صفحه پایانی مشخص نشود، کاربر به صفحه ابتدایی Workshop بازگردانده می‌شود.

در صورت استفاده از training portal، این environment variable override می‌شود تا پس از اتمام Workshop، کاربر به training portal هدایت شود.

توصیه می‌شود در صفحه پایانی مقدار ``exit_sign`` برابر با "Finish Workshop" تنظیم شود و ``exit_link`` مشخص نگردد. این کار باعث می‌شود مقصد نهایی از طریق workshop environment یا training portal کنترل شود.

(hugo-renderer-configuration)=
پیکربندی hugo renderer
----------------------

در صورت استفاده از ``hugo`` renderer برای Workshop instructions، مشخص کردن navigation path از طریق یک configuration file جداگانه اختیاری است.

به‌صورت پیش‌فرض هنگام استفاده از ``hugo`` renderer، تمام صفحات موجود در مسیر ``workshop/content`` در navigation path قرار می‌گیرند. ترتیب صفحات بر اساس pathname آن‌ها در این دایرکتوری محاسبه می‌شود. اگر بخواهید از مکانیزم پیش‌فرض Hugo برای ترتیب‌دهی استفاده کنید، می‌توانید فیلد weight را در metadata هر صفحه اضافه کنید. در این حالت ترتیب بر اساس weight و سپس عنوان صفحه تعیین می‌شود. برای مشخص کردن عنوان صفحه می‌توانید فیلد title را در metadata هر صفحه قرار دهید.

اگر ترجیح دهید ترتیب صفحات را از طریق یک فایل YAML مشخص کنید، تنها برخی صفحات را شامل شوید یا چند pathway مختلف داشته باشید، می‌توانید فایل ``workshop/config.yaml`` را ایجاد کنید.

در صورت استفاده از ``workshop/config.yaml``، ساختار آن برای تعریف navigation pathwayها به شکل زیر خواهد بود:

```yaml
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
```

برخلاف ``classic`` renderer، بخش ``modules`` حتی در صورت تعریف navigation pathway نیز اختیاری است. اگر moduleای که در یک pathway قرار دارد در بخش ``modules`` تعریف نشده باشد، عنوان آن از metadata همان صفحه گرفته می‌شود.

اگر بخواهید چند pathway مختلف داشته باشید، می‌توانید هرکدام را به‌صورت جداگانه در بخش ``pathways.paths`` تعریف کنید. مقدار ``pathways.default`` باید pathway پیش‌فرض را مشخص کند. همچنین می‌توانید عنوان Workshop را در فیلد ``title`` مربوط به هر pathway override کنید.

برای انتخاب pathway هنگام شروع Workshop، می‌توانید environment variable به نام ``PATHWAY_NAME`` را در Workshop definition تنظیم کنید.

در ``hugo`` renderer امکان override کردن برچسب دکمه‌های navigation یا تغییر مقصد آن‌ها مانند آنچه در ``classic`` renderer وجود داشت، فراهم نیست.
