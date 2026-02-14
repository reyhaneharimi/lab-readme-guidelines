
گزارش تحلیل فنی پلتفرم Educates (Enhanced + Use Case)

---

## ۱. ویژگی‌های ظرفیت و مقیاس‌پذیری

این دسته برای کنترل مصرف منابع، جلوگیری از overload و مدیریت رویدادهای بزرگ (Workshop / Bootcamp / Exam) حیاتی است.

> نکته مهم (مستند):
> `spec.portal.sessions.maximum` سقف کل تعداد `WorkshopSession`های همزمان در سطح پورتال است.
> وقتی `portal.sessions.maximum` تنظیم شود، ظرفیت پیش‌فرض هر ورکشاپ نیز برابر همان مقدار در نظر گرفته می‌شود؛ اما برای کنترل اینکه یک ورکشاپ کل ظرفیت را مصرف نکند باید برای همان ورکشاپ `spec.workshops[].capacity` تعیین شود.
> اگر `portal.sessions.maximum` را تنظیم نکنید، باید برای هر ورکشاپ `capacity` را جداگانه تنظیم کنید؛ در غیر این صورت ظرفیت قابل اشتراک بین چند ورکشاپ نخواهد بود.

---

### Portal-Wide Capacity (`spec.portal.sessions.maximum`)

**تعریف در:** `TrainingPortal`

**توضیح فنی:**
سقف تعداد `WorkshopSession`های همزمان در مجموع همه ورکشاپ‌های ثبت‌شده در پورتال.

**رفتار سیستم (Operational):**
وقتی به سقف برسد، `WorkshopSession` جدید دیگر ایجاد/اختصاص داده نمی‌شود. این سقف، «cap مطلق» است و `spec.workshops[].capacity` باید ≤ آن باشد. وقتی یک session به کاربر allocate شد، قابل reassignment به کاربر دیگر نیست.

**Use Case:**
در رویدادی با ۳۰۰ شرکت‌کننده، اما زیرساخت فقط ۱۰۰ سشن همزمان را تحمل می‌کند. در این حالت `maximum` را روی 100 می‌گذارید تا از overload جلوگیری شود. با استفاده از `expires` می‌توان تعداد کل کاربران سرویس‌گرفته در طول زمان را افزایش داد، در حالی که همزمانی همچنان کنترل می‌شود.

**نمونه YAML:**

```yaml
apiVersion: training.educates.dev/v1beta1
kind: TrainingPortal
metadata:
  name: edu-portal
spec:
  portal:
    sessions:
      maximum: 100
```

**نکات طراحی (پیشنهادی):**
اگر چند ورکشاپ دارید، برای هر کدام `capacity` جدا تعیین کنید تا یک ورکشاپ کل ظرفیت را مصرف نکند. اگر پورتال را طولانی‌مدت باز نگه می‌دارید، در کنار ظرفیت از `expires` استفاده کنید تا ظرفیت در طول زمان آزاد شود.

---

### Workshop-Specific Capacity (`spec.workshops[].capacity`)

**تعریف در:** `TrainingPortal`

**توضیح فنی:**
سقف تعداد `WorkshopSession`های همزمان برای یک ورکشاپ مشخص.

**رفتار سیستم:**
اگر ورکشاپ به `capacity` خودش برسد، همان ورکشاپ session جدید نمی‌گیرد. `capacity` باید همیشه ≤ `portal.sessions.maximum` باشد (وقتی maximum وجود دارد).

**Use Case:**
وقتی چند ورکشاپ در یک پورتال دارید و نمی‌خواهید یک ورکشاپ محبوب کل ظرفیت را مصرف کند، برای هر ورکشاپ سقف جدا تعیین می‌کنید تا تعادل مصرف منابع حفظ شود.

**نمونه YAML:**

```yaml
apiVersion: training.educates.dev/v1beta1
kind: TrainingPortal
metadata:
  name: edu-portal
spec:
  portal:
    sessions:
      maximum: 100
  workshops:
  - name: k8s-basics
    capacity: 30
  - name: gitops-intro
    capacity: 40
```

---

### Reserved / Initial Pool (`spec.workshops[].reserved` + `spec.workshops[].initial`)

**تعریف در:** `TrainingPortal`

