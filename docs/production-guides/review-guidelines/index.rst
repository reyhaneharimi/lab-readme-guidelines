Changes to the security policy
==============================

به‌صورت پیش‌فرض، RBAC برای namespaceهای مربوط به sessionهای Workshop طوری تنظیم می‌شود که containerها فقط بتوانند به‌عنوان کاربر non-root اجرا شوند. دلیل این موضوع افزایش ریسک امنیتی در صورت اجازه دادن به کاربران Workshop برای اجرای هر چیزی به‌صورت root است.

در حالت ایده‌آل این سیاست مناسب است، اما بسیاری از container imageهای موجود best practice اجرای به‌صورت non-root را رعایت نکرده‌اند و تنها در صورتی کار می‌کنند که به‌صورت root اجرا شوند. این موضوع به‌ویژه درباره imageهای رسمی Docker Hub صادق است.

به‌عنوان مثال، image مربوط به `nginx` در Docker Hub که اغلب در نمونه‌ deploymentهای Kubernetes استفاده می‌شود، تنها در صورتی به‌درستی اجرا می‌شود که به‌صورت root اجرا شود.

Workshopها می‌توانند policy پیش‌فرض `restricted` که از طریق RBAC اعمال می‌شود را override کنند. این کار با تنظیم مقدار `session.namespaces.security.policy` انجام می‌شود. در حال حاضر می‌توان آن را به مقدار `baseline` تغییر داد تا containerها بتوانند به‌صورت root اجرا شوند.

```yaml
spec:
  session:
    namespaces:
      security:
        policy: baseline
```

به‌طور کلی توصیه می‌شود از هیچ container imageای که نیاز به اجرای root دارد استفاده نشود. در مورد `nginx` می‌توان به‌جای آن از image `bitnami/nginx` استفاده کرد. catalog مربوط به Bitnami imageهای دیگری نیز ارائه می‌دهد که به‌صورت non-root اجرا می‌شوند، در حالی که imageهای رسمی Docker Hub تنها با root اجرا می‌شوند.

در صورت ضرورت مطلق، یک Workshop می‌تواند pod security policy مخصوص خود را تعریف کرده و آن را به یک service account مجزا که توسط یک deployment استفاده می‌شود bind کند تا دسترسی‌های سطح بالاتری بدهد، حتی اجرای privileged container. با این حال این کار به دلیل ریسک امنیتی بسیار بالا باید اجتناب شود، زیرا کاربر Workshop می‌تواند با استفاده از همان service account یک deployment مخرب ایجاد کند و cluster را compromise کند.

**Recommendations**

* در صورت امکان، Workshopها هرگز نباید از container imageهایی استفاده کنند که نیاز به اجرای root دارند.
* در صورتی که ضرورتی وجود ندارد، security policy نباید override شود تا container imageها به‌صورت root اجرا شوند.
* Workshopها نباید خارج از مقادیر پیش‌فرض `restricted` و `baseline` دسترسی elevated بدهند، به‌ویژه اجرای privileged container به دلیل ریسک شدید امنیتی.

**Related Issues**

توجه داشته باشید که زمانی که یک Kubernetes virtual cluster برای یک session فعال می‌شود، security policy مربوط به namespace آن session به‌صورت خودکار از `restricted` به `baseline` تغییر می‌کند. این موضوع برای عملکرد صحیح virtual cluster ضروری است، اما همچنین به این معناست که deploymentهایی که توسط کاربر Workshop در virtual cluster ایجاد می‌شوند می‌توانند به‌صورت root اجرا شوند.

Disabling Kubernetes access
===========================

به‌صورت پیش‌فرض، هر session از Workshop دسترسی به یک namespace در Kubernetes cluster دارد. این امکان برای آن است که به‌عنوان بخشی از Workshop بتوان deploymentهایی در Kubernetes cluster ایجاد کرد. معمولاً کاربر Workshop فقط به namespace اختصاصی همان session دسترسی دارد.

اگر موضوع Workshop به‌گونه‌ای است که کاربر هیچ‌گاه نیازی به deployment مستقیم در Kubernetes cluster ندارد، می‌توان دسترسی به Kubernetes REST API را برای آن Workshop غیرفعال کرد.

این کار با تنظیم مقدار `session.namespaces.security.token.enabled` به `false` در تعریف Workshop انجام می‌شود. در این حالت service account token در container مربوط به Workshop mount نخواهد شد.

