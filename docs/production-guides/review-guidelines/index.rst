.. _review-guidelines:

Review Guidelines
=================

.. _changes-security-policy:

Changes to the security policy
-------------------------------

به‌صورت پیش‌فرض، RBAC برای namespaceهای مربوط به WorkshopSession به‌گونه‌ای تنظیم شده است که containerها فقط به‌عنوان کاربر non‑root اجرا شوند. دلیل این موضوع، ریسک امنیتی بالای اجرای containerها با دسترسی root برای کاربران ورکشاپ است.

در حالت ایده‌آل این محدودیت مناسب است، اما در عمل بسیاری از container imageها best practice اجرای non‑root را رعایت نکرده‌اند و تنها در صورتی به‌درستی کار می‌کنند که به‌صورت root اجرا شوند. این مسئله به‌ویژه در مورد بسیاری از imageهای رسمی Docker Hub صدق می‌کند.

به‌عنوان مثال، image رسمی ``nginx`` در Docker Hub که اغلب در نمونه‌استقرارهای Kubernetes استفاده می‌شود، فقط در صورتی به‌درستی اجرا می‌شود که به‌صورت root اجرا گردد.

ورکشاپ‌ها می‌توانند policy امنیتی پیش‌فرض ``restricted`` که از طریق RBAC اعمال می‌شود را با تنظیم property زیر override کنند:

``session.namespaces.security.policy``

در حال حاضر این مقدار می‌تواند به ``baseline`` تنظیم شود تا اجرای containerها به‌صورت root مجاز گردد.

نمونه تنظیم در YAML:

.. code-block:: yaml

  spec:
    session:
      namespaces:
        security:
          policy: baseline

با این حال، راهکار توصیه‌شده این است که از هیچ container imageی که نیاز به اجرای root دارد استفاده نشود. در مورد ``nginx`` می‌توان به‌جای image رسمی، از image ``bitnami/nginx`` استفاده کرد. کاتالوگ Bitnami شامل imageهای متنوع دیگری نیز هست که می‌توانند به‌صورت non‑root اجرا شوند، در حالی‌که نسخه‌های رسمی Docker Hub معمولاً نیازمند root هستند.

در صورت ضرورت کامل، یک workshop definition می‌تواند pod security policies اختصاصی خود را تعریف کرده و آن‌ها را به یک service account مجزا که توسط deployment استفاده می‌شود bind کند تا سطح دسترسی بالاتری اعطا شود، حتی اجرای privileged container. اما این کار باید تا حد امکان اجتناب شود، زیرا ریسک امنیتی بسیار بالایی دارد. در چنین حالتی کاربر ورکشاپ می‌تواند deployment دلخواه خود را با استفاده از آن service account و یک container image مخرب ایجاد کرده و امنیت کل cluster را به خطر بیندازد.

**Recommendations**

* در صورت امکان، ورکشاپ‌ها هرگز نباید از container imageهایی استفاده کنند که نیاز به اجرای root دارند.
* اگر ضرورتی وجود ندارد، security policy را از ``restricted`` به ``baseline`` تغییر ندهید.
* از اعطای دسترسی‌های elevated خارج از مقادیر پیش‌فرض ``restricted`` و ``baseline`` خودداری کنید، به‌ویژه امکان اجرای privileged container، زیرا ریسک امنیتی بسیار شدیدی ایجاد می‌کند.

**Related Issues**

توجه داشته باشید که اگر Kubernetes virtual cluster برای یک WorkshopSession فعال شود، policy امنیتی namespace آن session به‌طور خودکار از ``restricted`` به ``baseline`` تغییر می‌کند. این تغییر برای عملکرد صحیح virtual cluster ضروری است، اما به این معناست که هر deploymentی که توسط کاربر در virtual cluster ایجاد شود می‌تواند به‌صورت root اجرا گردد.


.. _disabling-kubernetes-access:

Disabling Kubernetes access
----------------------------

به‌صورت پیش‌فرض، هر WorkshopSession دسترسی به یک namespace در Kubernetes cluster مربوط به همان session را در اختیار کاربر قرار می‌دهد. این دسترسی به این دلیل فراهم می‌شود که در طول Workshop بتوان deploymentهایی را در Kubernetes cluster ایجاد کرد. معمولاً کاربر Workshop فقط به همان namespace اختصاصی session خود دسترسی دارد.

اگر موضوع یک Workshop به‌گونه‌ای باشد که کاربر هرگز نیازی به deploy کردن منابع در Kubernetes cluster نداشته باشد، می‌توان دسترسی به Kubernetes REST API را برای آن Workshop غیرفعال کرد.

این کار با تنظیم مقدار ``session.namespaces.security.token.enabled`` به ``false`` در تعریف Workshop انجام می‌شود. در این حالت، service account token داخل container مربوط به Workshop mount نخواهد شد.

نمونه تنظیم در YAML:

.. code-block:: yaml

  spec:
    session:
      namespaces:
        security:
          token:
            enabled: false

**Recommendations**

* در صورتی که نیازی به دسترسی به Kubernetes REST API وجود ندارد، حتماً این دسترسی را غیرفعال کنید.


.. _workshop-user-admin-access:

Workshop user admin access
---------------------------

به‌صورت پیش‌فرض، اگر به کاربر Workshop دسترسی به cluster داده شود، او در namespace مربوط به session خود دارای نقش admin خواهد بود. با این حال، کاربر امکان انجام عملیات cluster admin روی کل Kubernetes cluster را ندارد.

یک کاربر Workshop می‌تواند با override کردن default RBAC rules به cluster admin access دست پیدا کند، اما این کار هرگز نباید برای Workshopهایی که روی shared cluster اجرا می‌شوند و کاربران عمومی یا غیرقابل‌اعتماد به آن‌ها دسترسی دارند، مجاز باشد.

cluster admin access فقط در شرایطی باید استفاده شود که کاربر Workshop آن را روی cluster شخصی خود و روی سیستم محلی خود اجرا می‌کند و تنها از Workshopهای خودش استفاده می‌کند.

