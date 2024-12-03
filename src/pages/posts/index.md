# نمذجة نظام طاقة شمسية منزلية

## إعداد وإشراف البحث

البحث من إعداد الطلاب: **[محمد معراوي](https://t.me/Mohammed_marawi)** (8) - **[محمد حمزة الشمالي](https://t.me/MohammadHamzaAl_Shamali)** (7) - **[عبد الرحمن رحال](mailto:arrahhal@proton.me)** (4)

إشراف الأستاذ: **نوّار مخلوف**

## محاور البحث

في هذا المشروع، قمنا بتطوير نموذج محاكاة لإدارة استهلاك الطاقة الشمسية ، والذي يحاكي نظامًا يتعامل مع توزيع الطاقة على أجهزة متعددة وفقًا لقدرة الطاقة الشمسية المتاحة. يتمثل الهدف الرئيسي في تحسين استغلال الطاقة وتقليل الهدر، مع ضمان تشغيل الأجهزة التي تتطلب طاقة مستمرة.

## منهجية البحث

اعتمدنا في هذا المشروع على تطبيق خطوات النمذجة والمحاكاة التي تم تناولها في المحاضرات، والتي يمكن تلخيصها كما يلي :

**وصف الحالة (System State):**

تمثل الحالة مجموعة المتغيرات التي تصف حالة النظام في لحظة معينة، مثل كمية الطاقة الشمسية المتوفرة (solarInput)، كمية الطاقة المستهلكة حاليًا (currentWatt)، وحالة كل جهاز (متصل، غير متصل، على الانتظار).

**ساعة المحاكاة (Simulation Clock):**
استخدمنا المتغير simTime كساعة محاكاة لتمثيل الزمن داخل النظام. يتحرك الزمن بشكل دوري وفقًا لفترات زمنية محسوبة باستخدام الدالة العشوائية expo لتوليد أحداث المحاكاة بشكل واقعي.

**قائمة الأحداث (Event List):**
تعتمد قائمة الأحداث في الكود على العمليات المختلفة مثل توصيل جهاز جديد أو فصل جهاز موجود. تم تنفيذ ذلك عبر دوال مثل connectingEvent وdisconnectingEvent التي تتعامل مع ترتيب الأحداث وتنفيذها بناءً على الوقت.

**الإجراءات المنطقية للأحداث (Event Routine):**
تتضمن الإجراءات المرتبطة بكل حدث اتخاذ قرارات بناءً على حالة النظام. على سبيل المثال، إذا كان هناك طاقة كافية، يتم توصيل الجهاز، وإذا لم تتوفر طاقة، يتم وضعه في قائمة الانتظار (onHoldDevices)

**إحصائيات الأداء (Statistical Counters):**
قمنا بجمع معلومات عن أداء النظام مثل كمية الطاقة المهدورة (wattWasted) والطاقة المطلوبة لتشغيل الأجهزة على الانتظار (wattNeeded)، مما يساعد في تقييم فعالية النظام.

 **تقرير المحاكاة (Report Routine):**
في نهاية المحاكاة، يتم توليد تقرير يوضح متوسط الطاقة المهدورة والطاقة المطلوبة، بالإضافة إلى بيانات حول أداء النظام طوال فترة التشغيل.

## أهمية المشروع

**تحسين استغلال الموارد:**
يوزع الطاقة الشمسية بشكل ديناميكي بين الأجهزة، مما يقلل الهدر ويضمن تشغيل الأجهزة الأساسية.

**التخطيط واتخاذ القرارات:**
 يوفر بيانات إحصائية تساعد على تقييم أداء النظام، مثل كمية الطاقة المهدورة والطاقة المطلوبة.

 **اختبار سيناريوهات مختلفة:**
 يتيح الكود تعديل المدخلات (مثل عدد الأجهزة وقدرتها) لفهم تأثير التغييرات على أداء النظام.

 **محاكاة نظام حقيقي:**
 يُستخدم الكود كنموذج أولي لفهم كيفية عمل الأنظمة الشمسية الحقيقية دون الحاجة إلى بناء النظام فعليًا.

## منهجية النتفيذ

### إضافة صف يمثل الجهاز الكهربائي

بدايًة ولكي نحاكي الأجهزة الكهربائية التي تتصل وتنفصل عن الكهرباء سنضيف صف ونسميه `device` وسنجعل من خصائصة الـ`status` وهي حالة الجهاز (متصل أو منفصل أو على الانتظار) وما تحتاجه من إجرائيات لتغيير هذه الحالة. وراعينا فيما إذا كان الجهاز يتصل بشكل دائم بالكهرباء بخاصية الـ `isAlwaysConnected` والوات الذي يستهلكه الجهاز عند اتصاله بخاصية الـ `wattUsage`.

```java
public enum DeviceStatus {
    CONNECTED, DISCONNECTED, ONHOLD
}

public class Device {
    DeviceStatus status;
    boolean isAlwaysConnected;
    int wattUsage;

    Device(int wattUsage) {
        this.isAlwaysConnected = false;
        this.status = DeviceStatus.DISCONNECTED;
        this.wattUsage = wattUsage;
    }

    public void connect() {
        this.status = DeviceStatus.CONNECTED;
    }
    public void disconnect() {
        if (!this.isAlwaysConnected)
            this.status = DeviceStatus.DISCONNECTED;
    }
    public void onhold() {
        this.status = DeviceStatus.ONHOLD;
    }
}
```



### إضافة الإجرائيات المطلوبة لتنفيذ النمذجة

سنتبع الأسلوب البرمجي الذي ينطلق من الإجرائيات العامة قبل أن يدخل في التفاصيل، وقد رأيناه مناسبًا لنعرف الخطوات والإجرائيات المحورية التي تمثل عملية النمذجة. سنكتب إجرائية تشغيل النمذجة ليوم واحد ولها الشكل التالي

```java
    public void run() throws IOException {
        // لجلب القيم من ملف الخل وإسنادها إلى المتحولات
        input();
        // ستجري النمذجة ليوم، وذلك عندما تصل قيمة ساعة النمذجة إلى 24 ساعة
        while (simTime < 24 * 60) {
            // هذه إجرائية تقابل إجرائية التايمين في كود المحاضرة الأولى وهي الحدث الذي يغير متحول الحالة كل مدة زمنية ما
            handleEvent();
            // تحديث الإحصائيات لتعكس التغييرات التي حصلت بعد الحدث
            update();
        }
        // إنشاء تقرير في ملف الخرج يمثل المعدلات الوسطية والنتائج التي جرت نمذجتها
        report();
    }
```

هذه الدالة سنستدعيها عند بداية تنفيذ البرنامج في إجرائية الـ `main`، على نهج المحاضرة الأولى نمذجة عملي:

```java
    public static void main(String[] args) {
        try {
            new SolarSim().run();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```



### التصريح عن متحولات الحالة ومتحولات أخرى مساعدة

سنصرح عن بعض المتحولات التي نريد أن تكون مشتركة بين جميع الإجرائيات، وبعضها ثوابت تحدد سلوك النمذجة كمؤقت الحدث، وهناك ما يلزم لنجعل إجابات النمذجة أكثر دقة وواقعية مثل الـ `onHoldDevices`، فلم نستخرج هذه الإجهزة من المصفوفة الأساس (`devices`) لأننا احتجنا إلى ترتيبهم بحسب الأقدمية، ثم متحولات الحالة لحساب الواط الفائض والواط الناقص وساعة النمذجة التي تجري على أساسها المحاكاة.

```java
    final double EVENT_INTERVAL = 60;
    final Random rand = new Random();

    int solarInput;
    List<Device> devices = new ArrayList<>();

    // store on hold devices indexes from oldest to newest
    List<Device> onHoldDevices = new ArrayList<>();

    int wattWasted = 0;
    int wattNeeded = 0;
    double simTime = 0.0;

    int numOfDevices = 0;

```

### الدخل

للدخل الشكل التالي:

الاستطاعة الداخلة من الألواح ← عدد الأجهزة الكهربائية ←استطاعة كل جهاز مقدرة بالواط ←1 الجهاز متصل دومًا أو 0 للجهاز التي تفصل وتوصل

وسنشرح ما يتعلق بالنمذجة من الأكواد، أما إجرائيات لغة البرمجة فلن نسهب في شرحها.

```java
    public void input() throws IOException {
        //  قارئ الدخل على غرار المحاضرة الأولى
        BufferedReader inFile = new BufferedReader(new FileReader("data/solar.in"));

        String[] params = inFile.readLine().trim().split("\\s+");

        // سنسند القيم من الدخل إلى المتحولات التي سبق وهيئناها في الصف الأساسي
        solarInput = Integer.parseInt(params[0]);
        numOfDevices = Integer.parseInt(params[1]);
        for (int idx = 2; idx < numOfDevices + 2; idx++) {
            int wattUsage = Integer.parseInt(params[idx]);
            Device device = new Device(wattUsage);
            devices.add(device);
        }

        for (int idx = numOfDevices + 2; idx < params.length; idx++) {
            if (params[idx].equals( "1")) {
                Device d = devices.get(idx - (numOfDevices + 2));
                d.isAlwaysConnected = true;
                if (d.wattUsage + currentWatt() <= solarInput) {
                    d.connect();
                } else {
                    d.onhold();
                    onHoldDevices.add(d);
                }
            }
        }
        inFile.close();
    }
```



### إجرائية الحدث

أول ما  يلزم هذه الإجرائية هو زيادة ساعة النمذجة بقدار قريب  من النصف ساعة، وقد استخدمنا إجرائية التقريب المستخدمة في المحاضرة الأولى لتنفيذ ذلك، ثم اخترنا حدثًا عشوائيًا (حدث فصل أو وصل) وستدعينا إجرائيته.

```java
    private void handleEvent() {
        simTime += expo(EVENT_INTERVAL);
        boolean isConnectEvent = rand.nextBoolean();

        if (isConnectEvent) {
            connectingEvent();
        } else {
            disconnectingEvent();
            // عند فصل جهاز ما، يصبح هناك استطاعة فائضة لذا نرى فيما إذا كان بالإمكان وصل جهاز كان على لائحة الانتظار
            connectingOnHoldEvent();
        }
    }
```



### إجرائيات الفصل والوصل

الإجرائيات الثلاثة التي استدعيناها في الحدث لها منطق قريب، لذا سنكتفي بشرح واحدة وهي إجرائية الوصل. وهي تختار جهازًا غير متصل **عشوائيا** وتختبر إمكانية وصله وإلا فتضعه على لائحة الانتظار، الاختيار العشوائي للجهاز مهم جدًا ولولاه لدخلت النمذجة في حلقة ولكانت نتائج المحاكاة خطأ.

```java
    private void connectingEvent() {
        List<Device> disconnectedDevices = devices.stream()
                .filter(d -> d.status == DeviceStatus.DISCONNECTED)
                .toList();
        if (disconnectedDevices.isEmpty()) return;

        Device randomDevice = disconnectedDevices.get(rand.nextInt(disconnectedDevices.size()));
        if (currentWatt() + randomDevice.wattUsage <= solarInput) {
            randomDevice.connect();
        } else {
            randomDevice.onhold();
            // عند وصل جهاز من لائحة الانتظار فإن الجهاز الأقدم له الأولوية
            onHoldDevices.add(randomDevice);
        }
    }
```



### تحديث قيمة متحولات الحالة بعد الحدث

وهي مهمة إجرائية الـ `update`، فإنها تحسب الاستطاعة الناقصة أو الفائضة وتضيفها إلى المتحولات المقابلة لها.

```java
    private void update() {
        // إذا كانت غير فارغة فهذا احتياج للمزيد من استطاعة الدخل
        if (!onHoldDevices.isEmpty()) {
            int overload = onHoldDevices.stream().mapToInt(d -> d.wattUsage).sum();
            wattNeeded += overload;
        }
        else {
            wattWasted += solarInput - currentWatt();
        }
    }
```



### إنشاء تقرير بنتائج المحاكاة

في التقرير سنذكر القيم المدخلة ومتوسط نتائج المحاكاة يوميًا. وسنذكر في الفقرة القادمة نموذجًا للدخل والخرج يوضح نتيجة التنفيذ.

```java
    public void report() throws IOException {
        BufferedWriter outFile = new BufferedWriter(new FileWriter("data/solar.out"));
        outFile.write("Solar Input: " + solarInput + " watts\n" +
                "Number of Devices: " + numOfDevices + "\n" +
                "Device Power Consumption: " +
                devices.stream().map(d -> String.valueOf(d.wattUsage)).collect(Collectors.joining(" ")) + "\n" +
                "Always Connected: " +
                devices.stream().map(d -> String.valueOf(d.isAlwaysConnected)).collect(Collectors.joining(" ")) + "\n\n");

        outFile.write("Simulation Report\n" +
                "Average Overload (Watts): " + wattNeeded / 24+ "\n" +
                "Average Wasted Solar Input (Watts): " + wattWasted / 24 + "\n");

        outFile.close();
    }
```

## نماذج عن تنفيذ المحاكاة على عدد الإدخالات

1. بيت فيه جهازين متصلين بشكل دائم وجهازين في حالة قطع وفصل، قيم الاستطاعة موضحة في الدخل وسنرى ما يحتاج من استطاعة دخل حسب برمجية المحاكاة

```
input:
0 4 400 400 100 200 1 1 0 0
output:
Solar Input: 0 watts
Number of Devices: 4
Device Power Consumption: 400 400 100 200
Always Connected: true true false false

Simulation Report
Average Overload (Watts): 1066
Average Wasted Solar Input (Watts): 0
```

وهي نتيجة قريبة من المتوقع.

2. جهاز فيه ستة أجهزة واثنين منها دائمة الاتصال بالكهرباء، واستطاعة هذه الأجهزة موضحة في الأدنى.

```
Solar Input: 1500 watts
Number of Devices: 6
Device Power Consumption: 400 400 600 50 200 200
Always Connected: true true false false false false

Simulation Report
Average Overload (Watts): 243
Average Wasted Solar Input (Watts): 75
```