**توضیح فنی:**
به‌صورت پیش‌فرض، ۱ instance رزرو از هر ورکشاپ upfront ساخته می‌شود تا اولین کاربر بدون انتظار وارد شود. `reserved` تعداد instanceهای standby است که باید آماده بمانند. `initial` تعداد instanceهایی است که در شروع ساخته می‌شوند.

**رفتار سیستم (مستند):**
وقتی یک reserved instance به کاربر allocate می‌شود، اگر ظرفیت پر نشده باشد یک instance جدید برای reserve ساخته می‌شود. مجموع `(allocated + reserved)` برای آن ورکشاپ هرگز از `capacity` بالاتر نمی‌رود. اگر `initial` غیر صفر ولی کمتر از `reserved` باشد، مقدار `initial` override می‌شود و به اندازه `reserved` ساخته می‌شود. اگر `initial: 0` باشد، هیچ reserved session ابتدایی ساخته نمی‌شود؛ اما با اولین درخواست، session on-demand ساخته می‌شود و سپس برای رسیدن به `reserved` تعداد لازم ایجاد می‌گردد.

**Use Case:**
برای کاهش latency ورود (بدون طراحی صف بیرونی) از `reserved/initial` استفاده می‌کنید و هزینه بالاتر منابع را به‌عنوان trade-off می‌پذیرید.

**نمونه YAML:**

```yaml
apiVersion: training.educates.dev/v1beta1
kind: TrainingPortal
metadata:
  name: edu-portal
spec:
  portal:
    sessions:
      maximum: 100
  workshops:
  - name: lab-kubernetes-fundamentals
    capacity: 100
    initial: 75
    reserved: 5
```

---

### Cap روی تعداد session همزمان هر کاربر (`portal.sessions.registered` / `portal.sessions.anonymous`)

**تعریف در:** `TrainingPortal`

**توضیح فنی:**
`portal.sessions.registered` سقف تعداد session همزمانی است که یک کاربر ثبت‌نام‌شده می‌تواند شروع کند. این limit به‌صورت پیش‌فرض روی anonymous هم اثر می‌گذارد؛ برای limit متفاوت روی anonymous از `portal.sessions.anonymous` استفاده می‌شود.

**Use Case:**
برای جلوگیری از اینکه یک کاربر چند session را همزمان استارت کند و منابع را هدر بدهد، سقف همزمانی per-user را محدود می‌کنید.

**نمونه YAML:**

```yaml
apiVersion: training.educates.dev/v1beta1
kind: TrainingPortal
metadata:
  name: edu-portal
spec:
  portal:
    sessions:
      maximum: 50
      registered: 1
      anonymous: 1
  workshops:
  - name: lab-asciidoc-sample
    capacity: 25
    reserved: 2
```

---

## ۲. ویژگی‌های چرخه حیات و مدیریت WorkshopEnvironment / Session

این بخش ستون فقرات کنترل هزینه، پاک‌سازی خودکار و جلوگیری از اشغال دائمی ظرفیت است.

---

### Session Expiry (`spec.workshops[].expires`)

**تعریف در:** `TrainingPortal` (سطح هر ورکشاپ)

**توضیح فنی (مستند):**
فرمت `expires` به‌شکل `integer + suffix` با `s|m|h` است. زمان از لحظه allocation به کاربر محاسبه می‌شود. پس از پایان زمان، session terminate و delete می‌شود و ظرفیت آزاد می‌گردد. وقتی `expires` تنظیم شده باشد، با Finish یا Restart کاربر نیز session delete می‌شود.

**Use Case:**
برای self-paced که باید ظرفیت در طول زمان آزاد شود، مدت session را محدود می‌کنید تا منابع قفل نشود.

**نمونه YAML:**

```yaml
apiVersion: training.educates.dev/v1beta1
kind: TrainingPortal
metadata:
  name: edu-portal
spec:
  workshops:
  - name: terraform-intro
    capacity: 40
    reserved: 2
    expires: 45m
```

---

### Orphaned Cleanup (`spec.workshops[].orphaned`)

**تعریف در:** `TrainingPortal`

