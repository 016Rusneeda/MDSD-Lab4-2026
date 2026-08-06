# 📱 ใบงานปฏิบัติ สัปดาห์ที่ 4
# Flutter Layout & Navigation — Multi-Screen Travel App

> **รายวิชา:** การพัฒนาซอฟต์แวร์สำหรับอุปกรณ์เคลื่อนที่  
> **สัปดาห์ที่:** 4 | **เวลา:** 3.5 ชั่วโมง  
> **เครื่องมือ:** Flutter SDK, Dart, VS Code / Android Studio, Go Router  

---

## 📋 วัตถุประสงค์การเรียนรู้

เมื่อจบใบงานนี้ นักศึกษาจะสามารถ:

1. ใช้ Layout Widgets หลัก (`Row`, `Column`, `Stack`, `GridView`, `ListView`) ได้อย่างถูกต้อง
2. สร้าง Responsive Layout ที่ปรับขนาดตามหน้าจอโดยอ้างอิงมาตรฐาน Material Design 3 (Window Size Classes) ด้วย `LayoutBuilder` และ `MediaQuery`
3. ตั้งค่า Navigation แบบ Multi-screen ด้วย **Go Router** พร้อม Named Routes และ `StatefulShellRoute`
4. ส่งผ่านข้อมูล (Arguments / Path Parameters) ระหว่าง Screen และจัดการ Fallback กรณี Deep Link / Web Refresh ได้อย่างถูกต้อง
5. ออกแบบ Navigation Hierarchy ที่เหมาะสมสำหรับ Mobile และ Web Application

---

## 🧪 ทฤษฎีก่อนการทดลอง

### ส่วนที่ 1 — Flutter Layout System & Material Design 3 Breakpoints

Flutter ใช้ **Constraint-based Layout** โดยมีกระบวนการทำงาน 3 ขั้น:

```
Parent → ส่ง Constraints (min/max width/height) ลงไป → Child
Child  → คำนวณขนาดตัวเอง → แจ้ง Size กลับ → Parent
Parent → จัดวางตำแหน่ง Child ตาม Size ที่ได้รับ
```

**Widget หลักที่ใช้ใบงานนี้:**

| Widget | หน้าที่ | คุณสมบัติสำคัญ |
|---|---|---|
| `Row` | จัดวาง Children แนวนอน | `mainAxisAlignment`, `crossAxisAlignment` |
| `Column` | จัดวาง Children แนวตั้ง | `mainAxisAlignment`, `crossAxisAlignment` |
| `Stack` | วาง Children ซ้อนกัน (Z-axis) | `alignment`, `fit` |
| `Expanded` | ยืดให้เต็มพื้นที่ใน Row/Column | `flex` (กำหนดสัดส่วน) |
| `Flexible` | ยืดได้แต่ไม่บังคับให้เต็ม | `flex`, `fit` |
| `SizedBox` | กำหนดขนาดตายตัว / ช่องว่าง | `width`, `height` |
| `Padding` | เพิ่ม Padding รอบ Child | `EdgeInsets` |
| `Container` | Box Model ครบ (padding, margin, border, color) | หลายคุณสมบัติ |
| `GridView` | แสดง Items เป็น Grid | `crossAxisCount`, `crossAxisSpacing` |
| `ListView` | แสดง Items เป็น List แบบ Scrollable | `builder`, `itemCount` |
| `LayoutBuilder` | รับ Constraints ของ Parent เพื่อทำ Responsive | `BoxConstraints` |

**Alignment ใน Row และ Column:**

```
MainAxis = แกนหลัก       CrossAxis = แกนตั้งฉาก

Row:    MainAxis = แนวนอน (←→)   CrossAxis = แนวตั้ง (↑↓)
Column: MainAxis = แนวตั้ง (↑↓)   CrossAxis = แนวนอน (←→)

MainAxisAlignment:
  .start          .center         .end
  .spaceBetween   .spaceAround    .spaceEvenly

CrossAxisAlignment:
  .start    .center    .end    .stretch    .baseline
```

**ตัวอย่าง Expanded กับ flex:**

```dart
Row(
  children: [
    Expanded(flex: 1, child: Container(color: Colors.red)),    // 1/3
    Expanded(flex: 2, child: Container(color: Colors.blue)),   // 2/3
  ],
)
```

**📐 มาตรฐาน Window Size Classes ของ Material Design 3 (M3):**

ในการออกแบบ Responsive Layout บน M3 จะใช้ความกว้างหน้าจอ (Width Breakpoints) แบ่งออกเป็น 3 ระดับหลัก:

- **Compact (< 600 dp):** มือถือแนวตั้ง (Phone Portrait) — แนะนำใช้ Grid 2 Columns หรือ List 1 Column, ใช้ Bottom Navigation Bar
- **Medium (600 dp – 839 dp):** แท็บเล็ตแนวตั้ง / มือถือพับได้ (Tablet Portrait / Phone Landscape) — แนะนำใช้ Grid 3 Columns, ใช้ Navigation Rail
- **Expanded (≥ 840 dp):** แท็บเล็ตแนวนอน / หน้าจอคอมพิวเตอร์ (Tablet Landscape / Desktop) — แนะนำใช้ Grid 4 Columns ขึ้นไป, ใช้ Navigation Drawer

---

### ส่วนที่ 2 — Go Router & Deep Linking

**Go Router** คือ Package Navigation ที่ Google แนะนำสำหรับ Flutter ซึ่งใช้ **Declarative / URL-based Navigation (Navigator 2.0)** เหมาะกับทั้ง Mobile และ Web

**แนวคิดหลัก:**

