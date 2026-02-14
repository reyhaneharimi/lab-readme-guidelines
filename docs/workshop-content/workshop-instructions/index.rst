.. _workshop-instructions:

دستورالعمل‌های Workshop
=======================

فایل‌های ماژول تشکیل‌دهنده دستورالعمل‌های Workshop می‌توانند هنگام استفاده از renderer نوع ``classic`` از قالب‌های Markdown یا AsciiDoc استفاده کنند. پسوند فایل باید ``.md`` یا ``.adoc`` باشد، متناسب با نوع قالب انتخاب‌شده.

در صورت استفاده از renderer نوع ``hugo`` فقط فایل‌های Markdown قابل استفاده هستند. همانند استفاده مستقل از Hugo، صفحات می‌توانند به صورت یک فایل منفرد با پسوند ``.md`` یا به صورت page bundle (دایرکتوری شامل فایل ``index.md``) تعریف شوند.

اگر از تصاویر استفاده شود:

- در renderer نوع ``classic`` تصاویر می‌توانند در همان مسیر فایل‌های Markdown یا AsciiDoc قرار گیرند.
- در renderer نوع ``hugo``:
  - اگر از فایل منفرد ``.md`` استفاده شود، تصاویر باید در مسیر ``workshop/static`` قرار گیرند.
  - اگر از page bundle استفاده شود، تصاویر می‌توانند در همان دایرکتوری صفحه قرار گیرند.

--------------------------------------------------------------------

نشانه‌گذاری دستورات اجرایی
----------------------------

علاوه بر Markdown و AsciiDoc استاندارد، می‌توان annotationهایی به code blockها اضافه کرد تا کاربر با کلیک روی آن‌ها دستور را در Terminal اجرا کند.

در Markdown:

```execute
echo "Execute command."
```

با کلیک روی این code block، دستور در اولین Terminal موجود در dashboard اجرا می‌شود.

Terminal انتخاب‌شده فعال باقی می‌ماند و هر ورودی بعدی نیز به همان Terminal ارسال خواهد شد.

در AsciiDoc:

[source,bash,role=execute]
----
echo "Execute command."
----

اگر چند Terminal فعال باشد، می‌توان مشخص کرد دستور در کدام اجرا شود:

```execute-1
echo "Execute command."
```

```execute-2
echo "Execute command."
```

برای اجرای دستور در همه Terminalها:

```execute-all
clear
```

در این حالت Terminal اول پس از اجرا انتخاب باقی می‌ماند.

اگر نیاز به قطع کردن یک دستور در حال اجرا باشد، در روش قدیمی می‌توانستید از:

```execute
<ctrl+c>
```

استفاده کنید.

اما این روش منسوخ شده و باید از action جدید استفاده شود:

```terminal:interrupt
session: 1
```

--------------------------------------------------------------------

نشانه‌گذاری متن قابل کپی
-------------------------

برای کپی محتوا به paste buffer مرورگر:

```copy
echo "Text to copy."
```

پس از کلیک، می‌توان متن را در پنجره‌ای دیگر paste کرد.

اگر متن نیاز به ویرایش قبل از استفاده دارد:

```copy-and-edit
echo "Text to copy and edit."
```

در AsciiDoc:

[source,bash,role=copy]
----
echo "Text to copy."
----

[source,bash,role=copy-and-edit]
----
echo "Text to copy and edit."
----

برای کپی inline در متن (در hugo):

```
Text to ``copy``{{<copy>}}.
```

در renderer نوع ``classic`` به جای آن از:

```
Text to ``copy``{{copy}}.
```

استفاده شود.

--------------------------------------------------------------------

Clickable Actions توسعه‌پذیر (Namespaced)
-------------------------------------------

روش‌های قبلی ``execute`` و ``copy`` همچنان پشتیبانی می‌شوند، اما نسخه جدید با namespace توصیه می‌شود.

به جای:

```execute
echo "Execute command."
```

از این استفاده کنید:

```terminal:execute
command: echo "Execute command."
```

محتوای code block در این حالت YAML است و دستور باید در property با نام ``command`` مشخص شود.

به طور پیش‌فرض در session 1 اجرا می‌شود. برای تعیین session خاص:

```terminal:execute
command: echo "Execute command."
session: 1
```

برای اجرا در همه Terminalها:

```terminal:execute-all
command: echo "Execute command."
```

اگر بخواهید قبل از اجرا Terminal پاک شود:

```terminal:execute
command: echo "Execute command."
clear: true
```

برای قطع کردن اجرای دستور:

```terminal:interrupt
session: 1
```

برای قطع در همه Terminalها:

```terminal:interrupt-all
```

برای وارد کردن ورودی (مثلاً password):

```terminal:input
text: password
```

برای ارسال به session خاص:

```terminal:input
text: password
session: 1
```

برای جلوگیری از افزودن newline:

```terminal:input
text: input
endl: false
```

برای پاک کردن Terminal:

```terminal:clear
session: 1
```

برای پاک کردن همه Terminalها:

```terminal:clear-all
```

این عملیات کل buffer را پاک می‌کند، نه فقط بخش نمایش داده‌شده.

برای کپی متن با روش جدید:

```workshop:copy
text: echo "Text to copy."
```

یا:

```workshop:copy-and-edit
text: echo "Text to copy and edit."
```

مزیت استفاده از این روش نسبت به مکانیزم قدیمی این است که با استفاده از نحو مناسب YAML می‌توانید کنترل کنید که آیا مقدار چندخطی در یک خط ادغام شود یا شکست خط‌ها حفظ شوند، و همچنین تعیین کنید که آیا newline ابتدایی یا انتهایی در نظر گرفته شود یا خیر. در مکانیزم قدیمی، رشته قبل از استفاده همیشه trim می‌شد.

با استفاده از فرم‌های جدید، بلاک کد هنگام نمایش می‌تواند با پیام متفاوتی مشخص شود که نشان دهد چه عملی انجام خواهد شد.

در AsciiDoc نیز روش مشابه است؛ از ``role`` برای نام annotation استفاده می‌شود و محتوای آن YAML خواهد بود:

[source,bash,role=terminal:execute]
----
command: echo "Execute command."
----

.. _clickable-actions-for-the-dashboard:

Clickable actions برای Dashboard
--------------------------------

علاوه بر clickable actionهای مرتبط با Terminal و کپی متن، actionهای دیگری برای کنترل Dashboard و باز کردن URL نیز وجود دارد.

باز کردن URL در یک تب جدید مرورگر:

```dashboard:open-url
url: https://www.example.com/
```