ریسک واضح اعطای cluster admin access این است که کاربر به سطح دسترسی‌ای فراتر از نیاز واقعی دست پیدا می‌کند. در یک shared cluster، این می‌تواند منجر به دخالت در کار سایر کاربران، دسترسی به اطلاعات محرمانه (Secrets)، یا حتی آسیب‌زدن به خود cluster شود.

اگر یک Workshop واقعاً به سطحی از cluster admin access نیاز داشته باشد، می‌توان به‌جای آن از virtual cluster استفاده کرد. یک virtual cluster برای هر کاربر Workshop یک Kubernetes cluster مجزا و مستقل با cluster admin access فراهم می‌کند. در این حالت کاربر می‌تواند عملیات مدیریتی را در همان virtual cluster انجام دهد، اما همچنان به سیستم‌عامل UNIX نودهای زیرساختی یا Kubernetes control plane اصلی دسترسی نخواهد داشت.

**Recommendations**

* اطمینان حاصل کنید که Workshopها به کاربران خود cluster admin access اعطا نکنند.
* هرگز cluster admin access یا سایر elevated privilegeها را به service accountهایی که در namespaceهای session قرار دارند و کاربر Workshop به آن‌ها دسترسی دارد، اعطا نکنید.
* اگر یک Workshop نیاز به نمایش قابلیت‌هایی دارد که مستلزم cluster admin access است، از virtual cluster استفاده کنید.


.. _workshop-container-memory:

Workshop container memory
--------------------------

به‌صورت پیش‌فرض، container مربوط به محیط Workshop با مقدار 512Mi حافظه تخصیص داده می‌شود. اگر editor فعال باشد، این مقدار به 1Gi افزایش می‌یابد. این مقدار حافظه به‌صورت guaranteed تخصیص داده می‌شود، بنابراین Kubernetes هنگام scheduling podها، همان میزان حافظه را روی node رزرو می‌کند.

این مقدار حافظه شامل تمام دستورات و فرآیندهایی است که از طریق command line اجرا می‌شوند. اگر Workshop شامل code compilation باشد (به‌ویژه برنامه‌های Java)، ممکن است لازم باشد مقدار حافظه اختصاص‌یافته به container افزایش یابد. همچنین اگر Workshop به‌طور گسترده از editor استفاده کند یا کاربران را تشویق به بررسی source code و فایل‌های همراه Workshop نماید، مصرف حافظه editor می‌تواند به‌مرور افزایش پیدا کند. بنابراین هنگام تصمیم‌گیری درباره افزایش حافظه، این موضوع را در نظر بگیرید.

در صورت عدم تخصیص حافظه کافی، editor یا برنامه‌هایی که از طریق command line اجرا می‌شوند ممکن است به دلیل کمبود حافظه (out of memory) با خطا مواجه شوند.

مقدار حافظه در دسترس Workshop container با استفاده از تنظیم ``session.resources.memory`` در تعریف Workshop قابل override است.

نمونه تنظیم در YAML:

.. code-block:: yaml

  spec:
    session:
      resources:
        memory: 2Gi

**Recommendations**

* در صورتی که editor مورد نیاز نیست، آن را فعال نکنید.
* در صورت عدم نیاز، Kubernetes web console را فعال نکنید.
* میزان حداکثر حافظه مورد نیاز Workshop container را تحلیل کرده و مقدار حافظه را به‌صورت مناسب تنظیم کنید.

**Related issues**

توجه داشته باشید اگر Docker support فعال باشد و شما در داخل محیط Workshop (و نه به‌صورت deployment در cluster) اقدام به build یا اجرای container کنید، حافظه تخصیص‌یافته به dockerd جدا از حافظه Workshop container است و نباید در محاسبه فوق لحاظ شود.

همچنین افزایش اندازه nodeهای Kubernetes cluster برای افزایش حافظه در دسترس هر node لزوماً توصیه نمی‌شود. تعداد persistent volumeهایی که می‌توان روی یک node mount کرد معمولاً محدود است و با افزایش حافظه node افزایش نمی‌یابد. به همین دلیل، یک راهنمای کلی این است که از nodeهایی با 32Gi حافظه (به‌جای 64Gi) استفاده کرده و در صورت نیاز تعداد nodeها را افزایش دهید.

حافظه به این صورت guaranteed می‌شود که مقدار ``requests`` برابر با مقدار ``limits`` در container مربوط به Workshop deployment تنظیم می‌گردد. اگرچه Kubernetes هنگام scheduling تلاش می‌کند deployment را روی nodeی قرار دهد که حافظه کافی دارد، اما در صورت فعال بودن autoscalar ممکن است همچنان با مشکل مواجه شوید.

مشکل autoscalar این است که اگر منابع یک node به اندازه کافی استفاده نشود، ممکن است تصمیم بگیرد workloadهای مستقر روی آن node را evict کرده و آن‌ها را روی node دیگری redeploy کند. نمونه‌ای از Kubernetes cluster مبتنی بر IaaS که از autoscalar استفاده می‌کند، GKE Autopilot است.

در clusterهایی که autoscaling فعال است، ممکن است Workshop session قطع شود و کار کاربر از دست برود، زیرا با redeploy شدن container، session از ابتدا شروع می‌شود. به همین دلیل توصیه می‌شود Educates را روی clusterهایی که autoscalar آن‌ها می‌تواند تعداد nodeها را به‌صورت خودکار کاهش دهد، مستقر نکنید.





.. _workshop-container-cpu:

Workshop container CPU
-----------------------

به‌صورت پیش‌فرض، برای Workshop session container هیچ مقدار resource requirement مشخصی برای CPU تعیین نشده است. در اغلب موارد این موضوع مشکلی ایجاد نمی‌کند، زیرا فعالیت‌هایی که از طریق terminal داخل Workshop container انجام می‌شوند معمولاً CPU intensive نیستند یا کوتاه‌مدت هستند.

در صورتی که تعداد زیادی Workshop session روی یک node اجرا شوند و همگی به‌طور هم‌زمان workloadهای CPU intensive اجرا کنند، Kubernetes مصرف CPU هر container را نسبت به سایر containerها محدود می‌کند. در نتیجه، هیچ containerی نمی‌تواند به‌طور کامل CPU موجود را monopolize کند، زیرا از آنجا که request مشخصی برای CPU تعریف نشده است، منابع CPU به‌صورت best effort تخصیص داده می‌شوند.