```
GoRouter (Router Configuration)
└── StatefulShellRoute (Bottom Navigation Wrapper)
    ├── Branch 0: GoRoute path: '/'                  → HomeScreen
    ├── Branch 1: GoRoute path: '/explore'           → ExploreScreen
    │   └── GoRoute path: 'destinations/:id'        → DestinationDetailScreen (รับ param :id)
    ├── Branch 2: GoRoute path: '/saved'             → SavedScreen
    └── Branch 3: GoRoute path: '/profile'           → ProfileScreen
```

**คำสั่งการนำทางที่สำคัญ:**

```dart
context.go('/explore')                                // เปลี่ยนไปหน้า /explore ตามโครงสร้าง Route Tree
context.push('/explore/destinations/1')               // Push หน้าใหม่ซ้อนทับขึ้นไปบน Stack (มีปุ่มย้อนกลับ)
context.pushNamed('destination-detail', pathParameters: {'id': '1'}) // เรียกผ่าน Named Route (ลดความเสี่ยงพิมพ์ Path ผิด)
context.pop()                                         // ย้อนกลับหน้าก่อนหน้า
```

**ความแตกต่างระหว่าง `ShellRoute` และ `StatefulShellRoute`:**

- **`ShellRoute`:** เมื่อผู้ใช้สลับ Tab ตัวหน้าเก่าจะถูกทำลาย (Destroy) ทำให้ State และตำแหน่ง Scroll หายไป
- **`StatefulShellRoute.indexedStack`:** รักษาสภาพหน้าและ State ของแต่ละ Tab ไว้ (Keep-Alive) เมื่อสลับ Tab กลับมา หน้าเดิมจะยังอยู่ตำแหน่งเดิม

> **⚠️ ข้อระวังการใช้ `extra` และ Deep Link:**
> การส่ง Object ผ่าน `extra` (เช่น `context.pushNamed('detail', extra: myObject)`) ช่วยให้ส่งข้อมูลข้ามหน้าได้สะดวก แต่หากผู้ใช้กด Refresh หน้าเว็บ หรือเปิดผ่าน Deep Link URL ตรง ๆ ค่า `extra` จะกลายเป็น `null` ดังนั้นในการทำงานจริง ต้องดึง `pathParameters['id']` มาใช้ค้นหาข้อมูลสำรอง (Fallback) เสมอ

---

## 🗂️ โครงสร้างโครงงานที่จะสร้าง

```
travel_app/
├── lib/
│   ├── main.dart                    ← Entry Point + Router Config
│   ├── router/
│   │   └── app_router.dart          ← GoRouter Setup (StatefulShellRoute + Named Routes)
│   ├── models/
│   │   └── destination.dart         ← Data Model & Sample Data
│   ├── screens/
│   │   ├── home_screen.dart         ← หน้าหลัก + Featured Items
│   │   ├── explore_screen.dart      ← รายการ Destination (Responsive Grid)
│   │   ├── destination_detail_screen.dart ← รายละเอียดสถานที่
│   │   ├── saved_screen.dart        ← รายการที่บันทึก
│   │   └── profile_screen.dart      ← Profile
│   └── widgets/
│       └── destination_card.dart    ← Reusable Card Widget
└── pubspec.yaml
```

---

## 🔬 การทดลอง

---

### การทดลองที่ 1 — สร้าง Project และ Setup Dependencies

**เวลาโดยประมาณ:** 15 นาที

#### ขั้นตอนที่ 1.1 — สร้าง Flutter Project ใหม่

```bash
flutter create travel_app
cd travel_app
```

#### ขั้นตอนที่ 1.2 — เพิ่ม Dependencies

เปิดไฟล์ `pubspec.yaml` แล้วแก้ไขส่วน `dependencies`:

```yaml
dependencies:
  flutter:
    sdk: flutter
  go_router: ^14.0.0
  cupertino_icons: ^1.0.8
```

บันทึกไฟล์แล้วรันคำสั่ง:

```bash
flutter pub get
```

ตรวจสอบว่าดาวน์โหลดสำเร็จ:

```bash
flutter pub deps | grep go_router
# ควรเห็น: go_router x.x.x
```

#### ขั้นตอนที่ 1.3 — สร้างโครงสร้าง Directory

```bash
mkdir -p lib/router lib/models lib/screens lib/widgets
```

---

### การทดลองที่ 2 — สร้าง Data Model

**เวลาโดยประมาณ:** 10 นาที

#### ขั้นตอนที่ 2.1 — สร้าง Destination Model

สร้างไฟล์ `lib/models/destination.dart`:

