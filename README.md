# مشروع: تحويل مستودع أغذية إلى مستودع آلي (Unmanned Warehouse)
الوصف: هذا المستند يحتوي على خوارزمية تنفيذية مفصّلة، متطلبات، عناصر Working Envelope و Working Area بصيغة قابلة للاستخدام في إعدادات النظام.

---

## الهدف
أتمتة كامل سلسلة التخزين والتقاط وتعبئة وتجهيز الشحنات داخل مستودع أغذية بحيث يعمل بنظام روبوتي (AMR/ASRS/Robotic Arm + conveyors) دون تدخل بشري أثناء التشغيل الاعتيادي، مع الحفاظ على معايير السلامة والامتثال للمتطلبات الصحية للمواد الغذائية.

---

## المحتويات
1. خوارزمية (خطوات مفصّلة)
2. عناصر Working Envelope (مغلف/حيز العمل) — تعريفات ومخطط بيانات جاهز
3. عناصر Working Area (منطقة العمل) — تعريفات ومخطط بيانات جاهز
4. مثال تكوين (YAML) لاستخدامه في برمجيات الملاحة/التخطيط
5. هيكل مجلدات مقترح وملفات مهمة

---

# 1) الخوارزمية — خطوات التنفيذ (مفصّلة)
> كل خطوة فيها مهام فرعية، مخرجات متوقعة، ونماذج للقياسات.

### 1. تحديد متطلبات العمل (Requirements)
- مدخلات: معدل الطلب/ساعة (throughput)، عدد SKU، أوزان وأحجام العناصر، معدل الرفع الأقصى، شروط التخزين (تبريد/مجمد/جاف)، أوقات العمل، ميزانية، قيود سرعة التسليم.
- مخرجات: مواصفات وظيفية (Functional Spec) — مثال: orders_per_hour=200, avg_item_weight=0.8kg, max_payload=25kg.

### 2. مسح الموقع (Site Survey)
- قياس أبعاد المستودع: طـول × عرض × ارتفاع سقف، عرض الممرات، ارتفاع الرفوف، مواقع الأعمدة، أبواب، ساحات التحميل، ممرات المشاة.
- فحص أرضية: نوع السطح، انحدار، عوائق، دويلات كهرباء، شبكات Wi-Fi.
- مخرجات: خريطة ثنائية/ثلاثية الأبعاد (CAD / floorplan)، قائمة “نقاط عائق” (obstacle list).

### 3. اختر بنية النظام (System Architecture)
- قرر: AMR (متنقل حر) أم AGV (مسار ثابت) أم AS/RS (نظام رف آلي ثابت) أم هجين.
- عناصر محتملة: روبوتات قاعدية متنقلة، ذراع آلية للتقاط، منصات لرفع البالات، أحزمة/قناطر، محطات تغليف، محطات شحن.
- مخرجات: مخطط Block Diagram وبيان واجهات (APIs) مع WMS/ERP.

### 4. تصميم الميكانيكا والكهرباء (Mechanical / Electrical)
- اختر: نوع سواعد (6-DOF أو SCARA أو 4DOF)، نوع المقبض (gripper) — vacuum / two-finger / multi-finger.
- اختر قاعدة: differential / mecanum / omni (اعتمادًا على الحاجة للمناورة).
- البطاريات: نوع، وقت التشغيل، نظام شحن أوتوماتيكي (docking).
- مخرجات: BOM (قائمة قطع) ومخططات كهربائية.

### 5. القياسات الهندسية: حساب Working Envelope و Working Area
- احسب مدى ذراع الالتقاط، حدود الوصول، نصف قطر الدوران، clearance للأرفف، نقاط التحميل/التفريغ.
- ضع خرائط no-go zones ومناطق السرعة المنخفضة.