با این حال، این استراتژی پیش‌فرض همیشه بهترین تجربه کاربری را تضمین نمی‌کند، به‌ویژه زمانی که تعداد Workshop sessionهای هم‌زمان افزایش یابد.

یک سناریوی مشکل‌ساز زمانی رخ می‌دهد که Workshop session دستورات CPU intensive و long running (برای مثال GraalVM compilation) را اجرا کند. اگر تعداد زیادی از این sessionها روی یک node schedule شوند و بازه‌های زمانی مصرف بالای CPU آن‌ها با یکدیگر overlap داشته باشد، ممکن است برای CPU با یکدیگر رقابت کرده (CPU starvation) و عملکرد به‌طور محسوسی کاهش یابد.

در چنین شرایطی توصیه می‌شود:

- این نوع Workshop روی یک cluster اختصاصی اجرا شود.
- nodeها از instance typeهایی با CPU بالا استفاده کنند.
- Workshopهای دیگر روی همان cluster اجرا نشوند.

علاوه بر این، باید یک pod template patch روی Workshop container اعمال شود تا مقادیر request و limit برای CPU مشخص گردد. این کار باعث می‌شود مقدار مشخصی CPU به‌صورت guaranteed در اختیار container قرار گیرد و در عین حال سقف مصرف CPU نیز محدود شود.

نمونه تنظیم در YAML:

.. code-block:: yaml

  spec:
    session:
      patches:
        containers:
        - name: workshop
          resources:
            requests:
              cpu: "500m"
            limits:
              cpu: "1000m"

**Recommendations**

* Workshopهایی که برای مدت طولانی به CPU بالا نیاز دارند باید روی clusterهای اختصاصی با nodeهای دارای CPU بالا اجرا شوند.
* برای Workshopهایی که مصرف CPU بالا و طولانی‌مدت دارند، حتماً مقادیر request و limit برای CPU در Workshop container مشخص شود.



.. _workshop-container-storage:

Workshop container storage
---------------------------

ورکشاپ‌ها باید تمام فایل‌هایی که در طول اجرای Workshop ایجاد می‌شوند را در دایرکتوری home کاربر Workshop ذخیره کنند. به‌صورت پیش‌فرض، این دایرکتوری بخشی از filesystem موقت (transient filesystem) داخل container است. بنابراین، تمام instanceهای Workshop که روی یک Kubernetes node اجرا می‌شوند، برای استفاده از فضای دیسک آزاد موجود روی آن node رقابت خواهند کرد.

اگر یک Workshop حجم زیادی از binaryها یا source code را داخل Workshop container دانلود کند، فرآیند compilation انجام دهد، یا تعداد زیادی package مورد نیاز برای build برنامه‌ها (به‌ویژه در Java یا NodeJS) را دانلود کند، در صورتی که تعداد زیادی instance از آن Workshop به‌طور هم‌زمان روی یک Kubernetes node اجرا شوند، ممکن است فضای filesystem node به اتمام برسد. در چنین شرایطی Kubernetes ممکن است podها را از node evict کند. در مورد Workshop instanceها، این موضوع منجر به از دست رفتن کار کاربر شده و او باید session را از ابتدا شروع کند.

برای جلوگیری از اتمام فضای filesystem زمانی که یک Workshop به مقدار زیادی transient storage نیاز دارد، باید فضای ذخیره‌سازی مشخصی به Workshop container برای دایرکتوری home کاربر اختصاص داده شود. در این حالت، یک persistent volume claim به میزان فضای درخواستی ایجاد شده و آن persistent volume روی دایرکتوری home کاربر mount می‌شود.

مقدار storage در دسترس Workshop container با استفاده از تنظیم ``session.resources.storage`` در تعریف Workshop قابل override است.

نمونه تنظیم در YAML:

.. code-block:: yaml

  spec:
    session:
      resources:
        storage: 5Gi

**Recommendations**

* اگر Workshop در طول اجرا حجم زیادی فایل تولید می‌کند، حتماً برای Workshop container فضای storage اختصاص دهید.

**Related Issues**

در صورتی که از یک custom workshop image استفاده شود که فایل‌های مورد نیاز Workshop را در دایرکتوری home قرار داده باشد، persistent volume روی آن mount خواهد شد. برای حل این مسئله، هنگامی که storage درخواست شود، محیط Workshop ابتدا یک init container اجرا می‌کند که محتوای دایرکتوری home در custom workshop image را به persistent volume کپی می‌کند. سپس هنگام اجرای container اصلی، persistent volume روی دایرکتوری home mount می‌شود. به این ترتیب، فایل‌های موجود در custom workshop image به‌صورت شفاف به persistent volume منتقل می‌شوند و نیازی به اقدام اضافی از طرف شما نیست. با این حال، اگر تعداد فایل‌ها زیاد باشد، این فرآیند می‌تواند باعث تأخیر در startup Workshop session شود.

توجه داشته باشید که نمی‌توان از مقادیر storage که روی Kubernetes clusterهای محلی مانند Kind یا Minikube استفاده می‌شوند، به‌عنوان مبنا برای محیط Production استفاده کرد. در این clusterهای محلی، مقدار storage درخواستی معمولاً نادیده گرفته می‌شود و عملاً می‌توان تا سقف فضای VM یا سیستم میزبان از filesystem استفاده کرد.

همچنین در بسیاری از infrastructure providerها، تعداد persistent volumeهایی که می‌توان روی هر node mount کرد محدود است. این تعداد با افزایش حافظه node افزایش نمی‌یابد. بنابراین اگر برای هر Workshop instance به persistent volume نیاز باشد، افزایش حافظه هر node برای جای دادن تعداد بیشتری Workshop روی یک node کمکی نخواهد کرد، زیرا ممکن است persistent volume کافی برای آن تعداد وجود نداشته باشد. به همین دلیل، یک راهنمای کلی این است که از nodeهایی با 32Gi حافظه (به‌جای 64Gi) استفاده کرده و در صورت نیاز تعداد nodeها را افزایش دهید.