```yaml
spec:
  session:
    namespaces:
      security:
        token:
          enabled: false
```

**Recommendations**

* در صورتی که دسترسی به Kubernetes REST API موردنیاز نیست، آن را غیرفعال کنید.

Workshop user admin access
==========================

به‌صورت پیش‌فرض، اگر کاربر Workshop به cluster دسترسی داشته باشد، در namespace مربوط به session خود دارای نقش admin خواهد بود. با این حال، دسترسی cluster admin روی کل Kubernetes cluster ندارد.

می‌توان با override کردن default RBAC rules به کاربر Workshop دسترسی cluster admin داد، اما این کار هرگز نباید در Workshopهایی که روی clusterهای shared اجرا می‌شوند و کاربران عمومی یا غیرقابل‌اعتماد به آن‌ها دسترسی دارند انجام شود.

دسترسی cluster admin تنها زمانی باید استفاده شود که کاربر Workshop، Workshop را روی cluster شخصی خود در سیستم local اجرا می‌کند.

ریسک واضح دادن دسترسی cluster admin این است که کاربر می‌تواند در cluster مشترک به سایر کاربران آسیب بزند، اطلاعات محرمانه را مشاهده کند یا حتی cluster را مختل کند.

اگر Workshop نیاز به سطحی از cluster admin access دارد، می‌توان از virtual cluster استفاده کرد. virtual cluster به هر کاربر Workshop یک Kubernetes cluster مجزا با دسترسی cluster admin می‌دهد، در حالی که همچنان دسترسی به nodeهای underlying UNIX یا کنترل plane اصلی Kubernetes نخواهد داشت.

**Recommendations**

* اطمینان حاصل کنید که Workshopها به کاربر Workshop دسترسی cluster admin نمی‌دهند.
* اطمینان حاصل کنید که service accountهای namespaceهای session که کاربر به آن‌ها دسترسی دارد، هیچ‌گونه elevated privilege نداشته باشند.
* در صورت نیاز به نمایش قابلیت‌های cluster admin، از virtual cluster استفاده کنید.

Workshop container memory
=========================

به‌صورت پیش‌فرض، container مربوط به محیط Workshop دارای 512Mi حافظه است. اگر editor فعال باشد این مقدار به 1Gi افزایش می‌یابد. این حافظه به‌صورت guaranteed رزرو می‌شود و Kubernetes این مقدار را روی node هنگام scheduling رزرو می‌کند.

این مقدار شامل حافظه موردنیاز برای commandهایی است که از command line اجرا می‌شوند. در صورت انجام compilation (به‌ویژه برای برنامه‌های Java)، ممکن است نیاز باشد مقدار حافظه افزایش یابد.

اگر Workshop استفاده زیادی از editor دارد یا کاربر را به بررسی source code تشویق می‌کند، مصرف حافظه editor می‌تواند افزایش یابد. در صورت کمبود حافظه، editor یا applicationهای اجراشده از command line ممکن است با خطای کمبود حافظه مواجه شوند.

مقدار حافظه با استفاده از تنظیم `session.resources.memory` در تعریف Workshop override می‌شود.

```yaml
spec:
  session:
    resources:
      memory: 2Gi
```

**Recommendations**

* اگر editor موردنیاز نیست آن را فعال نکنید.
* اگر Kubernetes web console موردنیاز نیست آن را فعال نکنید.
* میزان حافظه موردنیاز Workshop container را تحلیل کرده و مقدار مناسب را تنظیم کنید.

**Related Issues**

اگر Docker support فعال باشد و در داخل محیط Workshop container image ساخته یا اجرا شود (نه به‌صورت deployment در cluster)، حافظه dockerd جداگانه محاسبه می‌شود و در مقدار بالا لحاظ نمی‌شود.

افزایش اندازه nodeها در Kubernetes cluster برای افزایش حافظه توصیه نمی‌شود، زیرا تعداد persistent volumeهایی که می‌توان روی یک node mount کرد معمولاً محدود است و با افزایش حافظه افزایش نمی‌یابد. راهنمای کلی استفاده از nodeهای با 32Gi حافظه و افزایش تعداد nodeها است.

همچنین حافظه با برابر قرار دادن مقدار `requests` و `limits` در deployment مربوط به Workshop container تضمین می‌شود. در صورت فعال بودن autoscaler ممکن است nodeها تخلیه شوند و session کاربر از ابتدا شروع شود. بنابراین Educates نباید روی clusterهایی که autoscaler آن‌ها می‌تواند تعداد nodeها را کاهش دهد اجرا شود.