```dart
class Destination {
  final String id;
  final String name;
  final String country;
  final String description;
  final String imageUrl;
  final double rating;
  final int price; // ราคาโดยประมาณ (USD/คืน)
  final List<String> tags;

  const Destination({
    required this.id,
    required this.name,
    required this.country,
    required this.description,
    required this.imageUrl,
    required this.rating,
    required this.price,
    required this.tags,
  });
}

// ข้อมูลตัวอย่าง
final List<Destination> sampleDestinations = [
  Destination(
    id: '1',
    name: 'กรุงเทพฯ',
    country: 'ไทย',
    description:
        'เมืองหลวงที่คึกคักของไทย เต็มไปด้วยวัดวาอาราม อาหารริมทาง และชีวิตยามค่ำคืนที่ไม่รู้จบ',
    imageUrl: 'https://picsum.photos/seed/bangkok/400/300',
    rating: 4.7,
    price: 50,
    tags: ['วัด', 'อาหาร', 'ช้อปปิ้ง'],
  ),
  Destination(
    id: '2',
    name: 'เชียงใหม่',
    country: 'ไทย',
    description:
        'เมืองทางเหนือที่ล้อมรอบด้วยภูเขา วัดโบราณ และวัฒนธรรมล้านนา',
    imageUrl: 'https://picsum.photos/seed/chiangmai/400/300',
    rating: 4.8,
    price: 35,
    tags: ['ธรรมชาติ', 'วัฒนธรรม', 'Trekking'],
  ),
  Destination(
    id: '3',
    name: 'ภูเก็ต',
    country: 'ไทย',
    description: 'เกาะที่สวยงามที่สุดของไทย มีหาดทรายขาว น้ำทะเลใส และกิจกรรมดำน้ำ',
    imageUrl: 'https://picsum.photos/seed/phuket/400/300',
    rating: 4.6,
    price: 80,
    tags: ['ทะเล', 'ดำน้ำ', 'รีสอร์ท'],
  ),
  Destination(
    id: '4',
    name: 'โตเกียว',
    country: 'ญี่ปุ่น',
    description: 'มหานครที่ผสมผสานความทันสมัยและวัฒนธรรมดั้งเดิมได้อย่างลงตัว',
    imageUrl: 'https://picsum.photos/seed/tokyo/400/300',
    rating: 4.9,
    price: 120,
    tags: ['เทคโนโลยี', 'อาหาร', 'อนิเมะ'],
  ),
  Destination(
    id: '5',
    name: 'บาหลี',
    country: 'อินโดนีเซีย',
    description: 'เกาะแห่งพระเจ้า เต็มไปด้วยวัดและชายหาดสวยงาม',
    imageUrl: 'https://picsum.photos/seed/bali/400/300',
    rating: 4.7,
    price: 60,
    tags: ['วัฒนธรรม', 'ทะเล', 'Yoga'],
  ),
  Destination(
    id: '6',
    name: 'สิงคโปร์',
    country: 'สิงคโปร์',
    description: 'นครรัฐที่สะอาด ทันสมัย และปลอดภัย มีอาหารหลากหลายวัฒนธรรม',
    imageUrl: 'https://picsum.photos/seed/singapore/400/300',
    rating: 4.8,
    price: 150,
    tags: ['ช้อปปิ้ง', 'อาหาร', 'สวนสนุก'],
  ),
];
```

> **📌 สังเกต:** เราใช้ `const` constructor เพราะ Destination ไม่เปลี่ยนแปลงหลังสร้าง (Immutable)

---

### การทดลองที่ 3 — สร้าง Reusable Widget

**เวลาโดยประมาณ:** 20 นาที

#### ขั้นตอนที่ 3.1 — สร้าง DestinationCard Widget

สร้างไฟล์ `lib/widgets/destination_card.dart`:

```dart
import 'package:flutter/material.dart';
import '../models/destination.dart';

class DestinationCard extends StatelessWidget {
  final Destination destination;
  final VoidCallback onTap;

  const DestinationCard({
    super.key,
    required this.destination,
    required this.onTap,
  });

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: onTap,
      child: Container(
        // ── 1. Decoration: rounded corner + shadow ──────────────
        decoration: BoxDecoration(
          color: Colors.white,
          borderRadius: BorderRadius.circular(16),
          boxShadow: const [
            BoxShadow(
              color: Colors.black12,
              blurRadius: 8,
              offset: Offset(0, 4),
            ),
          ],
        ),
        // ── 2. ClipRRect ป้องกัน Image เกิน Rounded Corner ──────
        child: ClipRRect(
          borderRadius: BorderRadius.circular(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              // ── 3. Stack: Image + Rating Badge ────────────────
              Stack(
                children: [
                  // 3a. รูป Destination
                  AspectRatio(
                    aspectRatio: 16 / 9,
                    child: Image.network(
                      destination.imageUrl,
                      fit: BoxFit.cover,
                      errorBuilder: (ctx, err, _) => Container(
                        color: Colors.grey.shade200,
                        child: const Icon(Icons.image_not_supported, size: 48),
                      ),
                    ),
                  ),
                  // 3b. Badge Rating (ซ้อนบนรูป)
                  Positioned(
                    top: 8,
                    right: 8,
                    child: Container(
                      padding: const EdgeInsets.symmetric(
                        horizontal: 8,
                        vertical: 4,
                      ),
                      decoration: BoxDecoration(
                        color: Colors.black.withValues(alpha: 0.6),
                        borderRadius: BorderRadius.circular(12),
                      ),
                      child: Row(
                        mainAxisSize: MainAxisSize.min,
                        children: [
                          const Icon(Icons.star, color: Colors.amber, size: 14),
                          const SizedBox(width: 4),
                          Text(
                            destination.rating.toString(),
                            style: const TextStyle(
                              color: Colors.white,
                              fontSize: 12,
                              fontWeight: FontWeight.bold,
                            ),
                          ),
                        ],
                      ),
                    ),
                  ),
                ],
              ),
              // ── 4. Info Section ────────────────────────────────
              Padding(
                padding: const EdgeInsets.all(12),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    // ชื่อและราคา
                    Row(
                      mainAxisAlignment: MainAxisAlignment.spaceBetween,
                      children: [
                        Expanded(
                          child: Text(
                            destination.name,
                            style: const TextStyle(
                              fontSize: 16,
                              fontWeight: FontWeight.bold,
                            ),
                            overflow: TextOverflow.ellipsis,
                          ),
                        ),
                        Text(
                          '\$${destination.price}/คืน',
                          style: TextStyle(
                            fontSize: 14,
                            color: Theme.of(context).primaryColor,
                            fontWeight: FontWeight.w600,
                          ),
                        ),
                      ],
                    ),
                    const SizedBox(height: 4),
                    // แสดงประเทศ
                    Row(
                      children: [
                        const Icon(Icons.location_on,
                            size: 14, color: Colors.grey),
                        const SizedBox(width: 2),
                        Text(
                          destination.country,
                          style: const TextStyle(
                            fontSize: 13,
                            color: Colors.grey,
                          ),
                        ),
                      ],
                    ),
                    const SizedBox(height: 8),
                    // Tags
                    Wrap(
                      spacing: 6,
                      children: destination.tags
                          .map(
                            (tag) => Chip(
                              label: Text(
                                tag,
                                style: const TextStyle(fontSize: 11),
                              ),
                              materialTapTargetSize:
                                  MaterialTapTargetSize.shrinkWrap,
                              visualDensity: VisualDensity.compact,
                              backgroundColor: Colors.blue.shade50,
                              shape: const StadiumBorder(
                                  side: BorderSide(color: Colors.transparent)),
                              padding: EdgeInsets.zero,
                            ),
                          )
                          .toList(),
                    ),
                  ],
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

> **📌 สังเกตการใช้ Layout:**
> - `Column` → จัดภาพและ Info section แนวตั้ง
> - `Stack` → วาง Rating Badge ทับบนรูป
> - `Positioned` → กำหนดตำแหน่ง Badge ใน Stack
> - `Row` → จัดชื่อและราคาในบรรทัดเดียวกัน
> - `Expanded` → ทำให้ชื่อยืดและตัด ... เมื่อยาวเกิน
> - `Wrap` → จัด Tags โดย Wrap ขึ้นบรรทัดใหม่เองเมื่อไม่พอ

---

### การทดลองที่ 4 — สร้าง Screens

**เวลาโดยประมาณ:** 40 นาที

#### ขั้นตอนที่ 4.1 — Explore Screen (Responsive Grid Layout)

สร้างไฟล์ `lib/screens/explore_screen.dart`:

```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';
import '../models/destination.dart';
import '../widgets/destination_card.dart';