همچنین همواره این ریسک وجود دارد که کاربر Workshop تلاش کند نوعی denial of service attack علیه Kubernetes cluster انجام دهد، حتی زمانی که storage اختصاص داده شده است. برای مثال، با نوشتن حجم زیادی داده در بخش دیگری از filesystem مانند ``/tmp``. میزان تأثیر این موضوع بستگی به این دارد که آیا Kubernetes cluster محدودیتی برای استفاده از transient container filesystem توسط هر container اعمال کرده است یا خیر. در صورت نبود محدودیت، ممکن است کل فضای filesystem اختصاص‌یافته به containerها روی node پر شود و سایر applicationهای اجراشده روی همان node نیز تحت تأثیر قرار گیرند.



.. _namespace-resource-budget:

Namespace resource budget
--------------------------

برای هر WorkshopSession یک namespace اختصاصی در Kubernetes cluster ایجاد می‌شود که فقط توسط همان session استفاده می‌شود. این کار به Workshop اجازه می‌دهد تا با استفاده از Kubernetes resources و ابزارهای مرتبط، deploymentهایی را در cluster ایجاد کند. در صورتی که بیش از یک namespace مورد نیاز باشد، این موضوع از طریق workshop definition قابل پیکربندی است.

به‌صورت پیش‌فرض، Educates هیچ محدودیتی بر منابع مصرفی توسط applicationهایی که در session namespace مستقر می‌شوند اعمال نمی‌کند. هر محدودیتی که وجود داشته باشد، وابسته به تنظیمات خود Kubernetes cluster است و در یک Kubernetes cluster استاندارد معمولاً هیچ محدودیتی اعمال نشده است. این بدان معناست که برای محدود کردن میزان منابع قابل‌استفاده، باید یک resource budget در workshop definition تعریف شود.

این کار با override کردن property زیر انجام می‌شود:

``session.namespaces.budget``

نمونه تنظیم در YAML:

.. code-block:: yaml

  spec:
    session:
      namespaces:
        budget: medium

مقدار این property یک t‑shirt size است که به یک ResourceQuota نگاشت می‌شود. این مقدار مشخص می‌کند چه میزان memory و CPU می‌تواند توسط deploymentهای موجود در session namespace مصرف شود. همچنین limit rangeها و مقادیر پیش‌فرض memory و CPU را برای deploymentهایی که خودشان مقدار مشخص نکرده‌اند تعیین می‌کند.

مقادیر پیش‌فرض و limit rangeهای مشخص‌شده برای containerها توسط t‑shirt size قابل override هستند، اما نمی‌توان آن‌ها را خارج از محدوده‌ای که برای pod توسط همان t‑shirt size تعریف شده است تنظیم کرد. بهترین روش این است که هر deployment در session namespace، resource requirementهای خود را مستقیماً در تعریف deployment مشخص کند و به مقادیر پیش‌فرض LimitRange اتکا نکند.

اگر به کنترل دقیق‌تری روی ResourceQuota و LimitRange نیاز باشد، مقدار budget باید برابر با ``custom`` تنظیم شود. در این حالت باید resourceهای ``LimitRange`` و ``ResourceQuota`` به‌عنوان بخشی از ``session.objects`` ارائه شوند.

اگر Workshop از secondary namespaceها استفاده کند، budget مربوط به آن‌ها نیز می‌تواند به همین روش override شود.

**Recommendations**

* اطمینان حاصل کنید که Workshop برای primary session namespace و هر secondary namespace یک budget مشخص کرده باشد. در غیر این صورت، کاربر Workshop می‌تواند تمام منابع cluster را مصرف کند.
* در صورت امکان، deploymentهایی که از داخل Workshop به cluster اعمال می‌شوند باید resource requirementهای container را به‌صورت صریح مشخص کنند و به limit rangeهای پیش‌فرض اتکا نکنند.
* اگر Workshop از budget با مقدار ``custom`` استفاده می‌کند، مطمئن شوید که تعاریف مناسب ``LimitRange`` و ``ResourceQuota`` ارائه شده باشند.
* اگر session namespace برای یک Workshop مورد نیاز نیست، دسترسی به Kubernetes REST API را غیرفعال کنید.




.. _kubernetes-resource-objects:

Kubernetes resource objects
----------------------------

در تعریف Workshop این امکان وجود دارد که Kubernetes resource objectهایی تعریف شوند که به‌صورت مشترک برای workshop environment instance مربوط به آن Workshop ایجاد شوند. همچنین می‌توان Kubernetes resource objectهایی تعریف کرد که به‌صورت اختصاصی برای هر WorkshopSession ایجاد شوند.

برای namespaced Kubernetes resource objectهایی که در بخش ``environment.objects`` تعریف می‌شوند، اگر namespace مشخص نشده باشد، آن‌ها در namespace مشترک workshop environment که بین تمام WorkshopSessionها shared است ایجاد خواهند شد.

برای namespaced Kubernetes resource objectهایی که در بخش ``session.objects`` تعریف می‌شوند، اگر namespace مشخص نشده باشد، آن‌ها در namespace مربوط به همان WorkshopSession ایجاد خواهند شد.

نام namespace مشترک workshop environment برابر با نام همان workshop environment instance است. نام namespace مربوط به یک WorkshopSession خاص نیز شامل نام همان WorkshopSession است. از آنجا که این نام‌ها برای هر workshop environment instance و هر WorkshopSession منحصربه‌فرد هستند، تداخلی با سایر workshop environmentها یا WorkshopSessionها ایجاد نخواهد شد.

اگر cluster scoped Kubernetes resourceهایی را در ``environment.objects`` یا ``session.objects`` تعریف کنید، باید مشابه namespaceها، نامی منحصربه‌فرد برای آن‌ها انتخاب کنید. به عبارت دیگر، نباید از نامی برای cluster scoped Kubernetes resource object استفاده کنید که در بین WorkshopSessionها یا instanceهای مختلف workshop environment تکراری باشد.