Workshop container CPU
======================

به‌صورت پیش‌فرض، Workshop session container هیچ مقدار مشخصی برای CPU تعیین نمی‌کند. در اغلب موارد این مناسب است، زیرا workloadهای داخل Workshop معمولاً CPU intensive یا طولانی‌مدت نیستند.

اگر تعداد زیادی session روی یک node اجرا شوند و هم‌زمان CPU intensive باشند، Kubernetes مصرف CPU را به‌صورت proportional محدود می‌کند، زیرا CPU به‌صورت best effort تخصیص داده می‌شود.

در سناریوهایی مانند compilation با GraalVM که CPU intensive و طولانی هستند، ممکن است sessionها یکدیگر را از CPU محروم کنند.

در این شرایط توصیه می‌شود Workshop روی cluster اختصاصی با nodeهای high CPU اجرا شود و برای Workshop container مقدار CPU request و limit مشخص گردد.

```yaml
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
```

**Recommendations**

* Workshopهای با نیاز CPU بالا باید روی cluster اختصاصی اجرا شوند.
* برای Workshop container مقدار CPU request و limit مشخص شود.

Workshop container storage
==========================

Workshop باید فایل‌های تولیدشده را در home directory کاربر ذخیره کند. به‌صورت پیش‌فرض این فضا transient است و تمام sessionهایی که روی یک node اجرا می‌شوند از همان فضای دیسک استفاده می‌کنند.

اگر Workshop مقدار زیادی binary یا source code دانلود کند یا compilation انجام دهد (به‌ویژه در Java یا NodeJS)، ممکن است فضای دیسک node پر شود و podها evict شوند. در این صورت کاربر باید از ابتدا شروع کند.

برای جلوگیری از این مشکل، می‌توان storage اختصاصی برای home directory کاربر تخصیص داد. در این صورت یک persistent volume claim ایجاد شده و روی home directory mount می‌شود.

مقدار storage با استفاده از تنظیم `session.resources.storage` override می‌شود.

```yaml
spec:
  session:
    resources:
      storage: 5Gi
```

**Recommendations**

* اگر Workshop حجم زیادی فایل تولید می‌کند، storage اختصاصی تعریف کنید.

**Related Issues**

اگر image سفارشی Workshop شامل فایل‌هایی در home directory باشد، persistent volume روی آن mount خواهد شد. در این حالت یک init container ابتدا فایل‌های image را به persistent volume کپی می‌کند.

در clusterهای local مانند Kind یا Minikube مقدار storage درخواست‌شده نادیده گرفته می‌شود و معیار مناسبی برای محیط production نیست.

همچنین تعداد persistent volumeهایی که می‌توان روی یک node mount کرد محدود است و با افزایش حافظه افزایش نمی‌یابد. راهنمای کلی استفاده از nodeهای 32Gi و افزایش تعداد nodeها است.

همیشه این ریسک وجود دارد که کاربر تلاش کند با نوشتن داده زیاد در مسیرهایی مانند `/tmp` حمله denial of service انجام دهد. میزان ریسک بستگی به وجود یا عدم وجود محدودیت فضای filesystem container دارد.

Namespace resource budget
=========================

برای هر session از Workshop، یک namespace در Kubernetes cluster ایجاد می‌شود که فقط توسط همان session استفاده می‌شود. این کار برای آن است که Workshop بتواند با استفاده از Kubernetes resourceها، ابزارها و غیره، deploymentهایی در cluster ایجاد کند. اگر بیش از یک namespace موردنیاز باشد، این موضوع از طریق تعریف Workshop قابل پیکربندی است.

به‌صورت پیش‌فرض، Educates هیچ محدودیتی روی میزان resourceهایی که applicationهای deploy شده در session namespace مصرف می‌کنند اعمال نمی‌کند. هر محدودیتی وابسته به تنظیمات Kubernetes cluster است که در حالت استاندارد معمولاً هیچ محدودیتی ندارد. بنابراین برای محدود کردن مصرف resourceها، باید در تعریف Workshop یک resource budget مشخص شود. این کار با override کردن ویژگی `session.namespaces.budget` انجام می‌شود.

```yaml
spec:
  session:
    namespaces:
      budget: medium
```