class ExploreScreen extends StatefulWidget {
  const ExploreScreen({super.key});

  @override
  State<ExploreScreen> createState() => _ExploreScreenState();
}

class _ExploreScreenState extends State<ExploreScreen> {
  String _searchQuery = '';

  List<Destination> get _filteredDestinations {
    if (_searchQuery.isEmpty) return sampleDestinations;
    return sampleDestinations
        .where(
          (d) =>
              d.name.toLowerCase().contains(_searchQuery.toLowerCase()) ||
              d.country.toLowerCase().contains(_searchQuery.toLowerCase()) ||
              d.tags.any(
                (t) => t.toLowerCase().contains(_searchQuery.toLowerCase()),
              ),
        )
        .toList();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('สำรวจ'),
        centerTitle: false,
      ),
      body: Column(
        children: [
          // ── Search Bar ──────────────────────────────────────────
          Padding(
            padding: const EdgeInsets.fromLTRB(16, 8, 16, 0),
            child: TextField(
              onChanged: (value) => setState(() => _searchQuery = value),
              decoration: InputDecoration(
                hintText: 'ค้นหา Destination...',
                prefixIcon: const Icon(Icons.search),
                border: OutlineInputBorder(
                  borderRadius: BorderRadius.circular(12),
                  borderSide: BorderSide.none,
                ),
                filled: true,
                fillColor: Colors.grey.shade100,
              ),
            ),
          ),
          const SizedBox(height: 12),
          // ── Grid หรือ Empty State ────────────────────────────────
          Expanded(
            child: _filteredDestinations.isEmpty
                ? _buildEmptyState()
                : _buildGrid(),
          ),
        ],
      ),
    );
  }

  Widget _buildGrid() {
    // ── LayoutBuilder: ปรับ Column Count ตามมาตรฐาน M3 Window Size Classes ──
    return LayoutBuilder(
      builder: (context, constraints) {
        int crossAxisCount;
        if (constraints.maxWidth < 600) {
          crossAxisCount = 2; // Compact: Phone
        } else if (constraints.maxWidth < 840) {
          crossAxisCount = 3; // Medium: Tablet Portrait
        } else {
          crossAxisCount = 4; // Expanded: Tablet Landscape / Desktop
        }

        return GridView.builder(
          padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
          gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
            crossAxisCount: crossAxisCount,
            crossAxisSpacing: 16,
            mainAxisSpacing: 16,
            childAspectRatio: 0.72, // สัดส่วน Card width/height
          ),
          itemCount: _filteredDestinations.length,
          itemBuilder: (context, index) {
            final destination = _filteredDestinations[index];
            return DestinationCard(
              destination: destination,
              onTap: () {
                // เรียกใช้ Named Route แบบมี Type-safe parameters
                context.pushNamed(
                  'destination-detail',
                  pathParameters: {'id': destination.id},
                  extra: destination,
                );
              },
            );
          },
        );
      },
    );
  }

  Widget _buildEmptyState() {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Icon(Icons.search_off, size: 64, color: Colors.grey.shade400),
          const SizedBox(height: 16),
          Text(
            'ไม่พบ Destination ที่ค้นหา',
            style: TextStyle(fontSize: 16, color: Colors.grey.shade600),
          ),
          const SizedBox(height: 8),
          Text(
            '"$_searchQuery"',
            style: const TextStyle(
                fontSize: 14,
                fontWeight: FontWeight.bold,
                color: Colors.grey),
          ),
        ],
      ),
    );
  }
}
```

#### ขั้นตอนที่ 4.2 — Destination Detail Screen

สร้างไฟล์ `lib/screens/destination_detail_screen.dart`:

```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';
import '../models/destination.dart';

class DestinationDetailScreen extends StatelessWidget {
  final Destination destination;

