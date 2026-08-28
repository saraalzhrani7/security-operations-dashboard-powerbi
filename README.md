
اربطي `DateTable[Date]` مع `security_alerts_dataset[date_detected]` (علاقة Many-to-One، اتجاه واحد).

---

## 4. مقاييس DAX

أنشئي جدول Measures فاضي (New Table → `Measures = {}` احذفي الصف الوهمي) واكتبي فيه المقاييس التالية:

```DAX
Total Alerts = COUNTROWS(security_alerts_dataset)

Critical Alerts = CALCULATE([Total Alerts], security_alerts_dataset[severity] = "Critical")

High Alerts = CALCULATE([Total Alerts], security_alerts_dataset[severity] = "High")

Open Alerts = CALCULATE([Total Alerts], security_alerts_dataset[status] = "In Progress")

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

Top Target System =
CALCULATE(
    VALUES(security_alerts_dataset[target_system]),
    TOPN(1, VALUES(security_alerts_dataset[target_system]), [Total Alerts])
)
```

هذي المقاييس تكفي لتغطية كل الأسئلة في القسم 1، وتقدرين تضيفين غيرها لاحقًا (زي Alerts by Analyst أو SLA Compliance %).

---

## 5. تصميم الصفحات

### صفحة 1 — نظرة عامة تنفيذية (Executive Overview)
- شريط بطاقات KPI أعلى الصفحة: `Total Alerts`, `Critical Alerts`, `Avg Resolution Time`, `False Positive Rate`
- مخطط خطي (Line Chart): اتجاه عدد التنبيهات شهريًا مقسّم حسب `severity`
- مخطط دائري أو Donut: توزيع التنبيهات حسب `status`
- مخطط أعمدة أفقي: التنبيهات حسب `department`

### صفحة 2 — تحليل تفصيلي (Deep Dive)
- مصفوفة (Matrix): `target_system` × `severity` مع عدد التنبيهات (يبرز أكثر نظام مستهدف)
- مخطط أعمدة: متوسط وقت الحل حسب `severity` (يبرز إذا فعلاً الحرج يُحل أسرع)
- جدول: أحدث التنبيهات مع `analyst_assigned` و`status`
- شريحة تصفية (Slicer): `detection_source`

---

## 6. نظام الألوان (متناسق مع هوية بروتفوليوك)

بما إن بروتفوليوك يستخدم لوحة داكنة أنيقة، خليها نفسها في اللوحة عشان تبين كوحدة واحدة عند عرضها كسكرين شوت في البروتفوليو:

| الاستخدام | اللون |
|---|---|
| خلفية أساسية | `#14120F` (الحبر الداكن) |
| بطاقات/عناصر | `#1E1B15` |
| لون التمييز الأساسي (Critical/تنبيه) | `#C9A24B` (الذهبي) |
| نص ثانوي | `#BDB6A4` |
| Critical | أحمر دافئ `#B5473A` |
| High | `#C9A24B` |
| Medium | `#8C8574` |
| Low | `#4A4030` |

---

## 7. النشر ورفعها على GitHub

1. أنشئي مستودع جديد باسم مشابه لأسلوبك الحالي، مثلًا: `security-operations-dashboard-powerbi`
2. ارفعي: ملف `.pbix`، `security_alerts_dataset.csv`، وهذا الملف كـ `README.md`
3. خذي 2-3 سكرين شوت للوحة (Executive Overview + Deep Dive) واحفظيهم كصور داخل مجلد `screenshots/` واربطيهم داخل الـ README حتى تظهر معاينة مباشرة بدون ما يحتاج أحد يفتح Power BI
4. لو ما عندك Power BI Service منشور، تقدرين تصدّرين اللوحة كـ PDF أو صور وترفعينها كمعاينة

---

## 8. نص جاهز لإضافته في صفحة البروتفوليو

هذا نص بنفس أسلوب المشروعين الموجودين عندك حاليًا، جاهز تلصقينه داخل `.proj-grid` بعد ما تخلصين المشروع (فقط عدّلي رابط GitHub):

```html
<div class="proj">
  <span class="pnum">03 / POWER BI × SOC</span>
  <h3>لوحة معلومات أمنية (Security Ops Dashboard)</h3>
  <p>لوحة Power BI تحوّل سجل تنبيهات SIEM محاكى (720 تنبيه) إلى مؤشرات إدارية: حجم الحوادث حسب الخطورة، متوسط وقت الاستجابة (MTTR)، ونسبة الإنذارات الكاذبة.</p>
  <ul>
    <li>مقاييس DAX مخصصة لـMTTR واتجاه الحوادث الشهري حسب الخطورة</li>
    <li>تحليل الأنظمة والأقسام الأكثر استهدافًا عبر مصفوفة تفاعلية</li>
  </ul>
  <a class="link" href="https://github.com/saraalzhrani7/security-operations-dashboard-powerbi" target="_blank">github.com/saraalzhrani7/security-ops-dashboard →</a>
</div>
```

---

## الخلاصة

هذا المشروع تحديدًا هو أقوى دليل ملموس على الجملة المكتوبة في أعلى صفحتك: "أحوّل البيانات المتناثرة إلى قرارات واضحة، وأراقب سلامتها بعقلية أمنية" — لأنه يطبقها حرفيًا بدل ما يبقى مجرد شعار.
