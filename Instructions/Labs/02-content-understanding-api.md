---
lab:
  title: تطوير تطبيق عميل "فهم المحتوى"
  description: استخدم واجهة REST API الخاصة بـ Azure AI Content Understanding لتطوير تطبيق عميل لمحلل.
---

# تطوير تطبيق عميل "فهم المحتوى"

في هذا التمرين، يمكنك استخدام Azure AI Content Understanding لإنشاء محلل يستخرج المعلومات من بطاقات العمل. ستقوم بعد ذلك بتطوير تطبيق عميل يستخدم المحلل لاستخراج تفاصيل جهة الاتصال من بطاقات العمل الممسوحة ضوئيًا.

سيستغرق هذا التدريب حوالي **30** دقيقة.

## إنشاء مركز ومشروع Azure AI Foundry

تتطلب ميزات Azure AI Foundry التي سنستخدمها في هذا التمرين مشروعًا يعتمد على مورد *مركز* Azure AI Foundry.

1. في متصفح الويب، افتح [مدخل Azure AI Foundry](https://ai.azure.com) على `https://ai.azure.com` وسجّل الدخول باستخدام بيانات اعتماد Azure الخاصة بك. أغلق أي تلميحات أو أجزاء التشغيل السريع يتم فتحه عندما تقوم بتسجيل الدخول لأول مرة، وإذا لزم الأمر،استخدم شعار **Azure AI Foundry** في أعلى اليسار للانتقال إلى الصفحة الرئيسية، والتي تبدو مشابهة للصورة التالية (أغلق جزء **المساعدة** إذا كان مفتوحًا):

    ![لقطة شاشة لمدخل Azure AI Foundry.](./media/ai-foundry-home.png)

1. في المتصفح، انتقل إلى `https://ai.azure.com/managementCenter/allResources` وحدد **إنشاء جديد**. ثم حدد خيار إنشاء **مورد مركز ذكاء اصطناعي** جديد.
1. في معالج **إنشاء مشروع**، أدخل اسمًا صالحًا لمشروعك، ثم حدد الخيار لإنشاء مركز جديد. بعد ذلك، استخدم رابط **إعادة تسمية المركز** لتحديد اسم صالح للمركز الجديد، ثم وسّع **الخيارات المتقدمة**، وحدد الإعدادات التالية لمشروعك:
    - **Subscription**: *اشتراكك في Azure*
    - **Resource group**: *إنشاء مجموعة موارد أو تحديدها*
    - **اسم المركز**: اسم صالح لمركزك.
    - **الموقع**: اختر أحد المواقع التالية:\*
        - شرق أستراليا
        - منطقة السويد الوسطى
        - غرب الولايات المتحدة

    > \*في وقت كتابة هذا النص، تتوفر خدمة Azure AI Content Understanding في هذه المناطق فقط.

    > **تلميح**: إذا كان زر **إنشاء** لا يزال معطلا، فتأكد من إعادة تسمية مركزك بقيمة أبجدية رقمية فريدة.

1. انتظر حتى يتم إنشاء مشروعك، ثم انتقل إلى صفحة نظرة عامة على المشروع.

## استخدام واجهة REST API لإنشاء محلل Content Understanding

ستستخدم واجهة REST API لإنشاء محلل قادر على استخراج البيانات من صور بطاقات العمل.

1. افتح علامة تبويب جديدة للمتصفح (مع إبقاء مدخل مصنع الذكاء الاصطناعي في Azure مفتوحًا في علامة التبويب الموجودة). بعد ذلك في علامة التبويب الجديدة، انتقل إلى [بوابة Azure](https://portal.azure.com) على `https://portal.azure.com`؛ وسجّل الدخول باستخدام بيانات اعتماد Azure الخاصة بك إذا طُلب منك ذلك.

    أغلق أي إشعارات ترحيبية لرؤية الصفحة الرئيسية لبوابة Azure.

1. استخدم الزر **[\>_]** الموجود على يمين شريط البحث أعلى الصفحة لإنشاء Cloud Shell جديد في بوابة Azure، وتحديد بيئة ***PowerShell*** بدون مساحة تخزين في اشتراكك.

    يوفّر Cloud Shell واجهة سطر الأوامر في الجزء الموجود في أسفل بوابة Azure. يمكنك تغيير حجم هذا الجزء أو تكبيره لتسهيل العمل فيه.

    > **تلميح**: قم بتغيير حجم الجزء بحيث تتمكن من العمل بشكل أساسي في Cloud Shell، مع الاستمرار في عرض المفاتيح ونقطة النهاية في صفحة مدخل Microsoft Azure — إذ ستحتاج إلى نسخها إلى التعليمات البرمجية الخاصة بك.

1. في شريط أدوات Cloud Shell، في قائمة **الإعدادات**، حدد **الانتقال إلى الإصدار الكلاسيكي** (هذا مطلوب لاستخدام محرر التعليمات البرمجية).

    **<font color="red">تأكد من التبديل إلى الإصدار الكلاسيكي من cloud shell قبل المتابعة.</font>**

1. في جزء Cloud Shell، أدخل الأوامر التالية لاستنساخ مستودع GitHub الذي يحتوي على ملفات التعليمات البرمجية لهذا التمرين (اكتب الأمر أو انسخه إلى الحافظة ثم انقر بزر الماوس الأيمن في سطر الأوامر وقم بلصقه كنص عادي):

    ```
   rm -r mslearn-ai-info -f
   git clone https://github.com/microsoftlearning/mslearn-ai-information-extraction mslearn-ai-info
    ```

    > **تلميح**: أثناء إدخال الأوامر في CloudShell، قد يشغل الإخراج كمية كبيرة من مساحة المخزن المؤقت للشاشة. يمكنك مسح الشاشة عن طريق إدخال الأمر `cls` لتسهيل التركيز على كل مهمة.

1. بعد استنساخ مخزن البيانات الخاصة، انتقل إلى المجلد الذي يحتوي على ملفات التعليمات البرمجية لتطبيقك:

    ```
   cd mslearn-ai-info/Labfiles/content-app
   ls -a -l
    ```

    يحتوي المجلد على صورتين ممسوحتين ضوئيًا لبطاقتَي عمل، بالإضافة إلى ملفات التعليمات البرمجية بلغة Python التي تحتاج إليها لإنشاء تطبيقك.

1. في جزء سطر الأوامر Cloud Shell، أدخل الأمر التالي لتثبيت المكتبات التي ستستخدمها:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt
    ```

1. أدخل الأمر التالي لتحرير ملف التكوين الذي تم توفيره:

    ```
   code .env
    ```

    يتم فتح الملف في محرر التعليمات البرمجية.

1. في ملف التعليمات البرمجية، استبدل العنصرين النائبين **YOUR_ENDPOINT** و**YOUR_KEY** بنقطة نهاية خدمات Azure AI وأي من مفاتيحها (تم نسخها من مدخل Microsoft Azure)، وتأكد من تعيين **ANALYZER_NAME** على `business-card-analyzer`.
1. بعد استبدال العناصر النائبة في محرر التعليمات البرمجية، استخدم الأمر **CTRL+S** لحفظ التغييرات ثم استخدم الأمر **CTRL+Q** لإغلاق محرر التعليمات البرمجية مع إبقاء سطر أوامر Cloud Shell مفتوحًا.

    > **تلميح**: يمكنك تكبير جزء Cloud Shell الآن.

1. في سطر أوامر Cloud Shell، أدخِل الأمر التالي لعرض ملف JSON المسمى **biz-card.json**المُرفق:

    ```
   cat biz-card.json
    ```

    قم بالتمرير داخل جزء Cloud Shell لعرض محتوى ملف JSON، الذي يحدد مخطط المحلل لبطاقة عمل.

1. عند عرض ملف JSON للمحلل، أدخِل الأمر التالي لتحرير ملف التعليمات البرمجية بلغة Python المسمى **create-analyzer.py** المُرفق:

    ```
   code create-analyzer.py
    ```

    يتم فتح ملف التعليمات البرمجية بلغة Python في محرر التعليمات البرمجية.

1. راجع التعليمات البرمجية، والتي:
    - تحميل مخطط المحلل من الملف **biz-card.json**.
    - استرجاع بيانات نقطة النهاية، والمفتاح، واسم المحلل من ملف تكوين البيئة.
    - استدعاء دالة باسم **create_analyzer** وهي غير مُنفذة حاليًا

1. في الدالة **create_analyzer**، ابحث عن التعليق **إنشاء محلل Content Understanding** ثم أضِف التعليمات البرمجية التالية (تأكد من الحفاظ على المسافة البادئة الصحيحة):

    ```python
   # Create a Content Understanding analyzer
   print (f"Creating {analyzer}")

   # Set the API version
   CU_VERSION = "2025-05-01-preview"

   # initiate the analyzer creation operation
   headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/json"}

   url = f"{endpoint}/contentunderstanding/analyzers/{analyzer}?api-version={CU_VERSION}"

   # Delete the analyzer if it already exists
   response = requests.delete(url, headers=headers)
   print(response.status_code)
   time.sleep(1)

   # Now create it
   response = requests.put(url, headers=headers, data=(schema))
   print(response.status_code)

   # Get the response and extract the callback URL
   callback_url = response.headers["Operation-Location"]

   # Check the status of the operation
   time.sleep(1)
   result_response = requests.get(callback_url, headers=headers)

   # Keep polling until the operation is no longer running
   status = result_response.json().get("status")
   while status == "Running":
        time.sleep(1)
        result_response = requests.get(callback_url, headers=headers)
        status = result_response.json().get("status")

   result = result_response.json().get("status")
   print(result)
   if result == "Succeeded":
        print(f"Analyzer '{analyzer}' created successfully.")
   else:
        print("Analyzer creation failed.")
        print(result_response.json())
    ```

1. راجع التعليمات البرمجية التي أضفتها، والتي تقوم بما يلي:
    - إنشاء رؤوس مناسبة لطلبات REST
    - إرسال طلب HTTP من نوع *DELETE* لحذف المحلل إذا كان موجودًا مسبقًا.
    - إرسال طلب HTTP من نوع *PUT* لبدء إنشاء المحلل.
    - التحقق من الاستجابة لاسترداد عنوان URL الخاص بـ *Operation-Location* لرد الاتصال.
    - إرسال طلب HTTP من نوع *GET* بشكل متكرر إلى عنوان URL الخاص برد الاتصال للتحقق من حالة العملية حتى تتوقف عن العمل.
    - تأكيد نجاح (أو فشل) العملية للمستخدم.

    > **ملاحظة**: تتضمن التعليمات البرمجية بعض الفواصل الزمنية المقصودة لتجنّب تجاوز الحد الأقصى لمعدل الطلبات المسموح به من الخدمة.

1. استخدم الأمر **CTRL+S** لحفظ تغييرات التعليمات البرمجية، مع إبقاء جزء المحرر مفتوحًا في حال الحاجة إلى تصحيح أي أخطاء. قم بتغيير حجم الأجزاء بحيث تتمكن من رؤية جزء سطر الأوامر بوضوح.
1. في جزء سطر أوامر ِCloud Shell، أدخل الأمر التالي لتشغيل تعليمة Python البرمجية:

    ```
   python create-analyzer.py
    ```

1. راجع مخرجات البرنامج، والتي يُفترض أن تُشير إلى أنه تم إنشاء المحلل.

## استخدام واجهة REST API لتحليل المحتوى

الآن بعد أن قمت بإنشاء محلل، يمكنك استخدامه من خلال تطبيق عميل عبر واجهة برمجة تطبيقات Content Understanding REST.

1. في سطر أوامر Cloud Shell، أدخِل الأمر التالي لتحرير ملف التعليمات البرمجية **analyze_doc.py** بلغة Python المُرفق:

    ```
   code read-card.py
    ```

    يتم فتح ملف التعليمات البرمجية Python في محرر التعليمات البرمجية:

1. راجع التعليمات البرمجية، والتي:
    - تحديد ملف الصورة المطلوب تحليله، مع استخدام **biz-card-1.png** كقيمة افتراضية.
    - استرداد نقطة النهاية والمفتاح لمورد Azure AI Services من المشروع (باستخدام بيانات اعتماد Azure من جلسة Cloud Shell الحالية للمصادقة).
    - استدعاء دالة باسم **analyze_card** وهي غير مُنفذة حاليًا

1. في الدالة **analyze_card**، ابحث عن التعليق **استخدام Content Understanding لتحليل الصور** ثم أضِف التعليمات البرمجية التالية (تأكد من الحفاظ على المسافة البادئة الصحيحة):

    ```python
   # Use Content Understanding to analyze the image
   print (f"Analyzing {image_file}")

   # Set the API version
   CU_VERSION = "2025-05-01-preview"

   # Read the image data
   with open(image_file, "rb") as file:
        image_data = file.read()
    
   ## Use a POST request to submit the image data to the analyzer
   print("Submitting request...")
   headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/octet-stream"}
   url = f'{endpoint}/contentunderstanding/analyzers/{analyzer}:analyze?api-version={CU_VERSION}'
   response = requests.post(url, headers=headers, data=image_data)

   # Get the response and extract the ID assigned to the analysis operation
   print(response.status_code)
   response_json = response.json()
   id_value = response_json.get("id")

   # Use a GET request to check the status of the analysis operation
   print ('Getting results...')
   time.sleep(1)
   result_url = f'{endpoint}/contentunderstanding/analyzerResults/{id_value}?api-version={CU_VERSION}'
   result_response = requests.get(result_url, headers=headers)
   print(result_response.status_code)

   # Keep polling until the analysis is complete
   status = result_response.json().get("status")
   while status == "Running":
        time.sleep(1)
        result_response = requests.get(result_url, headers=headers)
        status = result_response.json().get("status")

   # Process the analysis results
   if status == "Succeeded":
        print("Analysis succeeded:\n")
        result_json = result_response.json()
        output_file = "results.json"
        with open(output_file, "w") as json_file:
            json.dump(result_json, json_file, indent=4)
            print(f"Response saved in {output_file}\n")

        # Iterate through the fields and extract the names and type-specific values
        contents = result_json["result"]["contents"]
        for content in contents:
            if "fields" in content:
                fields = content["fields"]
                for field_name, field_data in fields.items():
                    if field_data['type'] == "string":
                        print(f"{field_name}: {field_data['valueString']}")
                    elif field_data['type'] == "number":
                        print(f"{field_name}: {field_data['valueNumber']}")
                    elif field_data['type'] == "integer":
                        print(f"{field_name}: {field_data['valueInteger']}")
                    elif field_data['type'] == "date":
                        print(f"{field_name}: {field_data['valueDate']}")
                    elif field_data['type'] == "time":
                        print(f"{field_name}: {field_data['valueTime']}")
                    elif field_data['type'] == "array":
                        print(f"{field_name}: {field_data['valueArray']}")
    ```

1. راجع التعليمات البرمجية التي أضفتها، والتي تقوم بما يلي:
    - قراءة محتويات ملف الصورة
    - تعيين إصدار واجهة REST API الخاصة بـ Content Understanding لاستخدامها
    - إرسال طلب HTTP من النوع *POST* إلى نقطة نهاية Content Understanding، مع التوجيه بتحليل الصورة.
    - التحقق من الاستجابة الناتجة عن العملية لاسترداد مُعرّف عملية التحليل.
    - إرسال طلب HTTP من النوع *GET* بشكل متكرر إلى نقطة نهاية Content Understanding للتحقق من حالة العملية حتى تتوقف عن العمل.
    - في حال نجاح العملية، تُحفَظ الاستجابة بصيغة JSON، ثم يتم تحليلها لعرض القيم المستخرجة من كل حقل خاص بنوع معيّن.

    > **ملاحظة**: في مخطط بطاقة العمل البسيط، تكون جميع الحقول عبارة عن سلاسل نصية. توضح التعليمات البرمجية هنا الحاجة إلى التحقق من نوع كل حقل، حتى تتمكن من استخراج القيم ذات الأنواع المختلفة من مخطط أكثر تعقيدًا.

1. استخدم الأمر **CTRL+S** لحفظ تغييرات التعليمات البرمجية، مع إبقاء جزء المحرر مفتوحًا في حال الحاجة إلى تصحيح أي أخطاء. قم بتغيير حجم الأجزاء بحيث تتمكن من رؤية جزء سطر الأوامر بوضوح.
1. في جزء سطر أوامر ِCloud Shell، أدخل الأمر التالي لتشغيل تعليمة Python البرمجية:

    ```
   python read-card.py biz-card-1.png
    ```

1. راجع مخرجات البرنامج، والتي يُفترض أن تعرض القيم الخاصة بالحقول في بطاقة العمل التالية:

    ![بطاقة عمل تخص Roberto Tamburello، أحد موظفي Adventure Works Cycles.](./media/biz-card-1.png)

1. استخدم الأمر التالي لتشغيل البرنامج ببطاقة عمل مختلفة:

    ```
   python read-card.py biz-card-2.png
    ```

1. راجع النتائج، والتي يُفترض أن تعكس القيم الواردة في بطاقة العمل هذه:

    ![بطاقة عمل تخص Mary Duartes، موظفة في Contoso.](./media/biz-card-2.png)

1. في جزء سطر الأوامر في Cloud Shell، استخدم الأمر التالي لعرض الاستجابة الكاملة بصيغة JSON التي تم إرجاعها:

    ```
   cat results.json
    ```

    قم بالتمرير لعرض محتوى JSON.

## تنظيف

إذا انتهيت من العمل باستخدام خدمة "فهم المحتوى"، فيجب عليك حذف الموارد التي قمت بإنشائها في هذا التمرين لتجنب تكبد تكاليف Azure غير الضرورية.

1. في مدخل Microsoft Azure، احذف الموارد التي أنشأتها في أثناء هذا التمرين.