**توضیح فنی (مستند):**
اگر کاربر صفحه را ببندد، بعد از مدت `orphaned` session terminate می‌شود. اگر صفحه hidden/minimized باشد، termination بعد از ۳× مقدار orphaned رخ می‌دهد. برای supervised workshops توصیه شده از این گزینه اجتناب شود تا session کاربران در break یا sleep لپ‌تاپ حذف نشود.

**Use Case:**
در رویدادهای عمومی که کاربرها session می‌گیرند و صفحه را می‌بندند، با orphaned ظرفیت سریع‌تر آزاد می‌شود؛ اما باید ریسک حذف ناخواسته را مدیریت کنید.

**نمونه YAML:**

```yaml
apiVersion: training.educates.dev/v1beta1
kind: TrainingPortal
metadata:
  name: edu-portal
spec:
  workshops:
  - name: public-workshop
    capacity: 80
    expires: 60m
    orphaned: 5m
```

---

### تمدید کنترل‌شده زمان (`deadline` + `overtime`)

**تعریف در:** `TrainingPortal`

**توضیح فنی (مستند):**
`deadline` سقف نهایی زمانی است که session می‌تواند تا آن extend شود. به‌صورت پیش‌فرض هر بار extend، ۲۵٪ مدت اولیه به expiration اضافه می‌کند. با `overtime` می‌توانید مقدار افزایشی هر تمدید را override کنید. وقتی به `deadline` برسد دیگر قابل extend نیست و timer در بازه نهایی overtime به رنگ red نمایش داده می‌شود.

**Use Case:**
برای Exam عملی که نیاز به تمدید کنترل‌شده دارد، زمان پایه را با `expires` می‌گذارید، افزایش هر تمدید را با `overtime` کنترل می‌کنید و سقف نهایی را با `deadline` محدود می‌سازید.

**نمونه YAML:**

```yaml
apiVersion: training.educates.dev/v1beta1
kind: TrainingPortal
metadata:
  name: edu-portal
spec:
  workshops:
  - name: practical-exam
    capacity: 60
    reserved: 1
    expires: 60m
    overtime: 30m
    deadline: 120m
```

---

### Timeout برای دسترسی به Dashboard (`spec.workshops[].overdue`)

**تعریف در:** `TrainingPortal`

**توضیح فنی (مستند):**
اگر session allocate شود اما کاربر تا زمان `overdue` نتواند به dashboard برسد، سیستم session را delete می‌کند. سپس کاربر به URL حذف session redirect می‌شود و بعد به لیست ورکشاپ‌های پورتال (یا front-end سفارشی) برمی‌گردد.

**Use Case:**
برای ورکشاپ‌هایی که startup ممکن است گیر کند، overdue باعث می‌شود sessionهای stuck ظرفیت را قفل نکنند.

**نمونه YAML:**

```yaml
apiVersion: training.educates.dev/v1beta1
kind: TrainingPortal
metadata:
  name: edu-portal
spec:
  workshops:
  - name: lab-markdown-sample
    expires: 60m
    orphaned: 5m
    overdue: 2m
```

---

### Update / Replace محیط ورکشاپ‌ها (`portal.updates.workshop`)

**تعریف در:** `TrainingPortal`

**توضیح فنی (مستند):**
تغییر لیست ورکشاپ‌ها با update کردن `TrainingPortal` اعمال می‌شود. حذف یک ورکشاپ از لیست باعث می‌شود `WorkshopEnvironment` به حالت stopping برود و پس از اتمام sessionهای فعال حذف شود. اضافه کردن ورکشاپ باعث ایجاد `WorkshopEnvironment` جدید می‌شود. به‌صورت پیش‌فرض با تغییر `Workshop` definition محیط‌ها جایگزین نمی‌شوند، اما با `portal.updates.workshop: true` محیط‌ها replace می‌شوند و old/new می‌توانند همزمان وجود داشته باشند؛ بنابراین تنظیم `portal.sessions.maximum` برای کنترل همزمانی توصیه شده است.

**Use Case:**
در محیط توسعه محتوا، با فعال کردن updates می‌توانید تغییرات workshop definition را سریع‌تر به کاربران جدید اعمال کنید بدون اینکه sessionهای فعال قطع شوند.

**نمونه YAML:**

