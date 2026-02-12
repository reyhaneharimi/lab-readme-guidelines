.. _workshop-instructions:

دستورالعمل‌های ورکشاپ
======================

فایل‌های module که دستورالعمل‌های Workshop را تشکیل می‌دهند می‌توانند هنگام استفاده از ``classic`` renderer از قالب‌های Markdown یا AsciiDoc استفاده کنند.

پسوند فایل‌ها باید ``.md`` یا ``.adoc`` باشد.

در صورت استفاده از ``hugo`` renderer فقط Markdown پشتیبانی می‌شود.

در حالت ``hugo``:

- هر صفحه می‌تواند یک فایل ``.md`` باشد
- یا یک page bundle شامل پوشه‌ای با فایل ``index.md``

اگر از تصویر استفاده شود:

- در ``classic`` تصاویر کنار فایل Markdown/AsciiDoc قرار می‌گیرند
- در ``hugo`` اگر فایل تکی باشد تصاویر در ``workshop/static`` قرار می‌گیرند
- در page bundle تصاویر می‌توانند داخل همان پوشه باشند

--------------------------------------------------------------------

.. _annotation-executable-commands:

نشانه‌گذاری دستورات قابل اجرا
------------------------------

در کنار Markdown یا AsciiDoc می‌توان code blockها را به‌صورت clickable تعریف کرد.

Markdown:

.. raw:: html

   <div dir="ltr">

::

  ```execute
  echo "Execute command."