  const DestinationDetailScreen({
    super.key,
    required this.destination,
  });

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      // extendBodyBehindAppBar: ทำให้ Body ขยายใต้ AppBar (Hero Effect)
      extendBodyBehindAppBar: true,
      appBar: AppBar(
        backgroundColor: Colors.transparent,
        elevation: 0,
        leading: GestureDetector(
          onTap: () => context.pop(),
          child: Container(
            margin: const EdgeInsets.all(8),
            decoration: BoxDecoration(
              color: Colors.black.withValues(alpha: 0.45),
              shape: BoxShape.circle,
            ),
            child: const Icon(Icons.arrow_back, color: Colors.white),
          ),
        ),
        actions: [
          Container(
            margin: const EdgeInsets.all(8),
            decoration: BoxDecoration(
              color: Colors.black.withValues(alpha: 0.45),
              shape: BoxShape.circle,
            ),
            child: IconButton(
              icon: const Icon(Icons.favorite_border, color: Colors.white),
              onPressed: () {
                ScaffoldMessenger.of(context).showSnackBar(
                  SnackBar(
                      content: Text('บันทึก ${destination.name} แล้ว!'),
                      duration: const Duration(seconds: 2)),
                );
              },
            ),
          ),
        ],
      ),
      body: SingleChildScrollView(
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // ── Hero Image ──────────────────────────────────────────
            Stack(
              children: [
                // รูป Destination
                SizedBox(
                  height: 300,
                  width: double.infinity,
                  child: Image.network(
                    destination.imageUrl,
                    fit: BoxFit.cover,
                    errorBuilder: (ctx, err, _) => Container(
                      color: Colors.grey.shade300,
                      child: const Icon(Icons.image, size: 64),
                    ),
                  ),
                ),
                // Gradient Overlay สำหรับ Text ด้านล่างรูป
                Positioned(
                  bottom: 0,
                  left: 0,
                  right: 0,
                  child: Container(
                    height: 100,
                    decoration: const BoxDecoration(
                      gradient: LinearGradient(
                        begin: Alignment.bottomCenter,
                        end: Alignment.topCenter,
                        colors: [Colors.black87, Colors.transparent],
                      ),
                    ),
                  ),
                ),
                // ชื่อ Destination ทับบน Gradient
                Positioned(
                  bottom: 16,
                  left: 20,
                  right: 20,
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Text(
                        destination.name,
                        style: const TextStyle(
                          color: Colors.white,
                          fontSize: 28,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                      Row(
                        children: [
                          const Icon(Icons.location_on,
                              color: Colors.white70, size: 16),
                          const SizedBox(width: 4),
                          Text(
                            destination.country,
                            style: const TextStyle(
                                color: Colors.white70, fontSize: 14),
                          ),
                        ],
                      ),
                    ],
                  ),
                ),
              ],
            ),

            // ── Info Section ────────────────────────────────────────
            Padding(
              padding: const EdgeInsets.all(20),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  // Rating และ Price Row
                  Row(
                    children: [
                      // Rating
                      _InfoChip(
                        icon: Icons.star,
                        iconColor: Colors.amber,
                        label: '${destination.rating}',
                        subtitle: 'Rating',
                      ),
                      const SizedBox(width: 16),
                      // Price
                      _InfoChip(
                        icon: Icons.attach_money,
                        iconColor: Colors.green,
                        label: '\$${destination.price}',
                        subtitle: 'ต่อคืน',
                      ),
                    ],
                  ),
                  const SizedBox(height: 20),

                  // Description
                  const Text(
                    'เกี่ยวกับ',
                    style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
                  ),
                  const SizedBox(height: 8),
                  Text(
                    destination.description,
                    style: const TextStyle(
                        fontSize: 15, height: 1.6, color: Colors.black87),
                  ),
                  const SizedBox(height: 20),

                  // Tags
                  const Text(
                    'สิ่งที่น่าสนใจ',
                    style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
                  ),
                  const SizedBox(height: 10),
                  Wrap(
                    spacing: 10,
                    runSpacing: 8,
                    children: destination.tags
                        .map(
                          (tag) => Container(
                            padding: const EdgeInsets.symmetric(
                                horizontal: 16, vertical: 8),
                            decoration: BoxDecoration(
                              color: Colors.blue.shade50,
                              borderRadius: BorderRadius.circular(20),
                              border: Border.all(color: Colors.blue.shade200),
                            ),
                            child: Text(tag,
                                style: TextStyle(color: Colors.blue.shade700)),
                          ),
                        )
                        .toList(),
                  ),
                  const SizedBox(height: 32),

                  // CTA Button
                  SizedBox(
                    width: double.infinity,
                    height: 52,
                    child: ElevatedButton.icon(
                      onPressed: () {
                        showDialog(
                          context: context,
                          builder: (_) => AlertDialog(
                            title: const Text('จองสำเร็จ! 🎉'),
                            content: Text(
                                'คุณได้จอง ${destination.name} เรียบร้อยแล้ว'),
                            actions: [
                              TextButton(
                                onPressed: () {
                                  Navigator.pop(context);
                                  context.go('/');
                                },
                                child: const Text('กลับหน้าหลัก'),
                              ),
                            ],
                          ),
                        );
                      },
                      icon: const Icon(Icons.flight_takeoff),
                      label: const Text('จองเลย',
                          style: TextStyle(fontSize: 16)),
                      style: ElevatedButton.styleFrom(
                        shape: RoundedRectangleBorder(
                            borderRadius: BorderRadius.circular(12)),
                      ),
                    ),
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}

// Reusable Info Chip Widget
class _InfoChip extends StatelessWidget {
  final IconData icon;
  final Color iconColor;
  final String label;
  final String subtitle;

  const _InfoChip({
    required this.icon,
    required this.iconColor,
    required this.label,
    required this.subtitle,
  });

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 12),
      decoration: BoxDecoration(
        color: Colors.grey.shade100,
        borderRadius: BorderRadius.circular(12),
      ),
      child: Column(
        children: [
          Row(
            mainAxisSize: MainAxisSize.min,
            children: [
              Icon(icon, color: iconColor, size: 18),
              const SizedBox(width: 4),
              Text(label,
                  style: const TextStyle(
                      fontSize: 18, fontWeight: FontWeight.bold)),
            ],
          ),
          Text(subtitle,
              style: const TextStyle(fontSize: 12, color: Colors.grey)),
        ],
      ),
    );
  }
}
```

#### ขั้นตอนที่ 4.3 — Home, Saved, Profile Screens

สร้างไฟล์ `lib/screens/home_screen.dart`:

```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';
import '../models/destination.dart';
import '../widgets/destination_card.dart';