مقدار این ویژگی یک t-shirt size است که به یک resource quota برای میزان memory و CPU قابل استفاده توسط deploymentهای داخل session namespace نگاشت می‌شود. همچنین limit rangeها و مقادیر پیش‌فرض memory و CPU را تعیین می‌کند در صورتی که deploymentها خودشان مقدار مشخص نکرده باشند.

مقادیر پیش‌فرض و limit rangeهای تعیین‌شده توسط t-shirt size قابل override هستند، اما نمی‌توان آن‌ها را خارج از محدوده‌ای که برای pod مشخص شده تغییر داد. بهترین practice این است که هر deployment داخل session namespace، resource requirementهای خود را مشخص کند و به مقادیر پیش‌فرض تکیه نکند.

اگر نیاز به کنترل دقیق‌تر روی resource quota و limit range داشته باشید، باید مقدار budget را برابر `custom` قرار دهید و خودتان resourceهای `LimitRange` و `ResourceQuota` را به‌عنوان بخشی از `session.objects` تعریف کنید.

اگر Workshop از namespaceهای ثانویه استفاده می‌کند، budget آن‌ها نیز به همین شکل قابل override است.

**Recommendations**

* اطمینان حاصل کنید که Workshopها برای namespace اصلی session و هر namespace ثانویه budget مشخص می‌کنند، در غیر این صورت کاربر می‌تواند تمام resourceهای cluster را مصرف کند.
* در صورت امکان، deploymentهای انجام‌شده از داخل Workshop باید resource requirementهای container را مشخص کنند.
* اگر budget از نوع `custom` استفاده می‌شود، باید `LimitRange` و `ResourceQuota` مناسب تعریف شده باشد.
* اگر session namespace موردنیاز نیست، دسترسی به Kubernetes REST API غیرفعال شود.

Kubernetes resource objects
===========================

در تعریف Workshop می‌توان Kubernetes resource objectهایی را مشخص کرد که به‌صورت مشترک برای workshop environment ایجاد شوند. همچنین می‌توان resource objectهایی تعریف کرد که برای هر workshop session به‌صورت جداگانه ایجاد شوند.

برای resourceهای namespaced که در `environment.objects` تعریف می‌شوند، اگر namespace مشخص نشده باشد، در namespace مشترک workshop environment ایجاد می‌شوند.

برای resourceهای namespaced که در `session.objects` تعریف می‌شوند، اگر namespace مشخص نشده باشد، در namespace مربوط به همان session ایجاد می‌شوند.

نام namespace مربوط به workshop environment برابر با نام instance آن است. نام namespace مربوط به session شامل نام session خواهد بود. این نام‌ها یکتا هستند و با سایر environmentها یا sessionها تداخل ندارند.

اگر resourceهای cluster-scoped تعریف می‌کنید، باید نام آن‌ها را به‌گونه‌ای انتخاب کنید که یکتا باشد. نباید از نامی استفاده شود که بین sessionهای مختلف یا instanceهای مختلف environment مشترک باشد.

برای `session.objects` توصیه می‌شود نام resource شامل متغیر `$(session_name)` باشد.
برای `environment.objects` توصیه می‌شود نام resource شامل `$(environment_name)` باشد.

```yaml
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
```

**Recommendations**

* resourceهای cluster-scoped در `environment.objects` باید شامل نام environment باشند.
* resourceهای cluster-scoped در `session.objects` باید شامل نام session باشند.
* از environment یا session objects برای ایجاد CRDهایی که باید همیشه یک نام ثابت داشته باشند استفاده نکنید.

Workshop container startup
==========================

اگر Workshop نیاز دارد هنگام start شدن container کار خاصی انجام دهد، می‌تواند scriptهای shell اجرایی در مسیر `workshop/setup.d` قرار دهد. این scriptها بعد از تنظیمات اولیه container و قبل از start شدن serviceها یا dashboard اجرا می‌شوند.

این scriptها self-contained هستند و تنظیم environment variable در آن‌ها روی سایر processها تأثیر نمی‌گذارد.

کاربرد رایج این scriptها ایجاد فایل‌های resource سفارشی است که با اطلاعات session (مانند session name یا ingress hostname) پر شده‌اند.

در صورت نیاز، می‌توان از این scriptها برای download کردن source code یا binaryهای ابزارهای موردنیاز استفاده کرد، اما چون این scriptها هنگام start شدن هر session اجرا می‌شوند، اجرای طولانی آن‌ها باعث تأخیر در start شدن session می‌شود. اگر اجرای آن‌ها طولانی باشد، کاربر ممکن است تصور کند session خراب است.