```yaml
apiVersion: training.educates.dev/v1beta1
kind: TrainingPortal
metadata:
  name: edu-portal
spec:
  portal:
    sessions:
      maximum: 20
    updates:
      workshop: true
  workshops:
  - name: lab-dev-workshop
    capacity: 20
```

---

### Refresh دوره‌ای WorkshopEnvironment (`refresh`)

**تعریف در:** `TrainingPortal` (در `portal.workshop.defaults.refresh` یا `workshops[].refresh`)

**توضیح فنی (مستند):**
برای پاک‌سازی سرویس‌های shared که در `WorkshopEnvironment` انباشته می‌شوند، می‌توان refresh دوره‌ای انجام داد. در refresh، محیط قدیمی stopping می‌شود، محیط جدید ساخته می‌شود و درخواست‌های جدید به محیط جدید می‌روند. محیط قدیمی بعد از اتمام sessionهای فعال حذف می‌شود.

**Use Case:**
برای ورکشاپ‌هایی که shared service دارند و drift یا انباشت داده ایجاد می‌شود، refresh یک مسیر کنترل‌شده برای cleanup است.

**نمونه YAML:**

```yaml
apiVersion: training.educates.dev/v1beta1
kind: TrainingPortal
metadata:
  name: edu-portal
spec:
  portal:
    sessions:
      maximum: 50
    workshop:
      defaults:
        refresh: 168h
  workshops:
  - name: lab-asciidoc-sample
    capacity: 25
    reserved: 2
    refresh: 24h
```

---

## ۳. امنیت، RBAC و ایزولاسیون

---

### کنترل دسترسی Kubernetes Token (`session.namespaces.security.token.enabled`)

**تعریف در:** `Workshop` (`Workshop Definition`)

**توضیح فنی (مستند):**
اگر ورکشاپ اصلاً نیاز به Kubernetes ندارد، می‌توان دسترسی به API کلاستر را با `session.namespaces.security.token.enabled: false` قطع کرد. روش قدیمی `automountServiceAccountToken=false` دیگر کار نمی‌کند.

**Use Case:**
در ورکشاپ‌های غیر Kubernetes، این تنظیم جلوی سوءاستفاده از دسترسی کلاستر را می‌گیرد.

**نمونه YAML:**

```yaml
apiVersion: training.educates.dev/v1beta1
kind: Workshop
metadata:
  name: non-k8s-workshop
spec:
  session:
    namespaces:
      security:
        token:
          enabled: false
```

---

### Pod Security Policy سطح namespace (`session.namespaces.security.policy`)

**تعریف در:** `Workshop`

**توضیح فنی (مستند):**
پیش‌فرض policy سطح session namespace برابر `restricted` است. اگر نیاز به اجرای container به‌صورت root باشد باید policy را به `baseline` تغییر داد.

**Use Case:**
برای ورکشاپ‌هایی که با imageهای نیازمند root یا ابزارهای سطح پایین سروکار دارند، baseline لازم می‌شود.

**نمونه YAML:**

```yaml
apiVersion: training.educates.dev/v1beta1
kind: Workshop
metadata:
  name: root-required-workshop
spec:
  session:
    namespaces:
      security:
        policy: baseline
```

---

### Resource Budget و Quota / LimitRange (`session.namespaces.budget`)

**تعریف در:** `Workshop`

**توضیح فنی (مستند):**
با `session.namespaces.budget` presetهای بودجه منابع فعال می‌شود و برای namespace session، `ResourceQuota` و `LimitRange` ایجاد می‌گردد. علاوه بر CPU/Memory محدودیت‌هایی برای برخی resource countها هم اعمال می‌شود.

**Use Case:**
در دوره‌های پیشرفته که کاربران workload اجرا می‌کنند، budget مانع از تاثیرگذاری اشتباهات یک کاربر بر کل کلاستر می‌شود.

**نمونه YAML:**

```yaml
apiVersion: training.educates.dev/v1beta1
kind: Workshop
metadata:
  name: k8s-budgeted-workshop
spec:
  session:
    namespaces:
      budget: medium
```

**نمونه YAML (override روی LimitRange defaults با `session.namespaces.limits`):**