class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    final featured = sampleDestinations.take(3).toList();

    return Scaffold(
      body: SafeArea(
        child: SingleChildScrollView(
          padding: const EdgeInsets.symmetric(horizontal: 20, vertical: 16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              // ── Header ──────────────────────────────────────────
              Row(
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: [
                  Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Text(
                        'สวัสดี, นักเดินทาง! 👋',
                        style: TextStyle(
                            fontSize: 14, color: Colors.grey.shade600),
                      ),
                      const Text(
                        'ไปไหนดีวันนี้?',
                        style: TextStyle(
                            fontSize: 24, fontWeight: FontWeight.bold),
                      ),
                    ],
                  ),
                  CircleAvatar(
                    backgroundColor: Colors.blue.shade100,
                    child: const Icon(Icons.person, color: Colors.blue),
                  ),
                ],
              ),
              const SizedBox(height: 24),

              // ── Featured Section ────────────────────────────────
              Row(
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: [
                  const Text('แนะนำสำหรับคุณ',
                      style:
                          TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
                  TextButton(
                    onPressed: () => context.go('/explore'),
                    child: const Text('ดูทั้งหมด'),
                  ),
                ],
              ),
              const SizedBox(height: 12),

              // ListView แนวนอน
              SizedBox(
                height: 280,
                child: ListView.separated(
                  scrollDirection: Axis.horizontal,
                  itemCount: featured.length,
                  separatorBuilder: (_, __) => const SizedBox(width: 16),
                  itemBuilder: (context, index) {
                    final dest = featured[index];
                    return SizedBox(
                      width: 220,
                      child: DestinationCard(
                        destination: dest,
                        onTap: () => context.pushNamed(
                          'destination-detail',
                          pathParameters: {'id': dest.id},
                          extra: dest,
                        ),
                      ),
                    );
                  },
                ),
              ),

              const SizedBox(height: 24),

              // ── Quick Stats ─────────────────────────────────────
              const Text('สถิติการเดินทาง',
                  style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
              const SizedBox(height: 12),
              Row(
                children: [
                  Expanded(
                    child: _StatCard(
                        icon: Icons.flight,
                        label: 'Trip',
                        value: '5',
                        color: Colors.blue),
                  ),
                  const SizedBox(width: 12),
                  Expanded(
                    child: _StatCard(
                        icon: Icons.place,
                        label: 'Country',
                        value: '3',
                        color: Colors.orange),
                  ),
                  const SizedBox(width: 12),
                  Expanded(
                    child: _StatCard(
                        icon: Icons.favorite,
                        label: 'Saved',
                        value: '12',
                        color: Colors.pink),
                  ),
                ],
              ),
            ],
          ),
        ),
      ),
    );
  }
}

class _StatCard extends StatelessWidget {
  final IconData icon;
  final String label;
  final String value;
  final Color color;

  const _StatCard(
      {required this.icon,
      required this.label,
      required this.value,
      required this.color});

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: color.withValues(alpha: 0.1),
        borderRadius: BorderRadius.circular(12),
      ),
      child: Column(
        children: [
          Icon(icon, color: color, size: 28),
          const SizedBox(height: 8),
          Text(value,
              style: TextStyle(
                  fontSize: 20, fontWeight: FontWeight.bold, color: color)),
          Text(label, style: TextStyle(fontSize: 12, color: color)),
        ],
      ),
    );
  }
}
```

สร้างไฟล์ `lib/screens/saved_screen.dart`:

```dart
import 'package:flutter/material.dart';

class SavedScreen extends StatelessWidget {
  const SavedScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('บันทึกไว้')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(Icons.favorite, size: 64, color: Colors.pink.shade200),
            const SizedBox(height: 16),
            const Text('ยังไม่มีรายการที่บันทึก',
                style: TextStyle(fontSize: 16, color: Colors.grey)),
          ],
        ),
      ),
    );
  }
}
```

สร้างไฟล์ `lib/screens/profile_screen.dart`:

```dart
import 'package:flutter/material.dart';

