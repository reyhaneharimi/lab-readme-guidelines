.. _infrastructure-providers:

ارائه‌دهندگان زیرساخت
======================

بسته نصب Educates شامل پیکربندی‌های آماده برای چندین ارائه‌دهنده زیرساخت است.
همچنین امکان استفاده از پیکربندی سفارشی برای سایر پلتفرم‌ها نیز وجود دارد.

--------------------------------------------------------------------

.. _install-eks:

نصب روی Amazon EKS
-------------------

نصب روی Amazon Elastic Kubernetes Service (EKS) با تنظیم مقدار ``provider`` برابر ``eks`` پشتیبانی می‌شود.

کامپوننت‌های نصب‌شده:

- Educates training platform
- Contour به عنوان ingress controller
- Kyverno برای اعمال security policy

کامپوننت‌های اضافی:

- external-dns
- cert-manager
- certs (ایجاد ACME wildcard ClusterIssuer)

نیازمندی‌ها:

- ایجاد IAM Role برای external-dns
- ایجاد IAM Role برای cert-manager
- استفاده از IRSA (IAM Role for Service Account)

فرمت ARN:
arn:aws:iam::<ACCOUNT_ID>:role/<ROLE_NAME>


نمونه پیکربندی:

.. raw:: html

   <div dir="ltr">

::

  clusterInfrastructure:
    provider: "eks"
    aws:
      region: "eu-west-1"
      route53:
        hostedZone: "example.com"
      irsaRoles:
        external-dns: "arn:aws:iam::123456789012:role/external-dns"
        cert-manager: "arn:aws:iam::123456789012:role/cert-manager"
  clusterIngress:
    domain: "educates.example.com"

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _install-gke:

نصب روی Google GKE
-------------------

برای Google Kubernetes Engine مقدار ``provider`` را برابر ``gke`` تنظیم کنید.

کامپوننت‌های نصب‌شده:

- Educates
- Contour
- Kyverno
- external-dns
- cert-manager

نیاز به GKE Workload Identity دارید.

فرمت role:
<ROLE_NAME>@<PROJECT_ID>.iam.gserviceaccount.com


نمونه پیکربندی:

.. raw:: html

   <div dir="ltr">

::

  clusterInfrastructure:
    provider: "gke"
    gcp:
      project: "my-project"
      cloudDNS:
        zone: "example.com"
      workloadIdentity:
        external-dns: "external-dns@my-project.iam.gserviceaccount.com"
        cert-manager: "cert-manager@my-project.iam.gserviceaccount.com"
  clusterIngress:
    domain: "educates.example.com"

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _install-kind:

نصب روی Kind
------------

برای نصب روی Kubernetes cluster ایجاد شده با Kind مقدار ``provider`` را برابر ``kind`` قرار دهید.

نیاز است:

- پورت‌های 80 و 443 به host map شوند
- wildcard ingress domain به IP سیستم اشاره کند

اگر از دستور زیر استفاده کنید، cluster به صورت خودکار ایجاد می‌شود:

.. raw:: html

   <div dir="ltr">

::

  educates create-cluster

.. raw:: html

   </div>

--------------------------------------------------------------------

.. _install-minikube:

نصب روی Minikube
----------------

برای Minikube مقدار ``provider`` را برابر ``minikube`` تنظیم کنید.

اگر از driver نوع ``docker`` استفاده می‌کنید:

.. raw:: html

   <div dir="ltr">

::

  minikube tunnel

.. raw:: html

   </div>

اگر از driver با IP مستقل استفاده می‌کنید،
wildcard ingress domain باید به IP cluster اشاره کند.

اگر نمی‌خواهید از Contour استفاده کنید،
می‌توانید مقدار ``provider`` را ``generic`` قرار دهید
یا نصب Contour را غیرفعال کنید.

--------------------------------------------------------------------

.. _install-openshift:

نصب روی OpenShift
------------------

برای OpenShift مقدار ``provider`` را برابر ``openshift`` تنظیم کنید.

کامپوننت‌های نصب‌شده:

- Educates
- Kyverno

در این حالت:

- از OpenShift SCC استفاده می‌شود
- از ingress controller بومی OpenShift استفاده می‌شود

--------------------------------------------------------------------

.. _install-vcluster:

نصب روی vCluster
----------------

برای نصب روی vCluster مقدار ``provider`` را برابر ``vcluster`` قرار دهید.

کامپوننت‌های نصب‌شده:

- Educates
- Kyverno

Ingress باید داخل virtual cluster فعال باشد.

یا باید:

- ingressها از virtual cluster به host sync شوند
- یا ingress controller جداگانه نصب شود

نمونه پیکربندی:

.. raw:: html

   <div dir="ltr">

::

  clusterInfrastructure:
    provider: vcluster

  clusterIngress:
    domain: educates-local-dev.test

  clusterPackages:
    kyverno:
      enabled: false

  clusterSecurity:
    policyEngine: none

  workshopSecurity:
    rulesEngine: none

.. raw:: html

   </div>