```yaml
apiVersion: training.educates.dev/v1beta1
kind: Workshop
metadata:
  name: k8s-budgeted-workshop
spec:
  session:
    namespaces:
      budget: medium
      limits:
        min:
          cpu: 50m
          memory: 32Mi
        max:
          cpu: "1"
          memory: 1Gi
        defaultRequest:
          cpu: 50m
          memory: 128Mi
        default:
          cpu: 500m
          memory: 1Gi
```

---

### Object Injection (`session.objects` vs `request.objects`)

**تعریف در:** `Workshop`

**توضیح فنی (مستند):**
`session.objects` منابع را هنگام provision اولیه session ایجاد می‌کند و ممکن است قبل از allocation باشد. اگر deployment به secretی وابسته باشد که فقط موقع allocation تولید می‌شود، ممکن است `CreateContainerConfigError` رخ دهد. `request.objects` منابع را فقط زمان allocation ایجاد می‌کند و این مشکل را کاهش می‌دهد.

**Use Case:**
برای ورکشاپ‌هایی که secretها یا پارامترها فقط هنگام allocation قابل تولید هستند، `request.objects` پایدارتر است.

**نمونه YAML:**

```yaml
apiVersion: training.educates.dev/v1beta1
kind: Workshop
metadata:
  name: rbac-injection-workshop
spec:
  session:
    objects:
    - apiVersion: rbac.authorization.k8s.io/v1
      kind: ClusterRoleBinding
      metadata:
        name: $(session_name)-cluster-admin
      roleRef:
        apiGroup: rbac.authorization.k8s.io
        kind: ClusterRole
        name: cluster-admin
      subjects:
      - kind: ServiceAccount
        namespace: $(workshop_namespace)
        name: $(service_account)
```

---

## ۴. Secret Injection & Propagation

(SecretCopier / SecretExporter / SecretImporter / SecretInjector)

---

### ۴.۱ `SecretCopier` (cluster-scoped)

**کارکرد:**
کپی یک secret از یک namespace مشخص به namespaceهای دیگر بر اساس rule/selector.

**توضیح فنی (مستند):**
Secret هرگز به همان namespace مبدأ کپی نمی‌شود. اگر `targetNamespaces.nameSelector` تعریف نشود، رفتار پیش‌فرض معادل exclude کردن `kube-*` است. مقصد می‌تواند با nameSelector/labelSelector/uidSelector/ownerSelector انتخاب شود. می‌توان `targetSecret.name` برای تغییر نام مقصد و `targetSecret.labels` برای افزودن label تعیین کرد و labelهای مبدأ کپی نمی‌شوند. در صورت استفاده از `copyAuthorization.sharedSecret`، namespace مقصد باید SecretImporter هم‌نام داشته باشد و sharedSecret یکسان باشد.

**Use Case:**
کپی کنترل‌شده registry secret به namespaceهای session یا namespaceهای یک تیم.

**نمونه YAML:**

```yaml
apiVersion: secrets.educates.dev/v1beta1
kind: SecretCopier
metadata:
  name: registry-credentials
spec:
  rules:
  - sourceSecret:
      name: registry-credentials
      namespace: registry
    targetNamespaces:
      nameSelector:
        matchNames:
        - developer-*
        - !kube-*
    copyAuthorization:
      sharedSecret: my-shared-secret
```

---

### ۴.۲ `SecretExporter` + `SecretImporter`

**کارکرد:**
Export و import کنترل‌شده secret بین namespaceها با handshake و sharedSecret.

**توضیح فنی (مستند):**
نام `SecretExporter` باید هم‌نام secret مبدأ باشد. secret به مقصد کپی نمی‌شود مگر در مقصد `SecretImporter` هم‌نام secret مقصد وجود داشته باشد. برای authorize کردن، sharedSecret باید در هر دو یکسان باشد. می‌توان با `targetSecret.name` نام مقصد را تغییر داد و در این حالت نام SecretImporter هم باید مطابق باشد. مالکیت secret مقصد به SecretImporter داده می‌شود و با حذف importer، secret مقصد حذف می‌گردد.

**Use Case:**
برای اینکه session namespaceها فقط با داشتن sharedSecret بتوانند secretهای حساس (registry/git) را دریافت کنند.

**نمونه YAML (Exporter):**