نمایش یک Dashboard tab خاص:

```dashboard:open-dashboard
name: Terminal
```

اگر Dashboard شامل Terminal باشد، focus به صورت خودکار روی آن قرار نمی‌گیرد. برای انتخاب Terminal و قرار دادن focus:

```dashboard:expose-terminal
session: 1
```

ایجاد Dashboard جدید با URL مشخص:

```dashboard:create-dashboard
name: Example
url: https://www.example.com/
```

ایجاد Dashboard جدید با Terminal جدید:

```dashboard:create-dashboard
name: Example
url: terminal:example
```

فرمت باید به صورت ``terminal:<session>`` باشد.
نام session باید شامل حروف کوچک، اعداد و ``-`` باشد.
از نام‌های عددی مانند "1"، "2"، "3" استفاده نکنید چون مربوط به Terminalهای پیش‌فرض هستند.

Reload یک Dashboard موجود:

```dashboard:reload-dashboard
name: Example
```

تغییر URL هنگام Reload:

```dashboard:reload-dashboard
name: Example
url: https://www.example.com/
```

اگر Dashboard وجود نداشته باشد ایجاد می‌شود.

حذف Dashboard:

```dashboard:delete-dashboard
name: Example
```

Dashboardهای پیش‌فرض مانند Terminal، console، editor یا slides قابل حذف نیستند.

--------------------------------------------------------------------

Clickable actions برای Editor
-----------------------------

باز کردن فایل:

```editor:open-file
file: ~/exercises/sample.txt
```

باز کردن فایل و قرار دادن cursor روی خط مشخص:

```editor:open-file
file: ~/exercises/sample.txt
line: 1
```

جستجوی متن دقیق:

```editor:select-matching-text
file: ~/exercises/sample.txt
text: "int main()"
```

جستجو با محدوده قبل و بعد:

```editor:select-matching-text
file: ~/exercises/sample.txt
text: "int main()"
before: 1
after: 1
```

جستجو با Regex:

```editor:select-matching-text
file: ~/exercises/sample.txt
text: "image: (.*)"
isRegex: true
group: 1
```

جایگزینی متن انتخاب‌شده:

```editor:replace-text-selection
file: ~/exercises/sample.txt
text: nginx:latest
```

افزودن خطوط به انتهای فایل:

```editor:append-lines-to-file
file: ~/exercises/sample.txt
text: |
    Lorem ipsum dolor sit amet,
    consectetur adipiscing elit.
```

درج مقدار در YAML:

```editor:insert-value-into-yaml
file: ~/exercises/deployment.yaml
path: spec.template.spec.containers
value:
- name: nginx
  image: nginx:latest
```

اجرای VS Code command:

```editor:execute-command
command: spring.initializr.maven-project
args:
- language: Java
  dependencies: [ "actuator", "webflux" ]
  artifactId: demo
  groupId: com.example
```

--------------------------------------------------------------------

.. _clickable-actions-for-file-download:

Clickable actions برای دانلود فایل
----------------------------------

دانلود فایل:

```files:download-file
path: .kube/config
```

تغییر نام فایل هنگام دانلود:

```files:download-file
path: .kube/config
download: kubeconfig-{{session_name}}
```

نمایش preview:

```files:download-file
path: .kube/config
download: kubeconfig-{{session_name}}
preview: true
```

کپی فایل به clipboard:

```files:copy-file
path: .kube/config
preview: true
```

دانلود از backend سرویس دیگر:

```files:download-file
url: {{ingress_protocol}}://cluster-{{session_name}}.{{ingress_domain}}/config.yaml
download: {{session_name}}-config.yaml
preview: true
```

--------------------------------------------------------------------

.. _clickable-actions-for-file-upload:

Clickable actions برای آپلود فایل
---------------------------------

آپلود یک فایل:

فایلی که قرار است Upload شود باید انتخاب شده و دکمه Upload کلیک شود. فایل حاصل در دایرکتوری ``uploads`` قرار خواهد گرفت که به‌صورت پیش‌فرض زیرشاخه‌ای به نام ``uploads`` در دایرکتوری خانگی کاربر Workshop است. نام فایل پس از Upload همان مقداری خواهد بود که در ویژگی ``path`` مشخص شده است.

برای Upload مجموعه‌ای از فایل‌ها با نام‌های دلخواه، می‌توان به‌جای آن از clickable action با نام ``files:upload-files`` استفاده کرد.

```files:upload-files
```

تمام فایل‌های انتخاب‌شده برای Upload در دایرکتوری ``uploads`` قرار خواهند گرفت و نام آن‌ها همان نام اصلی فایل‌ها در سیستم محلی خواهد بود.

(clickable-actions-for-the-examiner)=
Clickable actions برای examiner
--------------------------------

اگر test examiner فعال باشد، actionهای ویژه‌ای در دسترس هستند که می‌توان از آن‌ها برای اجرای بررسی‌های تأیید (verification checks) استفاده کرد تا مشخص شود آیا کاربر Workshop یک مرحله موردنیاز را انجام داده است یا خیر. این بررسی‌ها می‌توانند با کلیک روی action اجرا شوند، یا به‌صورت اختیاری طوری پیکربندی شوند که هنگام Load شدن صفحه به‌صورت خودکار اجرا شوند.

برای یک بررسی تک‌مرحله‌ای که باید با کلیک اجرا شود، می‌توانید از نمونه زیر استفاده کنید:

```examiner:execute-test
name: test-that-pod-exists
title: Verify that pod named "one" exists.
args:
- one
```

فیلد ``title`` به‌عنوان عنوان clickable action نمایش داده می‌شود و باید ماهیت تست را توضیح دهد. در صورت نیاز می‌توانید فیلد ``description`` را برای توضیح طولانی‌تر اضافه کنید. این توضیح در بدنه clickable action نمایش داده می‌شود اما همیشه به‌صورت متن preformatted خواهد بود.

باید یک برنامه executable (اسکریپت یا برنامه کامپایل‌شده) در مسیر ``workshop/examiner/tests`` وجود داشته باشد که نام آن با مقدار فیلد ``name`` مطابقت داشته باشد.

لیست آرگومان‌هایی که در فیلد ``args`` مشخص شده‌اند به برنامه تست ارسال خواهند شد.

برنامه executable مربوط به تست باید در صورت موفقیت با status کد 0 خارج شود و در صورت شکست با مقدار غیر صفر خارج شود. تست باید تا حد امکان سریع اجرا شود و نباید یک برنامه دائمی (persistent) باشد.