توصیه اکید می‌شود که این scriptها بیش از 10 ثانیه طول نکشند. بنابراین برای download کردن بسته‌های بزرگ مناسب نیستند.

script باید executable باشد (`chmod +x`). در غیر این صورت اجرا نمی‌شود.

خروجی script به‌صورت خودکار در فایل
`~/.local/share/workshop/setup-scripts.log`
و logهای pod ذخیره می‌شود. نیازی به مدیریت دستی log نیست.

**Recommendations**

* scriptها نباید خروجی خود را جداگانه log کنند.
* زمان اجرای script کمتر از 10 ثانیه باشد.
* scriptها طوری نوشته شوند که در صورت اجرای مجدد دچار خطا نشوند.
* scriptها نباید serviceها را در background اجرا کنند.

Docker resource requirements
============================

در صورت فعال بودن docker support، یک container جانبی dockerd (`dind`) کنار workshop container اجرا می‌شود. عملیات `docker build` و اجرای containerها داخل این sidecar انجام می‌شود.

برای جلوگیری از تأثیر بر memory workshop container، dockerd memory جداگانه دارد. مقدار پیش‌فرض 768Mi است.

در صورت نیاز، مقدار memory با تنظیم
`session.applications.docker.memory`
قابل override است.

```yaml
spec:
  session:
    applications:
      docker:
        enabled: true
        memory: 1Gi
```

به‌دلیل نیاز احتمالی به فضای disk زیاد، یک persistent volume با مقدار پیش‌فرض 5Gi به dockerd اختصاص داده می‌شود. مقدار storage با
`session.applications.docker.storage`
قابل override است.

```yaml
spec:
  session:
    applications:
      docker:
        enabled: true
        storage: 20Gi
```

**Recommendations**

* docker support را در صورت عدم نیاز فعال نکنید.
* memory و storage کافی برای dockerd تعیین کنید.
* از buildهای متوالی غیرضروری خودداری شود.
* کاربران containerهای متوقف‌شده و imageهای غیرضروری را حذف کنند.

Docker container image registry
===============================

Workshop می‌تواند یک container image registry جداگانه برای هر session فعال کند. این registry برای نگهداری imageهای ساخته‌شده با `docker build`، `pack`، `kpack`، `kaniko` و غیره استفاده می‌شود.

به registry یک persistent volume پیش‌فرض 5Gi و memory پیش‌فرض 768Mi اختصاص داده می‌شود.

این مقادیر با
`session.applications.registry`
قابل override هستند.

```yaml
spec:
  session:
    applications:
      registry:
        enabled: true
        memory: 1Gi
        storage: 20Gi
```

**Recommendations**

* memory و storage کافی برای registry تعیین شود.
* از build و push تکراری imageها اجتناب شود.
* در صورت امکان imageها ابتدا به registry session push شوند.

Hosting images on Docker Hub
============================

اگر Workshop از imageهای Docker Hub استفاده کند، به‌دلیل rate limit هنگام اجرای تعداد زیاد session احتمالاً به محدودیت برخورد خواهید کرد.

برای جلوگیری از rate limit باید image registry pull-through cache فعال شود. این تنظیم در سطح کل Educates انجام می‌شود و در تعریف Workshop قابل تنظیم نیست.

mirror registry برای هر Workshop جداگانه ایجاد می‌شود و از همان memory و storage dockerd استفاده می‌کند.

این mirror فقط برای pullهایی که از طریق dockerd انجام می‌شوند کاربرد دارد. deploymentهایی که مستقیم از Docker Hub image می‌گیرند همچنان در معرض rate limit هستند.

بهتر است:

* imageها به registry داخلی push شوند
* از version tag ثابت استفاده شود
* از tagهایی مانند `latest`, `main`, `develop` استفاده نشود

استفاده از shared OCI image cache نیز گزینه‌ای مناسب است.

**Recommendations**

* OCI imageهای Workshop روی Docker Hub میزبانی نشوند.
* imageهای سفارشی روی Docker Hub میزبانی نشوند.
* از version tag ثابت استفاده شود.
* در صورت امکان imageها به registry داخلی منتقل شوند.

**Related Issues**

فقط docker pull از dockerd sidecar از mirror استفاده می‌کند. ابزارهایی مانند `dive`, `skopeo`, `imgpkg` از mirror استفاده نمی‌کنند.

هرگز credentialهای Docker Hub را داخل تعریف Workshop قرار ندهید.