در مورد WorkshopSessionها، از آنجا که ممکن است چندین session به‌صورت هم‌زمان برای یک workshop environment وجود داشته باشد، بدیهی است که نمی‌توان برای cluster scoped resourceها از نام یکسان در همه sessionها استفاده کرد.

در مورد workshop environmentها نیز ایجاد cluster scoped Kubernetes resource با نام ثابت می‌تواند با خطا مواجه شود، زیرا ممکن است یک workshop environment instance قبلی مربوط به همان Workshop همچنان وجود داشته باشد. این وضعیت زمانی رخ می‌دهد که TrainingPortal پس از تغییر یک Workshop، یک workshop environment instance جدید ایجاد می‌کند، اما همچنان instance قبلی را نگه می‌دارد تا WorkshopSessionهای در حال اجرا فرصت تکمیل داشته باشند، و سپس آن را حذف می‌کند.

برای cluster scoped Kubernetes resourceهایی که در ``session.objects`` تعریف می‌شوند، جهت تضمین یکتایی (uniqueness) توصیه می‌شود نام resource شامل data variable زیر باشد:

``$(session_name)``

برای cluster scoped Kubernetes resourceهایی که در ``environment.objects`` تعریف می‌شوند، توصیه می‌شود نام resource شامل data variable زیر باشد:

``$(environment_name)``

نمونه تنظیم در YAML:

.. code-block:: yaml

  spec:
    session:
      objects:
      - apiVersion: v1
        kind: Namespace
        metadata:
          name: $(session_name)-extra
    environment:
      objects:
      - apiVersion: v1
        kind: Namespace
        metadata:
          name: $(environment_name)-extra

**Recommendations**

* اطمینان حاصل کنید که cluster scoped Kubernetes resource objectهایی که در مجموعه ``environment.objects`` تعریف می‌شوند، نامی داشته باشند که شامل نام workshop environment باشد.
* اطمینان حاصل کنید که cluster scoped Kubernetes resource objectهایی که در مجموعه ``session.objects`` تعریف می‌شوند، نامی داشته باشند که شامل نام WorkshopSession باشد.
* از استفاده از ``environment.objects`` یا ``session.objects`` برای ایجاد Custom Resource Definition (CRD)هایی که نام آن‌ها باید همواره ثابت باشد، خودداری کنید.




.. _workshop-container-startup:

Workshop container startup
---------------------------

اگر یک Workshop نیاز داشته باشد هنگام startup شدن Workshop container مراحل خاصی را اجرا کند، می‌تواند shell scriptهای اجرایی را در دایرکتوری ``workshop/setup.d`` قرار دهد. این scriptها پس از انجام پیکربندی پایه container و قبل از راه‌اندازی application serviceها یا dashboard اجرا می‌شوند. این scriptها به‌صورت self‑contained اجرا می‌شوند و تنظیم environment variable در داخل آن‌ها روی فرآیندهایی که بعد از آن اجرا می‌شوند تأثیری ندارد.

کاربرد رایج این setup scriptها تولید فایل‌های resource سفارشی برای استفاده در Workshop است که با اطلاعات مخصوص همان session از پیش مقداردهی شده‌اند؛ مانند نام namespace مربوط به session، ingress hostname مورد استفاده و موارد مشابه.

در صورت نیاز، setup scriptها می‌توانند برای دانلود source code نمونه یا binary ابزارهای مورد نیاز Workshop استفاده شوند. با این حال، چون این scriptها در فرآیند startup Workshop container اجرا می‌شوند و برای هر instance جداگانه Workshop تکرار می‌گردند، اجرای طولانی‌مدت آن‌ها باعث تأخیر در startup container خواهد شد. اگر اجرای script بیش از حد طول بکشد، کاربر ممکن است تصور کند Workshop session دچار مشکل شده و آن را ترک کند. همچنین scriptهای طولانی‌مدت می‌توانند با مکانیزم‌های timeout که برای تشخیص ورود کاربر به Workshop session وجود دارند تداخل ایجاد کنند.

به‌شدت توصیه می‌شود که setup scriptها عملیاتی که بیش از ۱۰ ثانیه زمان می‌برد انجام ندهند. بنابراین این scriptها برای دانلود binaryهای بسیار بزرگ مناسب نیستند. به‌عنوان مثال، دانلود یک محیط JDK از طریق setup script عملی نخواهد بود. همچنین نمی‌توان از آن‌ها برای اجرای پیشاپیش یک Java compilation استفاده کرد.

وقتی گفته می‌شود shell script باید executable باشد، یعنی باید execute bit آن تنظیم شده باشد (با دستور ``chmod +x``). اگر فایل به‌عنوان executable علامت‌گذاری نشده باشد، اجرا نخواهد شد.

هرگونه خروجی تولیدشده توسط setup script به‌طور خودکار به فایل ``~/.local/share/workshop/setup-scripts.log`` در داخل Workshop container و همچنین به pod logs مربوط به Workshop افزوده می‌شود. بنابراین نیازی نیست setup script خروجی خود را به‌صورت دستی در فایل log جداگانه ثبت کند.

**Recommendations**

* setup scriptها نباید خروجی خود را به فایل log اختصاصی ثبت کنند، زیرا خروجی آن‌ها به‌طور خودکار در مکان‌های پیش‌فرض ثبت می‌شود.
* setup scriptها باید فقط عملیاتی را انجام دهند که زمان اجرای آن‌ها کوتاه باشد (ترجیحاً کمتر از ۱۰ ثانیه).
* setup scriptها باید به‌گونه‌ای نوشته شوند که در صورت اجرای بیش از یک‌بار (مثلاً پس از restart شدن Workshop container) دچار خطا نشوند.
* setup scriptها نباید application serviceها را در background اجرا کنند؛ این وظیفه بر عهده instance مربوط به ``supervisord`` است.

**Related issues**

به‌دلیل محدودیت‌هایی که در حفظ permission فایل‌ها هنگام unpack شدن archiveها توسط ``vendir`` وجود دارد، scriptهای موجود در ``workshop/setup.d`` در حال حاضر به‌صورت خودکار executable علامت‌گذاری می‌شوند. با این حال، برای سازگاری در آینده، این scriptها همچنان باید به‌صورت دستی executable علامت‌گذاری شوند، در صورتی که این workaround در آینده حذف شود.