```bash
#!/bin/bash
kubectl get pods --field-selector=status.phase=Running -o name | egrep -e "^pod/$1$"
if [ "$?" != "0" ]; then
    exit 1
fi
exit 0
```

دایرکتوری کاری فعلی برنامه هنگام اجرا، دایرکتوری خانگی کاربر Workshop خواهد بود. با این حال توصیه می‌شود در صورت نیاز مسیرهای absolute با استفاده از متغیر محیطی ``HOME`` ساخته شوند.

به‌صورت پیش‌فرض، برنامه تست پس از 15 ثانیه timeout به‌صورت خودکار متوقف (killed) می‌شود و تست ناموفق در نظر گرفته خواهد شد. در صورت نیاز می‌توانید مقدار ``timeout`` را تنظیم کنید. مقدار آن بر حسب ثانیه است. مقدار 0 باعث استفاده از timeout پیش‌فرض خواهد شد. امکان غیرفعال کردن توقف خودکار برنامه در صورت طولانی شدن اجرا وجود ندارد.

```examiner:execute-test
name: test-that-pod-exists
title: Verify that pod named "one" exists
args:
- one
timeout: 5
```

اگر بخواهید تست در صورت شکست چندین بار تکرار شود، می‌توانید تعداد retry و فاصله بین retryها را مشخص کنید. مقدار delay بر حسب ثانیه است.

```examiner:execute-test
name: test-that-pod-exists
title: Verify that pod named "one" exists
args:
- one
timeout: 5
retries: 10
delay: 1
```

در صورت استفاده از retry، به محض اینکه برنامه تست اعلام کند موفق بوده است، اجرای تکرارها متوقف خواهد شد.

اگر بخواهید retryها تا زمانی که صفحه دستورالعمل‌های Workshop نمایش داده می‌شود ادامه داشته باشند، می‌توانید مقدار ``retries`` را برابر مقدار ویژه YAML یعنی ``.INF`` قرار دهید.

```examiner:execute-test
name: test-that-pod-exists
title: Verify that pod named "one" exists
args:
- one
timeout: 5
retries: .INF
delay: 1
```

اگر به‌جای اجرای تست در context کانتینر Workshop با استفاده از اسکریپت موجود، بخواهید بررسی توسط یک backend service جداگانه مرتبط با Workshop session انجام شود، می‌توانید ویژگی ``url`` را مشخص کنید. پیاده‌سازی منطق تست بر عهده آن سرویس خواهد بود.

```examiner:execute-test
name: test-that-pod-does-not-exist
title: Verify that pod named "one" does not exist
url: {{ingress_protocol}}://examiner-{{session_name}}.{{ingress_domain}}/test-that-pod-does-not-exist
args:
- {{session_name}}
```

در صورت استفاده از ``url``، به دلیل محدودیت‌های cross domain، نام host باید همان parent domain مربوط به ingress domainی باشد که Educates برای آن پیکربندی شده است.

در مواردی که تست نیاز به ورودی کاربر دارد، می‌توان مجموعه‌ای از پارامترهای ورودی را با استفاده از بخش ``inputs`` مشخص کرد.

```examiner:execute-test
name: deploy-application
prefix: Task
title: Deploy application
inputs:
  schema:
    name:
      type: string
      title: "Name:"
      default: "my-app"
      required: true
    replicas:
      type: integer
      title: "Replicas:"
      default: "1"
      required: true
  form:
  - "*"
  - type: submit
    title: Deploy
```