```yaml
apiVersion: secrets.educates.dev/v1beta1
kind: SecretExporter
metadata:
  name: registry-credentials
  namespace: registry
spec:
  rules:
  - targetNamespaces:
      nameSelector:
        matchNames:
        - workshop-*
    copyAuthorization:
      sharedSecret: my-shared-secret
    targetSecret:
      name: registry-credentials
      labels:
        registry-pull-secret: ""
```

**نمونه YAML (Importer):**

```yaml
apiVersion: secrets.educates.dev/v1beta1
kind: SecretImporter
metadata:
  name: registry-credentials
  namespace: workshop-user-namespace
spec:
  sourceNamespaces:
    nameSelector:
      matchNames:
      - registry
  copyAuthorization:
    sharedSecret: my-shared-secret
```

---

### ۴.۳ `SecretInjector` (cluster-scoped)

**کارکرد:**
Inject کردن secretها به service accountها (برای image pull).

**توضیح فنی (مستند):**
SecretInjector نوع secret را validate نمی‌کند و هر secret match‌شده inject می‌شود. انتخاب source secret با nameSelector/labelSelector انجام می‌شود. انتخاب service account و namespace نیز با selectorها کنترل می‌شود و اگر nameSelector برای namespace تعریف نشود، رفتار پیش‌فرض exclude کردن `kube-*` است.

**Use Case:**
تزریق خودکار registry pull secret به `default` service account برای namespaceهای session.

**نمونه YAML:**

```yaml
apiVersion: secrets.educates.dev/v1beta1
kind: SecretInjector
metadata:
  name: registry-credentials
spec:
  rules:
  - sourceSecrets:
      nameSelector:
        matchNames:
        - registry-credentials
    serviceAccounts:
      nameSelector:
        matchNames:
        - default
    targetNamespaces:
      nameSelector:
        matchNames:
        - workshop-*
        - !kube-*
```

---

## ۵. محدودیت‌های فنی و چالش‌های عملیاتی (مستند)

---

### ۱) Nginx Ingress و WebSocket (Terminal/Live UI)

**محدودیت/ریسک مستند:**
در برخی سناریوها reload/reconfigure شدن Nginx می‌تواند باعث اختلال در WebSocket و sessionهای تعاملی شود.

**Use Case اثر:**
در رویدادهای بزرگ که ingress دائماً update می‌شود، کاربران ممکن است قطع اتصال مکرر تجربه کنند.

---

### ۲) Safari و HTTP/2

**محدودیت/ریسک مستند:**
Safari در برخی سناریوها با HTTP/2 مشکل دارد و ممکن است قطع ارتباط رخ دهد.

**Use Case اثر:**
اگر جمعیت قابل‌توجهی کاربر Safari دارید، این مورد باید در طراحی ingress لحاظ شود.

---

### ۳) “سیستم صف / Waiting room”

**وضعیت مستندات:**
در مستندات رسمی مکانیزم صف داخلی به‌عنوان قابلیت مشخص معرفی نشده است. کنترل رفتار در پر شدن ظرفیت عمدتاً با capها و pre-provision انجام می‌شود.

---

## ۶. کنترل دسترسی و تجربه کاربری پورتال (Registration / Catalog / Password / External Index)

---

### Registration Type (`portal.registration.type` + `portal.registration.enabled`)

**رفتار مستند:**
حالت پیش‌فرض نمایش صفحه ثبت‌نام است. اگر بخواهید فقط admin وارد شود و sessionها با REST API و یک front-end خارجی مدیریت شوند، `registration.enabled: false` استفاده می‌شود. برای anonymous access، `registration.type: anonymous` باعث می‌شود با ورود به صفحه اصلی یک حساب خودکار ساخته و login انجام شود.

**نمونه YAML (غیرفعال‌سازی ثبت‌نام):**

```yaml
apiVersion: training.educates.dev/v1beta1
kind: TrainingPortal
metadata:
  name: edu-portal
spec:
  portal:
    registration:
      type: one-step
      enabled: false
```

**نمونه YAML (anonymous):**

```yaml
apiVersion: training.educates.dev/v1beta1
kind: TrainingPortal
metadata:
  name: edu-portal
spec:
  portal:
    registration:
      type: anonymous
```

---

### Event Access Code (`portal.password`)

**رفتار مستند:**
برای جلوگیری از دسترسی هر کسی که URL را دارد، می‌توان یک access code مشترک تعریف کرد تا قبل از redirect به catalog/login از کاربر دریافت شود.