.. _docker-resource-requirements:

Docker resource requirements
-----------------------------

زمانی که Docker support فعال باشد، یک instance از ``dockerd`` (با مدل dind - Docker in Docker) به‌صورت side car container در کنار Workshop container اجرا می‌شود. هنگام build یا اجرای Docker imageها در طول Workshop، این عملیات داخل context همان side car container انجام می‌شود و نه در container اصلی Workshop.

این موضوع در مورد ابزارهایی مانند ``pack`` نیز صدق می‌کند. هرچند ``pack`` مستقیماً از دستور docker استفاده نمی‌کند، اما برای انجام عملیات واقعی به instance مربوط به ``dockerd`` که در side car container در حال اجراست متصل می‌شود.

برای جلوگیری از این‌که عملیات build یا اجرای container imageها بر میزان memory در دسترس Workshop container تأثیر بگذارد، برای dockerd side car یک تخصیص memory جداگانه در نظر گرفته شده است. به‌صورت پیش‌فرض مقدار memory برابر با 768Mi است و این مقدار به‌صورت guaranteed در اختیار dockerd قرار می‌گیرد.

اگر Docker buildهایی اجرا شوند که به memory زیادی نیاز دارند (مانند buildهای Java یا NodeJS)، ممکن است لازم باشد مقدار memory اختصاص‌داده‌شده به dockerd side car افزایش یابد. این مقدار را می‌توان در تعریف Workshop با استفاده از تنظیم ``session.applications.docker.memory`` override کرد.

نمونه تنظیم در YAML:

.. code-block:: yaml

  spec:
    session:
      applications:
        docker:
          enabled: true
          memory: 1Gi

به دلیل نیاز احتمالی به فضای موقتی (transient filesystem space) زیاد هنگام build کردن container imageها، و همچنین فضای لازم برای ذخیره imageهای ساخته‌شده یا imageهایی که به instance محلی dockerd pull می‌شوند، همواره یک persistent volume برای dockerd container تخصیص داده می‌شود.

این کار از بروز مشکل کمبود فضای دیسک روی Kubernetes node (در شرایطی که تعداد زیادی Workshop instance هم‌زمان اجرا می‌شوند) جلوگیری می‌کند. با این حال، بسته به اندازه container imageها یا میزان فضای موقتی موردنیاز برای build، ممکن است لازم باشد مقدار storage افزایش یابد.

به‌صورت پیش‌فرض مقدار storage اختصاص‌داده‌شده به dockerd برابر با 5Gi است. این مقدار را می‌توان با استفاده از تنظیم ``session.applications.docker.storage`` در تعریف Workshop override کرد.

نمونه تنظیم:

.. code-block:: yaml

  spec:
    session:
      applications:
        docker:
          enabled: true
          storage: 20Gi

**Recommendations**

* اگر از Docker استفاده نمی‌شود، Docker support را فعال نکنید، زیرا علاوه بر مصرف منابع بیشتر، ریسک امنیتی نیز به همراه دارد.
* اطمینان حاصل کنید که memory کافی به dockerd side car container اختصاص داده شده است.
* اطمینان حاصل کنید که storage کافی به dockerd side car container اختصاص داده شده است.
* تعداد container imageهایی که با Docker pull می‌شوند را به حداقل موردنیاز محدود کنید.
* از تشویق کاربران به اجرای buildهای متوالی خودداری کنید، زیرا هر تغییر منجر به ایجاد layerهای جدید container و افزایش مصرف storage می‌شود.
* کاربران را تشویق کنید قبل از ادامه مراحل Workshop، imageهای غیرضروری را حذف کنند تا فضای storage آزاد شود، به‌جای اینکه صرفاً storage بیشتری تخصیص داده شود.
* کاربران را تشویق کنید containerهای متوقف‌شده را حذف کنند تا فضای storage آزاد شود. به‌عنوان جایگزین می‌توان از گزینه ``--rm`` در دستور ``docker run`` استفاده کرد تا container پس از خروج به‌صورت خودکار حذف شود.

**Related Issues**

در محاسبه resource requirement برای هر Workshop instance، باید memory و storage مربوط به dockerd side car container را به مقادیر مربوط به Workshop container اصلی اضافه کرد.

در مورد memory، چون هر دو container در یک pod اجرا می‌شوند، مجموع memory موردنیاز باید روی node در دسترس باشد تا Workshop instance روی آن schedule شود.

در مورد storage، برای dockerd side car یک persistent volume claim جداگانه استفاده می‌شود. با این حال، چون هر دو container در یک pod قرار دارند، node باید تعداد کافی mount point برای persistent volumeها داشته باشد تا مجموع volumeهای موردنیاز پشتیبانی شود.

توجه داشته باشید که در Kubernetes clusterهای محلی مانند Kind یا Minikube، مقدار storage درخواست‌شده معمولاً نادیده گرفته می‌شود و می‌توان تا سقف فضای فایل‌سیستم VM یا سیستم میزبان از storage استفاده کرد. بنابراین نباید از این محیط‌ها به‌عنوان معیار تعیین storage برای محیط Production استفاده کرد.

فعال‌سازی Docker support همواره یک ریسک امنیتی محسوب می‌شود. از آنجا که dockerd باید در یک privileged container اجرا شود و کاربر نیز قادر به اجرای containerهای privileged از طریق Docker خواهد بود، یک کاربر آگاه می‌تواند به‌راحتی Kubernetes cluster را compromise کند.

در صورت امکان از Docker استفاده نکنید و از روش‌های جایگزین برای build container image استفاده کنید، مانند kaniko یا سیستم‌های native در Java برای build image.

اگر تنها دلیل استفاده از Docker اجرای چند service برای هر Workshop session است، بهتر است آن‌ها را به‌صورت Kubernetes service اجرا کنید. به‌عنوان جایگزین، می‌توان از قابلیت Docker support برای اجرای serviceها از طریق یک snippet پیکربندی docker-compose استفاده کرد. در این روش، به‌صورت پیش‌فرض docker socket داخل Workshop container expose نمی‌شود و ریسک امنیتی کاهش می‌یابد.