class ProfileScreen extends StatelessWidget {
  const ProfileScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('โปรไฟล์')),
      body: ListView(
        children: [
          // Profile Header
          Container(
            color: Colors.blue.shade50,
            padding: const EdgeInsets.all(24),
            child: const Column(
              children: [
                CircleAvatar(
                  radius: 48,
                  backgroundColor: Colors.blue,
                  child: Icon(Icons.person, size: 48, color: Colors.white),
                ),
                SizedBox(height: 12),
                Text('นักศึกษา Flutter',
                    style:
                        TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
                Text('student@example.com',
                    style: TextStyle(color: Colors.grey)),
              ],
            ),
          ),
          // Settings List
          ListTile(
            leading: const Icon(Icons.notifications_outlined),
            title: const Text('การแจ้งเตือน'),
            trailing: const Icon(Icons.chevron_right),
            onTap: () {},
          ),
          ListTile(
            leading: const Icon(Icons.language_outlined),
            title: const Text('ภาษา'),
            trailing: const Icon(Icons.chevron_right),
            onTap: () {},
          ),
          ListTile(
            leading: const Icon(Icons.info_outline),
            title: const Text('เกี่ยวกับแอป'),
            trailing: const Icon(Icons.chevron_right),
            onTap: () {},
          ),
          const Divider(),
          ListTile(
            leading: const Icon(Icons.logout, color: Colors.red),
            title: const Text('ออกจากระบบ',
                style: TextStyle(color: Colors.red)),
            onTap: () {},
          ),
        ],
      ),
    );
  }
}
```

---

### การทดลองที่ 5 — ตั้งค่า Go Router

**เวลาโดยประมาณ:** 25 นาที

#### ขั้นตอนที่ 5.1 — สร้าง Router Configuration

สร้างไฟล์ `lib/router/app_router.dart`:

```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';
import '../models/destination.dart';
import '../screens/home_screen.dart';
import '../screens/explore_screen.dart';
import '../screens/destination_detail_screen.dart';
import '../screens/saved_screen.dart';
import '../screens/profile_screen.dart';

// ── Scaffold Shell Wrapper ─────────────────────────────────────────
class ScaffoldWithNavBar extends StatelessWidget {
  final StatefulNavigationShell navigationShell;

  const ScaffoldWithNavBar({
    super.key,
    required this.navigationShell,
  });

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: navigationShell, // แสดง Content ของ active branch
      bottomNavigationBar: NavigationBar(
        selectedIndex: navigationShell.currentIndex,
        onDestinationSelected: (index) {
          navigationShell.goBranch(
            index,
            initialLocation: index == navigationShell.currentIndex,
          );
        },
        destinations: const [
          NavigationDestination(
            icon: Icon(Icons.home_outlined),
            selectedIcon: Icon(Icons.home),
            label: 'หน้าหลัก',
          ),
          NavigationDestination(
            icon: Icon(Icons.explore_outlined),
            selectedIcon: Icon(Icons.explore),
            label: 'สำรวจ',
          ),
          NavigationDestination(
            icon: Icon(Icons.favorite_outline),
            selectedIcon: Icon(Icons.favorite),
            label: 'บันทึก',
          ),
          NavigationDestination(
            icon: Icon(Icons.person_outline),
            selectedIcon: Icon(Icons.person),
            label: 'โปรไฟล์',
          ),
        ],
      ),
    );
  }
}

// ── Router Definition ──────────────────────────────────────────────
final GoRouter appRouter = GoRouter(
  initialLocation: '/',
  debugLogDiagnostics: true,
  routes: [
    // StatefulShellRoute.indexedStack ช่วยรักษาสภาพ State ของแต่ละ Tab
    StatefulShellRoute.indexedStack(
      builder: (context, state, navigationShell) {
        return ScaffoldWithNavBar(navigationShell: navigationShell);
      },
      branches: [
        // ── Branch 0: Home ──────────────────────────────────────
        StatefulShellBranch(
          routes: [
            GoRoute(
              path: '/',
              name: 'home',
              builder: (context, state) => const HomeScreen(),
            ),
          ],
        ),
        // ── Branch 1: Explore + Detail ──────────────────────────
        StatefulShellBranch(
          routes: [
            GoRoute(
              path: '/explore',
              name: 'explore',
              builder: (context, state) => const ExploreScreen(),
              routes: [
                GoRoute(
                  path: 'destinations/:id', // Sub-route path ไม่ต้องมี / นำหน้า
                  name: 'destination-detail',
                  builder: (context, state) {
                    final id = state.pathParameters['id'];
                    // Fallback ดึงข้อมูลจาก ID กรณี extra เป็น null (เช่น กด Refresh บน Web)
                    final destination = state.extra as Destination? ??
                        sampleDestinations.firstWhere(
                          (d) => d.id == id,
                          orElse: () => sampleDestinations.first,
                        );
                    return DestinationDetailScreen(destination: destination);
                  },
                ),
              ],
            ),
          ],
        ),
        // ── Branch 2: Saved ─────────────────────────────────────
        StatefulShellBranch(
          routes: [
            GoRoute(
              path: '/saved',
              name: 'saved',
              builder: (context, state) => const SavedScreen(),
            ),
          ],
        ),
        // ── Branch 3: Profile ───────────────────────────────────
        StatefulShellBranch(
          routes: [
            GoRoute(
              path: '/profile',
              name: 'profile',
              builder: (context, state) => const ProfileScreen(),
            ),
          ],
        ),
      ],
    ),
  ],
  errorBuilder: (context, state) => Scaffold(
    body: Center(
      child: Text('ไม่พบหน้าที่ต้องการ: ${state.error}'),
    ),
  ),
);
```

#### ขั้นตอนที่ 5.2 — ตั้งค่า main.dart

แก้ไขไฟล์ `lib/main.dart`:

```dart
import 'package:flutter/material.dart';
import 'router/app_router.dart';

void main() {
  runApp(const TravelApp());
}