**نمونه YAML:**

```yaml
apiVersion: training.educates.dev/v1beta1
kind: TrainingPortal
metadata:
  name: edu-portal
spec:
  portal:
    password: workshops-2026-02-10
```

---

### Catalog Visibility (`portal.catalog.visibility`)

**رفتار مستند:**
پیش‌فرض `private` است و catalog بعد از login دیده می‌شود. با `public`، catalog قبل از login نمایش داده می‌شود و REST API لیست ورکشاپ‌ها هم بدون auth قابل دسترس می‌شود.

**نمونه YAML:**

```yaml
apiVersion: training.educates.dev/v1beta1
kind: TrainingPortal
metadata:
  name: edu-portal
spec:
  portal:
    catalog:
      visibility: public
```

---

### External Workshop List (`portal.index`)

**رفتار مستند:**
وقتی registration خاموش است و sessionها از طریق REST API توسط یک وب‌سایت خارجی مدیریت می‌شوند، می‌توان URL لیست ورکشاپ خارجی را با `portal.index` مشخص کرد.

**نمونه YAML:**

```yaml
apiVersion: training.educates.dev/v1beta1
kind: TrainingPortal
metadata:
  name: edu-portal
spec:
  portal:
    index: https://www.example.com/
    registration:
      type: one-step
      enabled: false
```

---

## ۷. Ingress / Hostname / Iframe / Cookies (مستند)

---

### Override hostname و TLS (`portal.ingress.hostname` + `tlsCertificateRef`)

**رفتار مستند:**
`hostname` می‌تواند short یا FQDN باشد. در حالت FQDN باید common parent domain با cluster ingress domain وجود داشته باشد تا محدودیت‌های cookie cross-site مشکل ایجاد نکند. در صورت نیاز می‌توان `tlsCertificateRef` تعریف کرد.

**نمونه YAML:**

```yaml
apiVersion: training.educates.dev/v1beta1
kind: TrainingPortal
metadata:
  name: edu-portal
spec:
  portal:
    ingress:
      hostname: labs.educates.dev
      tlsCertificateRef:
        name: labs-educates-dev-tls
        namespace: default
```

---

### اجازه iframe و cookie domain (`portal.theme.frame.ancestors` / `portal.cookies.domain`)

**رفتار مستند:**
برای iframe شدن portal/session باید `portal.theme.frame.ancestors` تنظیم شود. در برخی شرایط برای سازگاری مرورگرها ممکن است `portal.cookies.domain` لازم باشد و همه سایت‌ها باید parent domain مشترک داشته باشند. سایت embed کننده باید HTTPS باشد.

**نمونه YAML:**

```yaml
apiVersion: training.educates.dev/v1beta1
kind: TrainingPortal
metadata:
  name: edu-portal
spec:
  portal:
    theme:
      frame:
        ancestors:
        - https://www.example.com
    cookies:
      domain: example.com
```

---

## ۸. ضمیمه: الگوهای Use Case محور (با YAML)

### الگوی A: رویداد بزرگ با ورود موجی

```yaml
apiVersion: training.educates.dev/v1beta1
kind: TrainingPortal
metadata:
  name: big-event
spec:
  portal:
    sessions:
      maximum: 100
  workshops:
  - name: lab-kubernetes-fundamentals
    capacity: 100
    initial: 75
    reserved: 5
    expires: 90m
```

### الگوی B: پورتال دائمی (Self-paced)

```yaml
apiVersion: training.educates.dev/v1beta1
kind: TrainingPortal
metadata:
  name: self-paced
spec:
  portal:
    sessions:
      maximum: 40
      registered: 1
  workshops:
  - name: terraform-intro
    capacity: 20
    reserved: 1
    expires: 60m
```

### الگوی C: Exam عملی با timeout و تمدید

```yaml
apiVersion: training.educates.dev/v1beta1
kind: TrainingPortal
metadata:
  name: exam-portal
spec:
  portal:
    sessions:
      maximum: 60
  workshops:
  - name: practical-exam
    capacity: 60
    reserved: 1
    expires: 60m
    overtime: 30m
    deadline: 120m
    overdue: 3m
```

---

