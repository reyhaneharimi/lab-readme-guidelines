راهنمای نصب
============

.. toctree::
   :maxdepth: 1
   :caption: پیش‌نیازهای Cluster

   نیازمندی‌های Cluster <cluster-requirements/index>
   Cluster اختصاصی <cluster-requirements/index#dedicated-kubernetes-cluster>
   اندازه Cluster <cluster-requirements/index#size-of-kubernetes-cluster>
   Ingress controller <cluster-requirements/index#kubernetes-ingress-controller>
   Persistent Volume <cluster-requirements/index#kubernetes-persistent-volumes>
   امنیت Cluster <cluster-requirements/index#cluster-security-enforcement>
   امنیت Workshop <cluster-requirements/index#workshop-security-enforcement>
   نصب packageهای Carvel <cluster-requirements/index#carvel-package-installation>


.. toctree::
   :maxdepth: 2
   :caption: نصب Educates

   دستورالعمل نصب <installation-instructions/index>
   CLI در مقابل kapp-controller <installation-instructions/index#cli-vs-kapp-controller>
   نصب Opinionated <installation-instructions/index#opinionated-cluster-install>
   سرویس‌های نصب‌شده اضافی <installation-instructions/index#additional-installed-services>
   فایل configuration <installation-instructions/index#package-configuration-file>
   اجرای فرآیند نصب <installation-instructions/index#performing-the-installation>


.. toctree::
   :maxdepth: 2

   نصب مبتنی بر CLI <cli-install/index>
   استقرار پلتفرم <cli-install/index#deploying-the-platform>
   Kubeconfig و Context <cli-install/index#kubeconfig-and-context>
   به‌روزرسانی configuration <cli-install/index#updating-configuration>
   حذف نصب <cli-install/index#deleting-the-installation>


.. toctree::
   :maxdepth: 2

   نصب مبتنی بر CLI <cli-install/index>
   نصب مبتنی بر Carvel <carvel-install/index>

   ابزارهای Carvel <carvel-install/index#carvel-command-line-tools>
   نصب kapp-controller <carvel-install/index#installing-kapp-controller>
   Service Account نصب‌کننده <carvel-install/index#installer-service-account>
   اعمال configuration <carvel-install/index#applying-package-values>
   نصب package Educates <carvel-install/index#installing-educates-package>
   به‌روزرسانی configuration <carvel-install/index#updating-package-configuration>
   حذف package نصب‌شده <carvel-install/index#deleting-installed-package>

.. toctree::
   :maxdepth: 1

   ارائه‌دهندگان زیرساخت <infra-providers/index>
   نصب روی Amazon EKS <infra-providers/index#install-eks>
   نصب روی Google GKE <infra-providers/index#install-gke>
   نصب روی Kind <infra-providers/index#install-kind>
   نصب روی Minikube <infra-providers/index#install-minikube>
   نصب روی OpenShift <infra-providers/index#install-openshift>
   نصب روی vCluster <infra-providers/index#install-vcluster>

.. toctree::
   :maxdepth: 2

   تنظیمات پیکربندی <configuration-settings/index>
   تعریف پیکربندی Ingress <configuration-settings/index#defining-configuration-for-ingress>
   تعریف موتور سیاست امنیتی کلاستر <configuration-settings/index#defining-cluster-policy-engine>
   تعریف موتور سیاست امنیتی ورکشاپ <configuration-settings/index#defining-workshop-policy-engine>
   Override کردن container runtime class <configuration-settings/index#overriding-container-runtime-class>
   تعریف image pull secret <configuration-settings/index#defining-image-registry-pull-secrets>
   تعریف storage class <configuration-settings/index#defining-storage-class>
   تعریف storage group <configuration-settings/index#defining-storage-group>
   محدودسازی دسترسی شبکه <configuration-settings/index#restricting-network-access>
   Override کردن MTU شبکه <configuration-settings/index#overriding-network-mtu>
   فعال‌سازی registry pull-through cache <configuration-settings/index#image-registry-pull-through-cache>
   تنظیم credential پیش‌فرض <configuration-settings/index#setting-default-access-credentials>
   ردیابی با webhook <configuration-settings/index#tracking-using-workshop-events>
   ردیابی با Google Analytics <configuration-settings/index#tracking-using-google-analytics>
   ردیابی با Microsoft Clarity <configuration-settings/index#tracking-using-microsoft-clarity>
   ردیابی با Amplitude <configuration-settings/index#tracking-using-amplitude>
   Override کردن استایل ورکشاپ <configuration-settings/index#overriding-workshop-styling>
   اجازه embed کردن ورکشاپ <configuration-settings/index#allowing-sites-to-embed-workshops>
   Override کردن دامنه Cookie <configuration-settings/index#overriding-session-cookie-domain>   