### 6. الحساسات والملاحة (Perception & Navigation)
- أجهزة: LiDAR لخرائط SLAM، كاميرات RGB-D للالتقاط والتحقق، IMU، مستشعرات القرب، مستشعرات قوة بالمقبض (force/torque).
- خوارزميات: SLAM، localization (AMCL/particle filter)، global path planning (A* / Dijkstra)، local planner (DWA).

### 7. برمجة النظام والتكامل (Software)
- طبقات: 
  - Middleware (ROS2 یا custom),
  - Fleet manager (تنسيق المهام بين الروبوتات),
  - WMS connector (REST / MQTT),
  - Safety supervisor (E-stop, Safety scanner interface).
- واجهات: حالة المهمة (TASK_STATUS), تقارير الأعطال, telemetry.

### 8. محاكاة واختبار (Simulation & Validation)
- استخدم Gazebo / Webots / Coppelia / Digital Twin.
- اختبر السيناريوهات: ازدحام، فقدان GPS داخلي، طوارئ، تبدّل أرفف.

### 9. نشر تدريجي (Pilot → Rollout)
- مرحلة Pilot على ممر/منطقة واحدة.
- قياس KPIs: fulfillment time, uptime, error rate, energy consumption.

### 10. السلامة والجودة (Safety & QA)
- نظام E-stop، حواجز ضوئية، مسارات منفصلة للمشاة، لوحات تحذير صوتية ومرئية، سياسات فشل آمن.

### 11. التشغيل والصيانة (Ops & Maintenance)
- جداول صيانة دورية، تسجيل الأخطاء، تحديث خرائط، تدريب فريق دعم، قطع غيار.

### 12. تحسين وقياس الأداء (Continuous Improvement)
- اجمع telemetries، حسّن مسارات، ضبط سرعة، خوارزميات التقاط أفضل، تعلم من الأخطاء.

---

# 2) Working Envelope (مغلف/حيز العمل) — العناصر الواجب توثيقها
تعريف: الحيز الثلاثي الأبعاد الذي يمكن لأنامل الروبوت (end effector) الوصول إليه بأمان، مع مراعاة قيود المفاصل والملحقات.

عناصر يجب توثيقها:
- envelope_id : معرف الحيز
- x_min, x_max (m) — مدى على المحور X (عالمي بالنسبة لقاعده الروبوت أو مرجع الرف)
- y_min, y_max (m)
- z_min, z_max (m)
- reach_radius_max (m) — أقصى نصف قطر وصول
- payload_max (kg) — حمولة قصوى ضمن هذا الحيز- 
joint_limits — جدول قيود مفاصل الذراع (deg أو rad)
- orientation_limits — حدود الزوايا (roll/pitch/yaw)
- clearance_required (m) — مسافة أمان من العوائق
- speed_limit (m/s) — حد السرعة داخل الحيز
- accel_limit (m/s²)
- temperature_range (°C) — إذا متعلق بمأكولات مبردة
- notes — ملاحظات خاصة (مثال: لا يصلح للرَف رقم 3 لوجود مواسير)

مثال بيانات (JSON-like):
working_envelope:
  envelope_id: "arm_zone_01"
  reference_frame: "base_link"
  x_min: -0.50
  x_max: 1.20
  y_min: -0.80
  y_max: 0.80
  z_min: 0.10
  z_max: 2.10
  reach_radius_max: 1.35
  payload_max: 15.0
  joint_limits:
    shoulder: [-150, 150]
    elbow: [-120, 120]
    wrist: [-180, 180]
  orientation_limits:
    roll: [-90,90]
    pitch: [-75,75]
    yaw: [-180,180]
  clearance_required: 0.15
  speed_limit: 0.5
  accel_limit: 1.0
  temperature_range: [2, 25]
  notes: "حظر العمل عند رف رقم 4 بسبب مواسير تبريد خلفية"

---

# 3) Working Area (منطقة العمل) — العناصر الواجب توثيقها
تعريف: المساحة الأرضية والبيئة التشغيلية التي يتحرك فيها الروبوت (ممرات، محطات تحميل، مناطق صيانة).