فرمت داده‌هایی که در بخش ``inputs`` ارائه می‌شود مطابق با بسته [jsonform](https://github.com/jsonform/jsonform/wiki) است که برای رندر فرم HTML استفاده می‌شود. توجه داشته باشید که همه انواع inputهای HTML قابل استفاده نیستند. برای مثال، این مکانیزم برای Upload فایل قابل استفاده نیست و باید از clickable actionهای جداگانه برای Upload فایل استفاده شود. همچنین ممکن است همه قابلیت‌های بسته ``jsonform`` قابل استفاده نباشند. برای مثال، قابلیت‌هایی که نیاز به ارائه کد Javascript دارند قابل استفاده نیستند.

زمانی که از inputs استفاده شود، داده‌ها به‌صورت JSON از طریق standard input به برنامه تست ارسال می‌شوند. این داده‌ها می‌توانند در یک فایل ذخیره شده و با ابزارهایی مانند ``jq`` پردازش شوند.

```shell
#!/bin/bash
CONFIG=$HOME/exercises/config.json
cat - >$CONFIG
NAME=$(jq -r -e ".name" $CONFIG)
REPLICAS=$(jq -r -e ".replicas" $CONFIG)
```

اگرچه در ابتدا این قابلیت برای پیاده‌سازی تست‌هایی طراحی شده بود که بررسی کنند آیا کاربر Workshop مراحل قبلی را با موفقیت انجام داده است یا خیر، اما استفاده از inputs این امکان را فراهم می‌کند که از این مکانیزم برای ایجاد quiz، جمع‌آوری ورودی‌هایی که در مراحل بعدی Workshop استفاده می‌شوند، یا اجرای وظایف اسکریپتی پیچیده‌تر که نیاز به ورودی دارند نیز استفاده شود.

توجه داشته باشید که استفاده از قابلیت اجرای خودکار تست هنگام Load شدن صفحه یا باز شدن یک section، نباید برای تست‌هایی که نیاز به ورودی کاربر دارند استفاده شود، زیرا در این صورت کاربر فرصت وارد کردن داده‌های موردنیاز را نخواهد داشت.

Clickable actions برای sectionها
--------------------------------

برای دستورالعمل‌هایی که اختیاری هستند یا می‌خواهید تا زمانی که کاربر Workshop آماده اجرای آن بخش نشده پنهان بمانند، می‌توانید sectionهایی تعریف کنید که در ابتدا به‌صورت collapse و مخفی باشند. با کلیک روی action مربوط به section، محتوای آن باز (expand) می‌شود. برای مثال می‌توان از این قابلیت برای مخفی کردن مجموعه‌ای از سوالات یا یک تست در انتهای هر صفحه از دستورالعمل‌های Workshop استفاده کرد.

برای مشخص کردن بخشی از محتوا که در ابتدا باید مخفی باشد، لازم است از دو بلاک کد action جداگانه برای مشخص کردن شروع و پایان section استفاده کنید.

```section:begin
title: Questions
```

To show you understand ...

```section:end
```

مقدار ``title`` باید متنی باشد که می‌خواهید در بنر clickable action نمایش داده شود.

تنها برای بخش begin یک clickable action نمایش داده می‌شود و action مربوط به پایان section همیشه مخفی خواهد بود. با کلیک روی action مربوط به begin، section باز می‌شود. این section با کلیک مجدد قابل collapse شدن است.

در صورت تمایل، می‌توان sectionهای تو در تو (nested) ایجاد کرد، اما باید برای actionهای begin و end نام مشخص کنید تا به‌درستی با هم match شوند.

```section:begin
name: questions
title: Questions
```

To show you understand ...

```section:begin
name: question-1
prefix: Question
title: 1
```

...

```section:end
name: question-1
```

```section:end
name: questions
```

ویژگی ``prefix`` به شما اجازه می‌دهد مقدار پیش‌فرض ``Section`` که در عنوان action استفاده می‌شود را تغییر دهید.

اگر یک collapsible section شامل یک examiner action block باشد و آن action طوری تنظیم شده باشد که به‌صورت خودکار اجرا شود، تنها زمانی شروع به اجرا خواهد کرد که آن collapsible section باز (expanded) شود.

در صورتی که بخواهید یک section header با همان استایل سایر clickable actionها نمایش داده شود، می‌توانید از نمونه زیر استفاده کنید:

```section:heading
title: Questions
```

کلیک روی این action همچنان باعث می‌شود به‌عنوان completed علامت‌گذاری شود، اما هیچ action دیگری را اجرا نخواهد کرد.

(automatically-triggering-actions)=
اجرای خودکار actionها
----------------------

به‌جای اینکه کاربر Workshop مجبور باشد روی یک clickable action کلیک کند، می‌توانید با تنظیم مقدار ``autostart`` برابر ``true`` باعث شوید action به‌صورت خودکار بلافاصله پس از Load شدن صفحه، یا هنگام باز شدن section حاوی آن، اجرا شود.

برای مثال، در صورت استفاده از clickable action مربوط به examiner test، می‌توانید از نمونه زیر استفاده کنید:

```examiner:execute-test
name: test-that-pod-exists
title: Verify that pod named "one" exists
args:
- one
timeout: 5
retries: .INF
delay: 1
autostart: true
```

اگر پس از موفقیت یک تست بخواهید clickable action بعدی در همان صفحه نیز به‌صورت خودکار اجرا شود، می‌توانید مقدار ``cascade`` را برابر ``true`` قرار دهید. این action بعدی می‌تواند یک تست دیگر یا هر نوع clickable action دیگر باشد.

```examiner:execute-test
name: test-that-pod-exists
title: Verify that pod named "one" exists
args:
- one
timeout: 5
retries: .INF
delay: 1
autostart: true
cascade: true
```

```examiner:execute-test
name: test-that-pod-does-not-exist
title: Verify that pod named "one" does not exist
args:
- one
retries: .INF
delay: 1
```

(overriding-action-cooldown-period)=
تغییر دوره cooldown برای action
---------------------------------

برای اغلب clickable actionها یک دوره cooldown به مدت 3 ثانیه اعمال می‌شود. این بدان معناست که کاربر تا پایان این بازه زمانی نمی‌تواند همان clickable action را مجدداً کلیک کند. هدف از این محدودیت جلوگیری از مشکلات ناشی از double click تصادفی است.

در صورت تمایل می‌توانید این دوره cooldown را override کنید تا کاربر برای مدت طولانی‌تری نتواند از action استفاده کند. این کار با تنظیم ویژگی ``cooldown`` انجام می‌شود. مقدار آن باید بر حسب ثانیه باشد. همچنین می‌توانید از مقدار ویژه ``.INF`` استفاده کنید تا از کلیک مجدد به‌طور کامل جلوگیری شود.

```examiner:execute-test
name: test-that-pod-exists
title: Verify that pod named "one" exists.
cooldown: .INF
args:
- one
```

توجه داشته باشید که Reload شدن صفحه وب حاوی دستورالعمل‌ها باعث reset شدن دوره cooldown می‌شود و clickable action دوباره قابل استفاده خواهد بود.

(hiding-clickable-actions-from-view)=
مخفی کردن clickable actionها
-----------------------------

به‌جز clickable action مربوط به پایان یک collapsible section و actionهایی که داخل sectionهای collapse‌شده قرار دارند، سایر clickable actionها همواره قابل مشاهده هستند و کاربران می‌توانند روی آن‌ها کلیک کنند.

در مورد action سطح بالا که ویژگی ``autostart`` را فعال کرده است، هنگام Load شدن صفحه به‌صورت خودکار اجرا خواهد شد، گویی کاربر روی آن کلیک کرده است. به همین ترتیب، actionی که داخل یک section قرار دارد و ``autostart`` آن فعال است، هنگام باز شدن section اجرا خواهد شد. همچنین actionی که بعد از action دارای ``cascade`` قرار دارد، پس از تکمیل موفق action قبلی اجرا می‌شود.

در شرایطی که actionها به‌صورت خودکار اجرا می‌شوند و نیازی به کلیک کاربر نیست، ممکن است بخواهید آن action را مخفی کنید. این کار با تنظیم ویژگی ``hidden`` در بدنه YAML مربوط به clickable action انجام می‌شود.

برای مثال، می‌توان از این قابلیت استفاده کرد تا هر بار که صفحه Load می‌شود، یک dashboard tab ایجاد شده و focus بگیرد:

```dashboard:reload-dashboard
name: Example
url: https://www.example.com/
autostart: true
hidden: true
```

همچنین می‌توان از ویژگی ``hidden`` استفاده کرد تا متنی تنها پس از تکمیل موفق یک action دارای cascade قابل مشاهده شود، بدون اینکه collapsible section قابل مشاهده باشد. برای این کار باید ویژگی ``hidden`` را روی clickable action مربوط به شروع section تنظیم کنید.

```section:begin
hidden: true
```

Text.

```section:end
```

(generating-events-for-actions)=
تولید event برای actionها
-------------------------

برای هر clickable action که در بدنه آن YAML استفاده می‌شود، می‌توانید مشخص کنید که هنگام کلیک روی action یک analytics event تولید شده و به webhook مربوط به registry analytics ارسال شود. این کار با افزودن فیلد ``event`` انجام می‌شود.

```dashboard:open-url
url: https://www.example.com/
event: "open-example-web-site"
```

مقدار فیلد ``event`` می‌تواند یک مقدار literal مانند یک string باشد، یا یک object شامل ویژگی‌های تو در تو تعریف کند.

```dashboard:open-url
url: https://www.example.com/
event:
  name: "dashboard:open-url"
```

انتخاب مقدار مناسب برای ``event`` به سیستم شما برای پردازش eventها بستگی دارد.

نوع analytics event ارسال‌شده توسط webhook برابر با ``Action/Event`` خواهد بود.

Overriding title and description
--------------------------------

به‌صورت پیش‌فرض، clickable action blockها عنوانی با prefix متناسب با نوع action خواهند داشت. همچنین بدنه action نیز مقدار پیش‌فرض متناسب با عملکرد آن action خواهد داشت.

به‌ویژه در سناریوهای پیچیده شامل ویرایش فایل‌ها، مقادیر پیش‌فرض ممکن است مناسب نباشند یا باعث سردرگمی شوند. در این حالت می‌توانید آن‌ها را override کنید.

برای این کار می‌توانید فیلدهای ``prefix``، ``title`` و ``description`` را در clickable action block تنظیم کنید.

```action:name
prefix: Prefix
title: Title
description: Description
```

در این حالت بنر action block به صورت "Prefix: Title" نمایش داده می‌شود و بدنه آن "Description" را نشان خواهد داد.

توجه داشته باشید که ``description`` همواره به‌صورت متن pre-formatted در صفحه رندر شده نمایش داده می‌شود.

Escaping of code block content
------------------------------

در صورت استفاده از renderer نوع ``classic``، موتور قالب‌سازی [Liquid](https://www.npmjs.com/package/liquidjs) روی محتوای Workshop اعمال می‌شود. اگر لازم باشد محتوایی در code block نمایش داده شود که با عناصر نحوی Liquid تداخل دارد، باید پردازش Liquid را برای آن بخش غیرفعال کنید تا به‌درستی رندر شود. این کار با استفاده از بلوک Liquid به شکل ``{% raw %}...{% endraw %}`` انجام می‌شود.

```
{% raw %}
```execute
echo "Execute command."
```
{% endraw %}
```

توجه داشته باشید که این کار باعث جلوگیری از interpolation متغیرهای داده نیز می‌شود، بنابراین فقط در محدوده موردنیاز از آن استفاده کنید.

در صورت استفاده از renderer نوع ``hugo``، اگر syntax مربوط به Hugo short codeها با محتوایی که می‌خواهید در code block نمایش دهید تداخل داشته باشد، باید در داخل delimiterهای shortcode از کامنت‌های C-style استفاده کنید.

```
{{</* highlight python */>}}
  if hello:
    print("World")
{{</* /highlight */>}}
```
درون‌یابی متغیرهای داده
-----------------------

هنگام ایجاد محتوای صفحه، می‌توانید به تعدادی از متغیرهای داده از پیش تعریف‌شده ارجاع دهید. مقدار این متغیرهای داده هنگام رندر شدن صفحه در مرورگر کاربر در محتوا جایگزین خواهد شد.

محیط Workshop متغیرهای داده داخلی زیر را برای استفاده در دستورالعمل‌های Workshop فراهم می‌کند:

* ``assets_repository`` - نام میزبان مخزن assets محیط Workshop در صورت فعال بودن.
* ``cluster_domain`` - دامنه داخلی مورد استفاده توسط Kubernetes cluster، معمولاً ``cluster.local``.
* ``config_password`` - مقدار رمز عبور تصادفی یکتا برای استفاده هنگام دسترسی به پیکربندی session.
* ``environment_name`` - نام محیط Workshop.
* ``image_repository`` - نام میزبان مخزن image مرتبط با cluster یا training portal برای ذخیره imageها.
* ``ingress_class`` - کلاس ingress که Educates برای همه ingressها استفاده می‌کند.
* ``ingress_domain`` - دامنه‌ای که باید در hostnameهای تولیدشده برای ingress routeها جهت نمایش applicationها استفاده شود.
* ``ingress_port`` - شماره پورت ingress مربوط به Workshop session، معمولاً 80 یا 443، اما در استقرار docker ممکن است متفاوت باشد.
* ``ingress_port_suffix`` - شماره پورت (با پیشوند :) برای ingress. در صورت استفاده از پورت‌های استاندارد 80 یا 443 مقدار آن خالی خواهد بود.
* ``ingress_protocol`` - پروتکل (http/https) مورد استفاده برای ingress routeهای ایجادشده برای Workshop.
* ``kubernetes_api_url`` - اگر session به Kubernetes cluster دسترسی داشته باشد، URL مربوط به Kubernetes API. فقط در terminal معتبر است.
* ``kubernetes_ca_crt`` - محتوای گواهی عمومی مورد نیاز برای دسترسی به Kubernetes API.
* ``kubernetes_token`` - توکن دسترسی Kubernetes مربوط به service account که session با آن اجرا می‌شود.
* ``oci_image_cache`` - نام میزبان OCI image cache محیط Workshop در صورت فعال بودن.
* ``pathway_name`` - نام pathway مورد استفاده در دستورالعمل‌های Workshop.
* ``platform_arch`` - معماری CPU کانتینر Workshop، یعنی ``amd64`` یا ``arm64``.
* ``policy_engine`` - نام موتور policy امنیتی اعمال‌شده، معمولاً ``kyverno``.
* ``policy_name`` - نام policy امنیتی محدودکننده نوع workloadهای قابل استقرار.
* ``services_password`` - مقدار رمز عبور تصادفی یکتا برای استفاده در سرویس‌های دلخواه.
* ``session_hostname`` - نام میزبان instance مربوط به session.
* ``session_id`` - شناسه کوتاه session. فقط در context محیط مربوطه یکتا است.
* ``session_name`` - نام session. در context Kubernetes cluster یکتا است.
* ``session_namespace`` - نام namespace مرتبط با session در cluster اشتراکی.
* ``session_url`` - URL کامل دسترسی به dashboard مربوط به session.
* ``ssh_private_key`` - بخش خصوصی کلید SSH یکتای تولیدشده برای session.
* ``ssh_public_key`` - بخش عمومی کلید SSH یکتای تولیدشده برای session.
* ``storage_class`` - storage class مورد استفاده توسط Educates.
* ``training_portal`` - نام training portal میزبان Workshop.
* ``workshop_description`` - توضیحات Workshop از تعریف آن.
* ``workshop_name`` - نام Workshop.
* ``workshop_namespace`` - نام namespace مربوط به محیط Workshop.
* ``workshop_title`` - عنوان Workshop از تعریف آن.

توجه داشته باشید که ``session_name`` از نسخه 2.6.0 افزوده شده است. در نسخه‌های قبل از آن از ``session_namespace`` به‌عنوان شناسه عمومی session استفاده می‌شد، در حالی که در عمل نشان‌دهنده namespace مرتبط با session بود. از آنجا که Educates از سناریوهایی پشتیبانی می‌کند که در آن‌ها ممکن است دسترسی به Kubernetes cluster وجود نداشته باشد یا cluster جداگانه‌ای با دسترسی کامل admin استفاده شود، این نام‌گذاری منطقی نبود و بنابراین ``session_name`` اضافه شد. اگر به شناسه نام session نیاز دارید از ``session_name`` استفاده کنید و فقط زمانی از ``session_namespace`` استفاده کنید که لازم است به namespace واقعی در Kubernetes cluster اشاره شود. در حال حاضر مقدار هر دو یکسان است، اما در آینده ممکن است ``session_namespace`` در صورت نبود namespace مرتبط مقدار خالی داشته باشد.

برای استفاده از یک متغیر داده در محتوای صفحه، در renderer نوع ``classic`` آن را داخل براکت‌های دوتایی قرار دهید:

```text
{{ session_name }}
```

این کار را می‌توان داخل code blockها، clickable actionها و همچنین در URLها انجام داد:

```dashboard:open-url
url: http://myapp-{{ session_name }}.{{ ingress_domain }}
```

در صورت استفاده از renderer نوع ``hugo`` باید از shortcode نوع ``params`` استفاده کنید:

```text
{{< param session_name >}}
```

این نیز در clickable actionها قابل استفاده است:

```dashboard:open-url
url: http://myapp-{{< param session_name >}}.{{< param ingress_domain >}}
```

توجه داشته باشید که در نسخه‌های قدیمی renderer نوع ``classic`` لازم بود متغیرهای داده با کاراکتر ``%`` در دو طرف احاطه شوند. این روش همچنان برای سازگاری با نسخه‌های قبلی پشتیبانی می‌شود، اما توصیه می‌شود از براکت‌های دوتایی استفاده کنید. پشتیبانی از درصدها ممکن است در نسخه‌های آینده حذف شود.

افزودن متغیرهای داده سفارشی
---------------------------

می‌توانید متغیرهای داده سفارشی خود را با افزودن آن‌ها به فایل‌های پیکربندی Workshop معرفی کنید.

در renderer نوع ``classic`` این کار در فایل ``workshop/modules.yaml`` انجام می‌شود.

یک متغیر داده دارای مقدار پیش‌فرض تعریف می‌شود، اما اگر متغیر محیطی هم‌نام آن وجود داشته باشد، مقدار آن override خواهد شد.

متغیرهای داده باید در بخش ``config.vars`` تعریف شوند:

```yaml
config:
    vars:
    - name: NAME
      value: undefined
```

اگر بخواهید نام متغیر داده با نام متغیر محیطی متفاوت باشد، می‌توانید از ``aliases`` استفاده کنید:

```yaml
config:
    vars:
    - name: NAME
      value: undefined
      aliases:
      - ALIAS
```

ابتدا متغیرهای محیطی تعریف‌شده در aliases بررسی می‌شوند، سپس متغیر هم‌نام با متغیر داده. اگر هیچ‌کدام تعریف نشده باشند، مقدار پیش‌فرض استفاده می‌شود.

مقدار پیش‌فرض یک متغیر داده می‌تواند برای یک Workshop خاص override شود. برای مثال در فایل ``workshop/workshop-python.yaml``:

```yaml
vars:
    NAME: python
```

اگر به کنترل بیشتری نیاز دارید، می‌توانید فایل ``workshop/config.js`` را ایجاد کنید:

```javascript
function initialize(workshop) {
    workshop.load_workshop();
    if (process.env['WORKSHOP_FILE'] == 'workshop-python.yaml') {
        workshop.data_variable('NAME', 'python');
    }
}
exports.default = initialize;
module.exports = exports.default;
```

از آنجا که این کد Javascript است، می‌توانید هر منطقی برای خواندن متغیرهای محیطی و تنظیم متغیرهای داده پیاده‌سازی کنید، حتی تولید مقادیر ترکیبی یا دریافت داده از یک میزبان remote.

در renderer نوع ``hugo`` می‌توان متغیرهای داده سفارشی را در فایل ``workshop/config.yaml`` تعریف کرد:

```yaml
params:
- name: NAME
  value: undefined
  aliases:
  - ALIAS
```

و همچنین در یک pathway خاص:

```yaml
pathways:
  default: python
  paths:
    python:
      title: "Python"
      params:
      - name: NAME
        value: python
```

انتقال متغیرهای محیطی
----------------------

انتقال متغیرهای محیطی، از جمله تغییر نام (remap) آن‌ها، از طریق تعریف custom data variable قابل انجام است. اگر از renderer نوع ``classic`` استفاده می‌کنید و نیازی به تعیین مقدار پیش‌فرض یا تغییر نام متغیر محیطی ندارید، می‌توانید مستقیماً نام متغیر محیطی را استفاده کنید؛ با این شرط که هنگام استفاده، نام آن را با پیشوند ``ENV_`` بنویسید.

برای مثال، اگر بخواهید مقدار متغیر محیطی ``KUBECTL_VERSION`` را در محتوای Workshop نمایش دهید، می‌توانید از ``ENV_KUBECTL_VERSION`` استفاده کنید، مانند:

```
{{ ENV_KUBECTL_VERSION }}
```

توجه داشته باشید که تنها متغیرهای محیطی‌ای که در تعریف Workshop روی container تنظیم شده‌اند، یا متغیرهایی که در اسکریپت‌های `profile.d` تنظیم و export شده‌اند، هنگام رندر شدن دستورالعمل‌های Workshop در دسترس هستند. هر متغیر محیطی‌ای که در `workshop/profile` تنظیم شود در دسترس نخواهد بود، زیرا آن فایل فقط روی terminal تعاملی اثر می‌گذارد.

در صورت استفاده از renderer نوع ``hugo``، معادل داخلی (builtin) برای این قابلیت وجود ندارد، اما در صورت تمایل می‌توانید shortcode سفارشی خود را برای Hugo پیاده‌سازی کنید.

Handling of embedded URL links
------------------------------

در محتوای Workshop می‌توان URL درج کرد. این URL می‌تواند به‌صورت literal یا با syntax مربوط به Markdown یا AsciiDoc برای درج و برچسب‌گذاری لینک باشد. رفتار هنگام کلیک روی URL به نوع آن بستگی دارد.

اگر URL مربوط به یک وب‌سایت خارجی باشد، با کلیک روی آن در یک تب یا پنجره جدید مرورگر باز خواهد شد.

اگر URL یک صفحه نسبی باشد که به صفحه‌ای دیگر در همان محتوای Workshop اشاره کند، آن صفحه جایگزین صفحه فعلی خواهد شد.

می‌توانید URLی تعریف کنید که بخش‌هایی از آن از data variableها تشکیل شده باشد. data variableهای مفید در این زمینه شامل ``session_name`` و ``ingress_domain`` هستند، زیرا می‌توان از آن‌ها برای ساخت URL مربوط به یک application که از داخل Workshop deploy شده استفاده کرد.

در صورت استفاده از renderer نوع ``classic``:

```
https://myapp-{{ session_name }}.{{ ingress_domain }}
```

در صورت استفاده از renderer نوع ``hugo``:

```
https://myapp-{{< param session_name >}}.{{< param ingress_domain >}}
```

اگر Workshop instance به‌عنوان proxy برای یک web application تنظیم شده باشد، لازم است از data variable به نام ``ingress_protocol`` نیز استفاده کنید، که مشخص‌کننده HTTP protocol scheme مورد استفاده برای دسترسی به session Workshop است.

```
{{< param ingress_protocol >}}://myapp-{{< param session_name >}}.{{< param ingress_domain >}}
```

این موضوع ضروری است زیرا نوع نصب Educates تعیین می‌کند که ``https`` یا ``http`` استفاده شود و به‌عنوان نویسنده Workshop از قبل نمی‌توانید این موضوع را بدانید.

Conditional rendering of content
--------------------------------

در صورت استفاده از renderer نوع ``classic``، چون رندر صفحات تا حدی با استفاده از موتور قالب‌سازی [Liquid](https://www.npmjs.com/package/liquidjs) انجام می‌شود، می‌توانید از ساختارهای شرطی پشتیبانی‌شده توسط آن برای نمایش محتوای شرطی استفاده کنید.

```
{% if LANGUAGE == 'java' %}
....
{% endif %}
{% if LANGUAGE == 'python' %}
....
{% endif %}
```

در صورت استفاده از renderer نوع ``hugo``، چون Hugo روش استانداردی برای شرط‌ها ندارد، لازم است از shortcodeهای سفارشی استفاده شود.

برای درج محتوای شرطی بر اساس pathway مورد استفاده در Workshop، Educates یک shortcode فراهم کرده است:

```
{{< pathway python >}}
....
{{< /pathway >}}
{{< pathway java >}}
....
{{< /pathway >}}
```

اگر به بخش‌های شرطی بر اساس متغیرهای دیگر یا منطق پیچیده‌تری نیاز دارید، باید shortcode سفارشی خود را تعریف کنید. این shortcodeها باید در مسیر ``workshop/layouts/shortcodes`` قرار گیرند.

برای مثال، shortcode مربوط به انتخاب بر اساس pathway به شکل زیر پیاده‌سازی شده است:

```
{{ if eq $.Page.Site.Params.pathway_name (.Get 0) }}
{{- .Inner | markdownify }}
{{ end }}
```

Adding admonitions with shortcodes
----------------------------------

از نسخه Educates v2.6.0 به بعد، مجموعه‌ای از admonitionهای سفارشی هنگام استفاده از renderer نوع ``hugo`` پشتیبانی می‌شود. در حال حاضر سه نوع admonition وجود دارد:

- **note** — نمایش به‌صورت جعبه آبی
- **warning** — نمایش به‌صورت جعبه زرد
- **danger** — نمایش به‌صورت جعبه قرمز

این shortcodeها به شکل زیر استفاده می‌شوند:

```
{{< note >}}
A friendly admonition.
{{< /note >}}
{{< warning >}}
Consider this admonition.
{{< /warning >}}
{{< danger >}}
You better consider this admonition!
{{< /danger >}}
```

نسخه رندر شده به شکل زیر خواهد بود:

![Rendered admonitions supported by Educates](admonitions.png)

اطلاعات بیشتر درباره shortcodeها در مستندات Hugo در دسترس است:
https://gohugo.io/content-management/shortcodes/

Embedding custom HTML content
-----------------------------

می‌توان HTML سفارشی را با استفاده از مکانیزم مناسب renderer در محتوای Workshop درج کرد.

در صورت استفاده از renderer نوع ``classic`` و Markdown، می‌توان HTML را مستقیماً بدون نیاز به علامت‌گذاری خاص درج کرد.

```
Lorem ipsum dolor sit amet, consectetur adipiscing elit. Proin justo.
<div>
<table style="width:100%">
  <tr>
    <th>Firstname</th>
    <th>Lastname</th>
    <th>Age</th>
  </tr>
  <tr>
    <td>Jill</td>
    <td>Smith</td>
    <td>50</td>
  </tr>
  <tr>
    <td>Eve</td>
    <td>Jackson</td>
    <td>94</td>
  </tr>
</table>
</div>
Lorem ipsum dolor sit amet, consectetur adipiscing elit. Proin justo.
```

در صورت استفاده از renderer نوع ``classic`` و AsciiDoc، می‌توان HTML را با استفاده از passthrough block درج کرد.

```
Lorem ipsum dolor sit amet, consectetur adipiscing elit. Proin justo.
++++
<div>
<table style="width:100%">
  <tr>
    <th>Firstname</th>
    <th>Lastname</th>
    <th>Age</th>
  </tr>
  <tr>
    <td>Jill</td>
    <td>Smith</td>
    <td>50</td>
  </tr>
  <tr>
    <td>Eve</td>
    <td>Jackson</td>
    <td>94</td>
  </tr>
</table>
</div>
++++
Lorem ipsum dolor sit amet, consectetur adipiscing elit. Proin justo.
```

در صورت استفاده از renderer نوع ``hugo``، shortcodeهای استانداردی برای درج HTML سفارشی مانند ویدئو یا تصویر وجود دارد. در صورت نیاز به قابلیت سفارشی، باید shortcode خود را در مسیر ``workshop/layouts/shortcodes`` ایجاد کرده و در دستورالعمل‌ها از آن استفاده کنید.

در همه موارد توصیه می‌شود HTML فقط شامل یک عنصر سطح بالا باشد. اگر بیش از یک عنصر دارید، همه آن‌ها را داخل یک ``div`` قرار دهید. این موضوع به‌ویژه زمانی مهم است که برخی عناصر HTML به‌صورت hidden علامت‌گذاری شده باشند و بخشی از یک collapsible section باشند؛ در غیر این صورت ممکن است هنگام باز شدن section، عنصر hidden ناخواسته نمایش داده شود.

علاوه بر عناصر بصری HTML، می‌توانید اسکریپت‌ها یا stylesheetها را نیز درج کنید.

اگر از renderer نوع ``classic`` استفاده می‌کنید و لازم است HTML در چند صفحه استفاده شود، آن را در فایل جداگانه قرار داده و از include مربوط به Liquid استفاده کنید. همچنین می‌توانید از مکانیزم partial در Liquid به‌عنوان macro برای گسترش HTML با مقادیر ورودی استفاده کنید.

(embedding-images-and-static-assets)=
Embedding images and static assets
----------------------------------

در صورت استفاده از renderer نوع ``classic``، فایل‌های تصویر می‌توانند در همان محل فایل‌های Markdown یا AsciiDoc قرار گیرند.

در صورت استفاده از renderer نوع ``hugo``:
- اگر از فایل منفرد ``.md`` استفاده شود، تصاویر باید در مسیر ``workshop/static`` قرار گیرند.
- اگر از page bundle استفاده شود، تصاویر می‌توانند در همان دایرکتوری bundle قرار گیرند.

در صورت استفاده از renderer نوع ``classic``، فقط نام فایل تصویر کافی است زیرا تصاویر در همان مسیر نسبی صفحه HTML تولیدشده قرار می‌گیرند.

در صورت استفاده از renderer نوع ``hugo`` و page bundle، فقط نام فایل تصویر کافی است.

اگر از renderer نوع ``hugo`` استفاده می‌کنید و تصاویر در مسیر ``workshop/static`` قرار دارند، باید از shortcode ``baseurl`` برای دسترسی به مسیر URL استفاده کنید. هر فایل (تصویر یا asset دیگر) در ``workshop/static`` نسبت به آن URL قابل دسترسی خواهد بود.

برای مثال، اگر فایل ``educates.png`` در مسیر ``workshop/static/educates.png`` قرار گیرد، باید به شکل زیر در Markdown استفاده شود:

```
![Educates]({{<baseurl>}}/educates.png)
```

اگر فایل در مسیر ``workshop/static/images/educates.png`` باشد:

```
![Educates]({{<baseurl>}}/images/educates.png)
```

نام دایرکتوری ``static`` در URL ظاهر نمی‌شود.

(triggering-actions-from-javascript)=
Triggering actions from Javascript
----------------------------------

Clickable actionها می‌توانند در دستورالعمل‌های Workshop درج شوند تا مراحل دستی کاربر کاهش یابد. در صورت نیاز به اتوماسیون بیشتر، زیرمجموعه‌ای از وظایف داخلی که از طریق clickable action قابل اجرا هستند، می‌توانند از طریق Javascript درج‌شده در صفحه دستورالعمل اجرا شوند. برای مثال می‌توان از این قابلیت استفاده کرد تا هنگام مشاهده یک صفحه، یک dashboard tab به‌صورت خودکار visible شود.

در صورت استفاده از renderer نوع ``classic`` و Markdown، می‌توان Javascript را مستقیماً در سند درج کرد:

```
<script>
window.addEventListener("load", function() {
    educates.expose_dashboard("Editor");
});
</script>
```

در صورت استفاده از renderer نوع ``classic`` و AsciiDoc، می‌توان از passthrough block استفاده کرد:

```
++++
<script>
window.addEventListener("load", function() {
    educates.expose_dashboard("Editor");
});
</script>
++++
```

در صورت استفاده از renderer نوع ``hugo``، باید shortcode سفارشی برای درج Javascript در مسیر ``workshop/layouts/shortcodes`` ایجاد کرده و از آن استفاده کنید.

تمام توابع قابل دسترس در scope شیء `educates` تعریف شده‌اند. API موجود به شکل زیر است:

```
interface API {
    paste_to_terminal(text: string, session: string): void
    paste_to_all_terminals(text: string): void
    execute_in_terminal(command: string, session: string, clear: boolean): void
    execute_in_all_terminals(command: string, clear: boolean): void
    clear_terminal(session: string): void
    clear_all_terminals(): void
    interrupt_terminal(session: string): void
    interrupt_all_terminals(): void
    expose_terminal(session: string): boolean
    expose_dashboard(name: string): boolean
    create_dashboard(name: string, url: string): boolean
    delete_dashboard(name: string): boolean
    reload_dashboard(name: string, url?: string): boolean
}
export educates: API
```

صفحات وب یا وب‌سایت‌های جداگانه که داخل یک dashboard tab نمایش داده می‌شوند، می‌توانند با ارسال پیام Javascript به parent iframe مربوط به dashboard tab به برخی از قابلیت‌های clickable action دسترسی پیدا کنند.

```
<script>
function doit() {
    parent.postMessage({action: "dashboard:open-dashboard", data: { name: "Editor"}}, "*")
}
</script>
<button onclick="doit()">Click me</button>
```

نام actionهایی که می‌توان هدف قرار داد همان نام clickable actionها در دستورالعمل‌های Workshop است. نام action در ویژگی `action` پیام مشخص می‌شود و آرگومان‌ها در ویژگی `data` قرار می‌گیرند.

Actionهای در دسترس از طریق مکانیزم پیام Javascript عبارتند از:

* ``terminal:execute``
* ``terminal:execute-all``
* ``terminal:clear``
* ``terminal:clear-all``
* ``terminal:interrupt``
* ``terminal:interrupt-all``
* ``terminal:input``
* ``dashboard:expose-terminal``
* ``dashboard:open-dashboard``
* ``dashboard:create-dashboard``
* ``dashboard:delete-dashboard``
* ``dashboard:reload-dashboard``

علاوه بر handlerهای متناظر با برخی clickable actionها، handlerهایی برای موارد زیر نیز فراهم شده‌اند:

* ``dashboard:preview-image``
* ``dashboard:finished-workshop``
* ``dashboard:terminate-session``

این‌ها می‌توانند برای نمایش popup پیش‌نمایش تصویر یا پایان دادن به Workshop session استفاده شوند.

تنها پیام‌های Javascriptی پردازش می‌شوند که از صفحاتی ارسال شده باشند که origin آن‌ها مربوط به URL Workshop session باشد، یا وب‌سایت‌های embedشده‌ای که origin آن‌ها با URL اولیه dashboard tab یکسان باشد. اگر کاربر Workshop در context یک dashboard tab به وب‌سایت دیگری منتقل شود و آن سایت پیام Javascript ارسال کند، پیام نادیده گرفته خواهد شد.