.. _docker-container-image-registry:

Docker container image registry
--------------------------------

یک Workshop می‌تواند برای هر WorkshopSession یک container image registry مجزا فعال کند. از این registry می‌توان برای ذخیره container imageهایی استفاده کرد که در طول اجرای دستورالعمل‌های Workshop با ابزارهایی مانند ``docker build``، ``pack``، ``kpack``، ``kaniko`` و غیره ساخته می‌شوند. سپس این container imageها می‌توانند از همان registry به Kubernetes cluster deploy شوند.

container image registry مربوط به هر session، به‌صورت یک deployment مستقل و جدا از deployment مربوط به Workshop instance همان session اجرا می‌شود.

به‌صورت پیش‌فرض، برای ذخیره imageها، یک persistent volume با ظرفیت 5Gi به registry اختصاص داده می‌شود. میزان حافظه اختصاص‌یافته به container image registry نیز به‌صورت پیش‌فرض 768Mi است.

میزان storage و memory را می‌توان با تنظیم propertyهای زیر در تعریف Workshop و در مسیر ``session.applications.registry`` override کرد.

نمونه تنظیم در YAML:

.. code-block:: yaml

  spec:
    session:
      applications:
        registry:
          enabled: true
          memory: 1Gi
          storage: 20Gi

**Recommendations**

* اطمینان حاصل کنید که memory کافی برای per session container image registry اختصاص داده شده است.
* اطمینان حاصل کنید که فضای storage کافی برای per session container image registry اختصاص داده شده است.
* از طراحی Workshop به‌گونه‌ای که کاربران به‌طور مداوم imageهای جدید بسازند و آن‌ها را به registry push کنند، خودداری کنید؛ زیرا هیچ pruning خودکاری برای لایه‌های قدیمی image انجام نمی‌شود و به‌مرور storage مصرف خواهد شد.
* در صورت امکان، به‌جای آنکه deploymentها مستقیماً به imageهای Docker Hub ارجاع دهند، کاربر image را از Docker Hub pull کرده، به per workshop session image registry push کند و سپس از همان registry برای deployment استفاده شود. همچنین توصیه می‌شود برای جلوگیری از rate limiting احتمالی Docker Hub، یک Docker Hub mirror برای هر محیط Workshop در Educates پیکربندی شود. به‌عنوان جایگزین، می‌توان از یک shared OCI image cache برای mirror کردن imageهای موردنیاز در محیط‌های خاص Workshop استفاده کرد.


.. _hosting-images-dockerhub:

Hosting images on Docker Hub
-----------------------------

اگر یک Workshop از یک OCI image شامل دستورالعمل‌های Workshop که روی Docker Hub میزبانی شده استفاده کند، یا docker support فعال باشد و در دستورالعمل‌های Workshop از کاربر خواسته شود یک image را از Docker Hub pull کند و اجرا نماید، یا در یک docker build از imageهای Docker Hub استفاده شود، از آنجا که هر Workshop session منجر به یک pull مجزا از Docker Hub می‌شود، در صورت اجرای هم‌زمان تعداد زیادی Workshop، به‌صورت اجتناب‌ناپذیر به rate limiting اعمال‌شده توسط Docker Hub برخورد خواهید کرد.

در سناریوهایی که imageها مستقیماً اجرا می‌شوند یا در فرآیند build استفاده می‌شوند، برای جلوگیری از rate limiting لازم است سیستم Educates به‌گونه‌ای پیکربندی شود که imageهایی که توسط dockerd pull می‌شوند، از طریق یک image registry pull‑through cache برای Docker Hub mirror شوند. توجه داشته باشید که این تنظیم در تعریف خود Workshop قابل انجام نیست و باید برای کل سیستم Educates در آن cluster فعال شود.

بسیار مهم است که هنگام راه‌اندازی mirror registry، آن را با credentials مربوط به یک Docker Hub account پیکربندی کنید تا محدودیت rate limit افزایش یابد. حتی بهتر است از یک paid account استفاده شود تا سقف rate limit بالاتری اعمال شود یا در برخی پلن‌ها محدودیتی وجود نداشته باشد. استفاده از anonymous access یا free account تضمین نمی‌کند که در صورت اجرای هم‌زمان تعداد زیادی Workshop در یک Kubernetes cluster، همچنان با rate limiting مواجه نشوید.

زمانی که این قابلیت فعال باشد، برای هر Workshopی که docker support در آن فعال است، یک image registry mirror جداگانه ایجاد می‌شود. یک mirror registry مشترک برای کل Workshopهای cluster ایجاد نمی‌شود، زیرا تعیین میزان storage مناسب برای یک mirror registry مشترک بسیار دشوار است.

از نظر storage، mirror registry مربوط به هر Workshop به همان میزان storage پیکربندی می‌شود که dockerd sidecar container استفاده می‌کند (به‌صورت پیش‌فرض 5Gi). از نظر memory نیز mirror registry همان مقدار memory را دریافت می‌کند که dockerd sidecar container استفاده می‌کند (به‌صورت پیش‌فرض 768Mi). این تنظیم بر این فرض استوار است که imageهایی که pull می‌شوند در storage مربوط به dockerd sidecar container ذخیره خواهند شد و بنابراین همین مقدار برای mirror registry نیز کافی خواهد بود.

mirror registry فقط imageهایی را پوشش می‌دهد که از طریق dockerd sidecar container pull می‌شوند. اگر deploymentهایی در Kubernetes cluster مستقیماً به imageهای Docker Hub اشاره کنند، mirror registry دخالتی نخواهد داشت. در این حالت imageها روی nodeهای cluster cache می‌شوند، اما اگر کل cluster از دید Docker Hub به‌عنوان یک IP واحد دیده شود، در صورت اجرای هم‌زمان تعداد زیادی Workshop همچنان احتمال rate limiting وجود دارد.

برای جلوگیری از این مشکل:

- از imageهای Docker Hub استفاده نکنید، یا
- docker support به‌همراه per‑workshop session image registry را فعال کنید.

