.. _workshop-content:

محتوای ورکشاپ
==============

.. toctree::
   :maxdepth: 1
   :caption: پیکربندی ورکشاپ

   پیکربندی ورکشاپ <workshop-configuration/index>
   الزامات راه‌اندازی <workshop-configuration/index#workshop-setup-requirements>
   گزینه‌های رندر <workshop-configuration/index#instructions-rendering-options>
   پیکربندی کلاسیک <workshop-configuration/index#classic-renderer-configuration>
   پیکربندی هوگو <workshop-configuration/index#hugo-renderer-configuration>


.. toctree::
   :maxdepth: 1
   :caption: دستورالعمل‌های ورکشاپ

   دستورالعمل‌های ورکشاپ <workshop-instructions/index>
   annotation دستورات executable <workshop-instructions/index#annotation-of-executable-commands>
   annotation متن قابل copy <workshop-instructions/index#annotation-of-text-to-be-copied>
   extensible clickable actions <workshop-instructions/index#extensible-clickable-actions>
   clickable actions برای dashboard <workshop-instructions/index#clickable-actions-for-the-dashboard>
   clickable actions برای editor <workshop-instructions/index#clickable-actions-for-the-editor>
   clickable actions برای file download <workshop-instructions/index#clickable-actions-for-file-download>
   clickable actions برای file upload <workshop-instructions/index#clickable-actions-for-file-upload>
   clickable actions برای examiner <workshop-instructions/index#clickable-actions-for-the-examiner>
   clickable actions برای sections <workshop-instructions/index#clickable-actions-for-sections>
   automatically triggering actions <workshop-instructions/index#automatically-triggering-actions>
   overriding action cooldown period <workshop-instructions/index#overriding-action-cooldown-period>
   hiding clickable actions from view <workshop-instructions/index#hiding-clickable-actions-from-view>
   generating events برای actions <workshop-instructions/index#generating-events-for-actions>
   overriding title و description <workshop-instructions/index#overriding-title-and-description>
   escaping code block content <workshop-instructions/index#escaping-of-code-block-content>
   interpolation of data variables <workshop-instructions/index#interpolation-of-data-variables>
   adding custom data variables <workshop-instructions/index#adding-custom-data-variables>
   passing environment variables <workshop-instructions/index#passing-of-environment-variables>
   handling embedded URL links <workshop-instructions/index#handling-of-embedded-url-links>
   conditional rendering of content <workshop-instructions/index#conditional-rendering-of-content>
   adding admonitions with shortcodes <workshop-instructions/index#adding-admonitions-with-shortcodes>
   embedding custom HTML content <workshop-instructions/index#embedding-custom-html-content>
   embedding images and static assets <workshop-instructions/index#embedding-images-and-static-assets>
   triggering actions from Javascript <workshop-instructions/index#triggering-actions-from-javascript>
   

.. toctree::
   :maxdepth: 1

   Runtime ورکشاپ <runtime/index>
   Pre-defined environment variables از پیش تعریف‌شده <runtime/index#pre-defined-environment-variables>
   اجرای مراحل هنگام شروع container <runtime/index#running-steps-on-container-start>
   اجرای background application‌ها <runtime/index#running-background-applications>
   محیط Terminal user shell <runtime/index#terminal-user-shell-environment>
   Override کردن Terminal shell command <runtime/index#overriding-terminal-shell-command>




.. toctree::
   :maxdepth: 1

   ساخت image <build-image/index>
   ساختار Dockerfile <build-image/index#structure-of-the-dockerfile>
   base image و version tagها <build-image/index#base-images-and-version-tags>
   custom workshop base imageها <build-image/index#custom-workshop-base-images>
   اجرای container با user ID تصادفی <build-image/index#container-run-as-random-user-id>
   نصب system packageهای اضافی <build-image/index#installing-extra-system-packages>
   نصب third party packageها <build-image/index#installing-third-party-packages>   


.. _working-on-content:

کار با workshop content
=======================

.. toctree::
   :maxdepth: 1

   انتشار workshop content <#publishing-of-workshop-content>
   ساخت local workshop image <#local-build-of-workshop-image>
   غیرفعال کردن reserved sessions <#disabling-reserved-sessions>
   دسترسی به workshop error logs <#accessing-workshop-error-logs>
   به‌روزرسانی زنده content <#live-updates-to-the-content>
   تغییرات custom workshop image <#custom-workshop-image-changes>
   overlay برای custom workshop image <#custom-workshop-image-overlay>
   proxy به local workshop content <#proxy-to-local-workshop-content>
   تغییرات در workshop definition <#changes-to-workshop-definition>