class TravelApp extends StatelessWidget {
  const TravelApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'Travel App',
      debugShowCheckedModeBanner: false,
      // ── Theme Configuration ──────────────────────────────────
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(
          seedColor: Colors.blue,
          brightness: Brightness.light,
        ),
        useMaterial3: true,
        // AppBar theme
        appBarTheme: const AppBarTheme(
          centerTitle: false,
          elevation: 0,
          scrolledUnderElevation: 1,
        ),
      ),
      // ── Router Config ────────────────────────────────────────
      routerConfig: appRouter,
    );
  }
}
```

---

### การทดลองที่ 6 — รันและทดสอบ

**เวลาโดยประมาณ:** 15 นาที

#### ขั้นตอนที่ 6.1 — ตรวจสอบว่า Build ผ่าน

```bash
flutter analyze
# ควรไม่มี Error หลัก
```

#### ขั้นตอนที่ 6.2 — รันบน Emulator หรือ Device

```bash
flutter run
```

#### ขั้นตอนที่ 6.3 — ทดสอบฟีเจอร์ต่าง ๆ

ทดสอบตามรายการนี้และจด ✅ / ❌:

| # | สิ่งที่ทดสอบ | ผลที่คาดหวัง | ผลจริง |
|---|---|---|---|
| 1 | เปิดแอป | เห็น Home Screen + Bottom Navigation Bar | |
| 2 | กด Tab "สำรวจ" | เปลี่ยนไป Explore Screen แสดง Grid | |
| 3 | พิมพ์ค้นหา "โตเกียว" | ผลการค้นหาเหลือเฉพาะโตเกียว | |
| 4 | กดที่ Card ใด ๆ | เปิด Detail Screen พร้อมข้อมูลถูกต้อง | |
| 5 | กด Back บน Detail | กลับมา Explore Screen | |
| 6 | กด Tab "หน้าหลัก" | กลับหน้าหลัก โดยที่ Stack ใน Explore ยังไม่หาย | |
| 7 | กดหัวใจบน Detail | Snackbar แจ้งบันทึกสำเร็จ | |
| 8 | กด "จองเลย" บน Detail | Dialog แสดงการจองสำเร็จ | |
| 9 | กด "กลับหน้าหลัก" ใน Dialog | Navigate กลับ Home | |
| 10 | หมุนหน้าจอ Landscape | Grid ปรับ Column Count ตาม M3 Breakpoint | |

---

### การทดลองที่ 7 — ทดลองเพิ่มเติม (ถ้ามีเวลา)

**เวลาโดยประมาณ:** 20 นาที

#### ขั้นตอนที่ 7.1 — เพิ่ม Category Filter

เพิ่ม Filter Chip แนวนอนใน ExploreScreen ระหว่าง Search Bar กับ Grid:

```dart
// เพิ่มใน State ของ _ExploreScreenState
String _selectedTag = 'ทั้งหมด';
final List<String> _allTags = ['ทั้งหมด', 'ทะเล', 'ธรรมชาติ', 'วัฒนธรรม', 'อาหาร', 'ช้อปปิ้ง'];

// Widget Filter:
SizedBox(
  height: 40,
  child: ListView.separated(
    scrollDirection: Axis.horizontal,
    padding: const EdgeInsets.symmetric(horizontal: 16),
    itemCount: _allTags.length,
    separatorBuilder: (_, __) => const SizedBox(width: 8),
    itemBuilder: (context, index) {
      final tag = _allTags[index];
      final isSelected = tag == _selectedTag;
      return FilterChip(
        label: Text(tag),
        selected: isSelected,
        onSelected: (_) => setState(() => _selectedTag = tag),
      );
    },
  ),
),
```

#### ขั้นตอนที่ 7.2 — เพิ่ม Transition Animation

แก้ใน GoRoute ของ Detail Screen เพิ่ม `pageBuilder`:

```dart
GoRoute(
  path: 'destinations/:id',
  name: 'destination-detail',
  pageBuilder: (context, state) {
    final id = state.pathParameters['id'];
    final destination = state.extra as Destination? ??
        sampleDestinations.firstWhere((d) => d.id == id);

    return CustomTransitionPage(
      child: DestinationDetailScreen(destination: destination),
      transitionsBuilder: (context, animation, secondaryAnimation, child) {
        return SlideTransition(
          position: Tween<Offset>(
            begin: const Offset(1.0, 0.0),
            end: Offset.zero,
          ).animate(CurvedAnimation(
            parent: animation,
            curve: Curves.easeInOut,
          )),
          child: child,
        );
      },
    );
  },
),
```

---

## 📝 คำถามท้ายใบงาน

**ให้ตอบคำถามต่อไปนี้ในรายงาน:**

1. `LayoutBuilder` ต่างกับ `MediaQuery` อย่างไร? เลือกใช้อันไหนในสถานการณ์ใด?

2. ทำไม Go Router ถึงใช้ `StatefulShellRoute` แทน `ShellRoute` ธรรมดา? ผลต่างเรื่อง State Management คืออะไร?

3. ในโค้ด `DestinationCard` เราใช้ `Expanded` ครอบ `Text` ชื่อ Destination ทำไม? จะเกิดอะไรขึ้นถ้าลบออก?

4. การส่งข้อมูลผ่าน `extra` ของ Go Router มีข้อจำกัดอะไรกรณี Deep Link / Web Refresh? และแก้ปัญหานี้ได้อย่างไร?

5. วาด Navigation Hierarchy ของแอปนี้ (สามารถวาดบนกระดาษแล้วถ่ายรูป)

---

## 📤 การส่งงาน

1. Push โค้ดขึ้น GitHub Repository ส่วนตัว (Branch: `week04-layout-navigation`)
2. สร้าง Pull Request พร้อมเขียน Description ว่าทำอะไรไปบ้าง
3. แนบ Screenshot หรือ Screen Recording แสดง Navigation ที่ทำงานได้
4. ตอบคำถามท้ายใบงานใน Comment ของ Pull Request

**Deadline:** ก่อนชั้นเรียนสัปดาห์ที่ 5

---

## 🔗 เอกสารอ้างอิง

- [Go Router Documentation](https://pub.dev/packages/go_router)
- [Flutter Layout Guide](https://docs.flutter.dev/ui/layout)
- [Material Design 3 — Navigation](https://m3.material.io/components/navigation-bar/overview)
- [Flutter Cookbook — Navigation](https://docs.flutter.dev/cookbook/navigation)