اگر در دستورالعمل‌های Workshop از کاربر خواسته می‌شود از دستور ``docker pull`` استفاده کند، کاربر باید image را به dockerd محلی pull کرده، سپس آن را به per‑workshop session image registry push کند. سپس deployment در Kubernetes باید به image موجود در per‑workshop session image registry اشاره کند. در این حالت image فقط یک‌بار از طریق mirror registry pull می‌شود و نه برای هر session.

یک سناریوی مشکل‌ساز دیگر مربوط به custom workshop imageهایی است که روی Docker Hub میزبانی می‌شوند. چون این imageها به‌صورت مستقیم در Kubernetes cluster deploy می‌شوند، مستقیماً از Docker Hub pull شده و مشمول rate limiting خواهند شد. در حالت ایده‌آل، این imageها باید روی یک image registry دیگر re‑host شوند و از آنجا مورد استفاده قرار گیرند.

یک راهکار جایگزین برای استفاده از mirror registry مبتنی بر dockerd، استفاده از shared OCI image cache در محیط Workshop است. در این حالت custom workshop image فقط یک‌بار از Docker Hub pull می‌شود و سایر pullها از cache محلی انجام می‌گیرد. البته در این روش باید image referenceها به‌صورت صریح به shared OCI image cache تغییر یابند.

shared OCI image cache می‌تواند با سایر upstream image registryها نیز استفاده شود، اما mirror کردن imageها از registryهای خصوصی یا registryهایی که نیاز به credentials دارند (مانند Docker Hub برای افزایش rate limit) پشتیبانی نمی‌شود. مزیت shared OCI image cache این است که می‌تواند برای OCI imageهایی که شامل دستورالعمل‌های Workshop هستند نیز استفاده شود و محدود به imageهای docker سنتی نیست.

در هر حال، چه برای اجرا، چه در build با dockerd، و چه برای deployment در cluster، همیشه باید از version tag ثابت و تغییرناپذیر استفاده شود. هرگز نباید از tagهایی مانند ``main``، ``master``، ``develop``، ``latest`` یا حتی tagهایی که فقط نسخه major یا major/minor را نشان می‌دهند (بدون patch level) برای imageهای Docker Hub استفاده شود.

دلیل این موضوع این است که اگر image در Docker Hub به‌روزرسانی شود ولی از همان tag قبلی استفاده کند، pull بعدی ممکن است نسخه جدیدی از image را دریافت کند که با نسخه قبلی متفاوت است و این موضوع می‌تواند باعث پر شدن storage مربوط به mirror registry یا shared OCI image cache شود.

به‌طور مشابه، اگر ناچار به استفاده از Docker Hub برای custom workshop image هستید، از tagهایی مانند ``main``، ``master``، ``develop`` یا ``latest`` استفاده نکنید. استفاده از این tagها باعث می‌شود image pull policy در Educates به ``Always`` تنظیم شود، یعنی در صورت هر به‌روزرسانی در Docker Hub، image دوباره pull خواهد شد. حتی مشخص نیست که بررسی وجود نسخه جدید نیز ممکن است به‌عنوان یک image pull محسوب شود و در rate limit Docker Hub لحاظ گردد.

**Recommendations**

* از میزبانی OCI imageهای مربوط به دستورالعمل‌های Workshop روی Docker Hub خودداری کنید.
* از میزبانی custom workshop imageها روی Docker Hub خودداری کنید.
* اگر از imageهای Docker Hub استفاده می‌شود، حتماً image registry pull‑through mirror را برای کل سیستم Educates پیکربندی کنید.
* هنگام استفاده از imageهای Docker Hub، حتماً از version tag ثابت و تغییرناپذیر استفاده کنید.
* در صورت امکان، به‌جای آنکه deploymentها مستقیماً به imageهای Docker Hub اشاره کنند، imageها ابتدا توسط کاربر pull شده و به per‑workshop session image registry push شوند و سپس از آن registry استفاده شوند.
* در صورت امکان، به‌طور کلی از imageهای Docker Hub استفاده نکنید. اگر اجتناب‌ناپذیر است و نمی‌خواهید کاربران imageها را pull و push کنند، imageها را از قبل به یک image registry دیگر (در همان cluster یا خارج از آن) منتقل کنید و دستورالعمل‌های Workshop را طوری تنظیم کنید که از آن registry استفاده شود. در صورت امکان از shared OCI image cache برای این منظور استفاده کنید.

**Related Issues**

اگر کاربران خارج از دستورالعمل‌های Workshop اقدام به pull کردن imageهای دلخواه از Docker Hub کنند، ممکن است نوعی denial of service علیه آن Workshop ایجاد شود، زیرا storage مربوط به mirror registry می‌تواند پر شود. مشخص نیست که docker image registry در حالت mirror به‌صورت خودکار prune انجام می‌دهد یا در شرایط کمبود storage صرفاً درخواست‌ها را عبور می‌دهد و مجدداً مشمول rate limiting می‌شود. در بدترین حالت ممکن است لازم باشد به pod مربوط به mirror registry دسترسی پیدا کرده و به‌صورت دستی imageهای غیرضروری را حذف کنید.

توجه داشته باشید که فقط ``docker pull`` از طریق dockerd sidecar container از mirror registry استفاده می‌کند. ابزارهایی مانند ``dive``، ``skopeo`` یا ``imgpkg`` که خودشان image را pull می‌کنند، از mirror registry استفاده نمی‌کنند و همچنان مشمول rate limiting خواهند بود. به همین دلیل نباید دستورالعمل‌های Workshop را به‌صورت OCI image artifact روی Docker Hub ذخیره کنید، زیرا در این حالت از ``imgpkg`` برای pull و unpack استفاده می‌شود و در ابتدای هر Workshop session مجدداً pull انجام خواهد شد.

همچنین اکیداً توصیه می‌شود credentials شخصی Docker Hub خود را در تعریف Workshop قرار ندهید و از کاربران نخواهید با آن وارد شوند. این کار باعث افشای credentials شما می‌شود و ممکن است کاربران بتوانند به حساب شما وارد شده، password را تغییر دهند یا مشکلات امنیتی دیگری ایجاد کنند.