عناصر يجب توثيقها:
- area_id
- footprint (length x width) للمتحرك (m)
- turning_radius (m)
- aisle_width_min (m) — الحد الأدنى لممرات الحركة
- rack_layout — إحداثيات الرفوف (rows, columns, bay numbers)
- picking_stations — قائمة محطات التعبئة (x,y,heading)
- loading_docks — نقاط التحميل والتفريغ
- charging_stations — مواقع وواجهة الإرساء
- maintenance_zones
- human_access_zones — مناطق مخصصة للعاملين
- no_go_zones — مناطق ممنوعة
- floor_type — concrete/epoxy/tiles ودرجة الانحدار القصوى
- lighting_lux — قياس الإضاءة إن لزم
- network_coverage — نقاط Wi-Fi أو تغطية 4G

مثال بيانات (YAML-like):
working_area:
  area_id: "warehouse_floor_a"
  footprint:
    length: 45.0
    width: 22.0
  turning_radius: 0.75
  aisle_width_min: 1.20
  rack_layout:
    - row: 1
      bays: 12
      start_coord: [2.0, 2.0]
      bay_spacing: 1.2
    - row: 2
      bays: 12
      start_coord: [2.0, 5.0]
  picking_stations:
    - id: "pack_01"
      coord: [40.5, 3.0]
      heading: 90
  loading_docks:
    - id: "dock_A"
      coord: [0.5, 11.0]
  charging_stations:
    - id: "charge_1"
      coord: [1.0, 1.0]
      connector_type: "pogo"
  maintenance_zones:
    - id: "maint_01"
      coord: [1.0, 20.0]
  no_go_zones:
    - polygon: [[10,10],[12,10],[12,12],[10,12]]
  floor_type: "epoxy"
  max_slope_percent: 2.0
  network_coverage: "wifi_ap_1,wifi_ap_2"

---

# 4) خوارزمية تشغيل روبوت (Pseudo-Code)# PSEUDO: loop تشغيل المهمة الأساسية للمركبة/الذراع
while True:
    task = fleet_manager.request_next_task()
    if not task:
        vehicle.go_to(charging_station) if battery_low() else vehicle.idle()
        continue

    # تنسيق المهمة مع WMS
    pick_location = WMS.get_pick_location(task.sku)
    place_station = WMS.get_pack_station(task.order_id)

    # تخطيط المسار
    path = planner.compute_path(current_pose, pick_location)
    nav.follow(path)

    # التمركز والالتقاط
    manip.align_to(pick_location)
    success = manipulator.pick(sku_id=task.sku, grip_params=task.grip_profile)
    if not success:
        report_failure(task, reason="pick_failed")
        continue

    # نقل للمحطة
    path = planner.compute_path(current_pose, place_station)
    nav.follow(path)
    manip.place(place_station)

    # تحديث حالة في WMS
    WMS.update_task_status(task.id, "done")
    telemetry.log(task.id, energy_used, time_taken)

---

# 5) هيكل مجلدات README المقترح للمشروع/project-root
  /hardware        # رسومات، BOM، مخططات كهربية
  /software
    /ros_pkg
    /fleet_manager
    /wms_connector
  /configs
    workspace_config.yaml   # (ملف مثال بالـ YAML أعلاه)
  /sim             # ملفات محاكاة Gazebo / URDF
  /docs            # خرائط الموقع، تقارير المسح
  README.md        # هذا الملف

---

# 6) قائمة تحقق سريعة قبل التشغيل (Quick Checklist)
- [ ] قياسات الممرات والارتفاع مدخلة في workspace_config.yaml
- [ ] اختبار SLAM في الليل والنهار
- [ ] مثبتات سلامة E-stop تعمل
- [ ] نقاط شحن مثبتة ومجربة
- [ ] محاكاة كاملة لسيناريوهات الذروة
- [ ] ربط مع WMS/ERP وواجهات اختبارية

---
