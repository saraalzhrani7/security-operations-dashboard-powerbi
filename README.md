# لوحة معلومات أمنية (Security Operations Dashboard) — Power BI

لوحة Power BI تحوّل سجل تنبيهات SIEM (720 تنبيه أمني، يناير–أغسطس 2026) إلى مؤشرات إدارية: حجم الحوادث حسب الخطورة، متوسط وقت الاستجابة (MTTR)، ونسبة الإنذارات الكاذبة، مع ربط التنبيهات بتكتيكات إطار **MITRE ATT&CK**.

> البيانات محاكاة (simulated) بمعرفتي بأدوات Python، مبنية على بنية بيانات SIEM حقيقية — تصميم مقصود لإظهار فهمي لطبيعة بيانات مراكز العمليات الأمنية (SOC).

![Executive Overview](Executive%20Overview.png)
![Deep Dive](Deep%20Dive.png)

---

## الفكرة

فريق SOC يستقبل يوميًا مئات التنبيهات من أنظمة مختلفة. الإدارة لا تحتاج لمراجعة كل تنبيه على حدة، بل صورة عامة تجيب على:

- كم عدد التنبيهات حسب مستوى الخطورة، وكيف يتغير عبر الوقت؟
- كم متوسط وقت الاستجابة (MTTR)، وهل يختلف حسب الخطورة؟
- ما نسبة الإنذارات الكاذبة (False Positives)؟
- أي الأنظمة والأقسام الأكثر استهدافًا؟
- ما التكتيكات الأكثر تكرارًا وفق MITRE ATT&CK؟

---

## البيانات

جدول واحد (`security_alerts_dataset.csv`)، 720 صف، يغطي: معرّف التنبيه، تاريخ الاكتشاف والحل، مستوى الخطورة، نوع التنبيه، التكتيك حسب MITRE ATT&CK، أداة الكشف، الأنظمة والأقسام المستهدفة، المحلل المسؤول، وحالة التنبيه ووقت حله.

---

## مقاييس DAX الأساسية

```DAX
Total Alerts = COUNTROWS(security_alerts_dataset)

Critical Alerts = CALCULATE([Total Alerts], security_alerts_dataset[severity] = "Critical")

False Positive Rate =
DIVIDE(
    CALCULATE([Total Alerts], security_alerts_dataset[status] = "False Positive"),
    [Total Alerts]
)

Avg Resolution Time (Hours) =
AVERAGEX(
    FILTER(security_alerts_dataset, NOT(ISBLANK(security_alerts_dataset[resolution_time_hours]))),
    security_alerts_dataset[resolution_time_hours]
)

MTTR Critical =
CALCULATE(
    [Avg Resolution Time (Hours)],
    security_alerts_dataset[severity] = "Critical"
)

Alerts MoM Change =
VAR CurrentMonth = [Total Alerts]
VAR PrevMonth = CALCULATE([Total Alerts], DATEADD('DateTable'[Date], -1, MONTH))
RETURN DIVIDE(CurrentMonth - PrevMonth, PrevMonth)
```

بُنيت على نموذج بيانات بعلاقة بين جدول التنبيهات وجدول تقويم منفصل (Date Table)، لتفعيل Time Intelligence (اتجاه شهري، مقارنة فترات).

---

## اللوحة — صفحتان

**Executive Overview:** بطاقات KPI (إجمالي التنبيهات، التنبيهات الحرجة، متوسط وقت الاستجابة، نسبة الإنذارات الكاذبة)، اتجاه شهري للتنبيهات حسب الخطورة، توزيع حسب الحالة، وتوزيع حسب القسم.

**Deep Dive:** متوسط وقت الحل حسب الخطورة، مصفوفة تفاعلية (نظام مستهدف × خطورة) تبرز أكثر الأنظمة تعرضًا للهجوم، جدول تفصيلي للتنبيهات، وتحليل حسب تكتيكات MITRE ATT&CK.

---

## الأدوات

Power BI Desktop · DAX · Power Query · إطار MITRE ATT&CK
