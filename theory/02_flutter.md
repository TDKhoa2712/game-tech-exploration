# 02 · Flutter — Framework UI Đa Nền Tảng

> Flutter là framework UI của Google cho phép build app iOS, Android, Web, Desktop từ một codebase Dart duy nhất. Mọi thứ trong Flutter đều là **Widget**.

---

## 1. Widget Tree — Khái Niệm Cốt Lõi

Flutter render UI bằng cách xây dựng một cây Widget. Mỗi widget mô tả một phần của giao diện.

```
MaterialApp
└── Scaffold
    ├── AppBar
    │   └── Text("Tiêu đề")
    └── Column
        ├── Text("Hello")
        └── ElevatedButton
            └── Text("Bấm vào")
```

### StatelessWidget vs StatefulWidget

```dart
// StatelessWidget — không có state thay đổi
class Greeting extends StatelessWidget {
  final String name;
  const Greeting({super.key, required this.name});

  @override
  Widget build(BuildContext context) {
    return Text('Xin chào $name!');
  }
}

// StatefulWidget — có state thay đổi theo thời gian
class Counter extends StatefulWidget {
  const Counter({super.key});

  @override
  State<Counter> createState() => _CounterState();
}

class _CounterState extends State<Counter> {
  int _count = 0;

  void _increment() {
    setState(() {
      _count++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Đếm: $_count', style: const TextStyle(fontSize: 24)),
        ElevatedButton(
          onPressed: _increment,
          child: const Text('Tăng'),
        ),
      ],
    );
  }
}
```

---

## 2. Layout Widgets

### Column & Row

```dart
// Column — xếp dọc
Column(
  mainAxisAlignment: MainAxisAlignment.center,   // căn giữa theo trục dọc
  crossAxisAlignment: CrossAxisAlignment.start,  // căn trái theo trục ngang
  children: [
    Text('Item 1'),
    Text('Item 2'),
    Text('Item 3'),
  ],
)

// Row — xếp ngang
Row(
  mainAxisAlignment: MainAxisAlignment.spaceBetween,
  children: [
    Icon(Icons.home),
    Text('Trang chủ'),
    Icon(Icons.arrow_forward),
  ],
)
```

### Container & SizedBox

```dart
// Container — widget đa năng nhất
Container(
  width: 200,
  height: 100,
  margin: const EdgeInsets.all(16),
  padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 8),
  decoration: BoxDecoration(
    color: Colors.blue.shade100,
    borderRadius: BorderRadius.circular(12),
    border: Border.all(color: Colors.blue, width: 1.5),
    boxShadow: [
      BoxShadow(color: Colors.black26, blurRadius: 8, offset: Offset(2, 4)),
    ],
  ),
  child: Text('Nội dung'),
)

// SizedBox — tạo khoảng cách hoặc ép kích thước
const SizedBox(height: 16)          // khoảng cách dọc
const SizedBox(width: 8)            // khoảng cách ngang
SizedBox.expand(child: Container()) // chiếm toàn bộ không gian
```

### Stack & Positioned

```dart
// Stack — chồng widget lên nhau
Stack(
  children: [
    // Widget nền
    Container(width: 300, height: 200, color: Colors.grey),
    // Widget phủ lên, định vị tuyệt đối
    Positioned(
      right: 12,
      top: 12,
      child: Icon(Icons.favorite, color: Colors.red),
    ),
    // Căn giữa
    Center(child: Text('Chồng lên nhau')),
  ],
)
```

### Flexible & Expanded

```dart
Row(
  children: [
    // Chiếm 1 phần
    Expanded(
      flex: 1,
      child: Container(color: Colors.red),
    ),
    // Chiếm 2 phần (gấp đôi)
    Expanded(
      flex: 2,
      child: Container(color: Colors.blue),
    ),
    // Kích thước tự nhiên, không co giãn
    Flexible(
      child: Text('Text tự nhiên'),
    ),
  ],
)
```

---

## 3. Danh Sách (Lists)

```dart
// ListView — danh sách cuộn được
ListView(
  children: [
    ListTile(
      leading: const Icon(Icons.person),
      title: const Text('Nguyễn Văn A'),
      subtitle: const Text('khoa@email.com'),
      trailing: const Icon(Icons.arrow_forward_ios, size: 16),
      onTap: () {},
    ),
  ],
)

// ListView.builder — hiệu quả cho danh sách dài
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) {
    return Card(
      margin: const EdgeInsets.symmetric(horizontal: 16, vertical: 4),
      child: ListTile(
        title: Text(items[index].title),
        subtitle: Text(items[index].desc),
      ),
    );
  },
)

// GridView.builder — dạng lưới
GridView.builder(
  gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
    crossAxisCount: 2,
    crossAxisSpacing: 12,
    mainAxisSpacing: 12,
    childAspectRatio: 0.8,
  ),
  itemCount: products.length,
  itemBuilder: (context, index) => ProductCard(product: products[index]),
)
```

---

## 4. Input & Form

```dart
class LoginForm extends StatefulWidget {
  @override
  State<LoginForm> createState() => _LoginFormState();
}

class _LoginFormState extends State<LoginForm> {
  final _formKey = GlobalKey<FormState>();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  bool _obscurePassword = true;

  @override
  void dispose() {
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }

  void _submit() {
    if (_formKey.currentState!.validate()) {
      // Form hợp lệ — xử lý đăng nhập
    }
  }

  @override
  Widget build(BuildContext context) {
    return Form(
      key: _formKey,
      child: Column(
        children: [
          TextFormField(
            controller: _emailController,
            keyboardType: TextInputType.emailAddress,
            decoration: const InputDecoration(
              labelText: 'Email',
              prefixIcon: Icon(Icons.email_outlined),
              border: OutlineInputBorder(),
            ),
            validator: (value) {
              if (value == null || value.isEmpty) return 'Nhập email';
              if (!value.contains('@')) return 'Email không hợp lệ';
              return null;
            },
          ),
          const SizedBox(height: 16),
          TextFormField(
            controller: _passwordController,
            obscureText: _obscurePassword,
            decoration: InputDecoration(
              labelText: 'Mật khẩu',
              prefixIcon: const Icon(Icons.lock_outline),
              border: const OutlineInputBorder(),
              suffixIcon: IconButton(
                icon: Icon(_obscurePassword ? Icons.visibility : Icons.visibility_off),
                onPressed: () => setState(() => _obscurePassword = !_obscurePassword),
              ),
            ),
            validator: (value) {
              if (value == null || value.length < 6) return 'Tối thiểu 6 ký tự';
              return null;
            },
          ),
          const SizedBox(height: 24),
          SizedBox(
            width: double.infinity,
            child: ElevatedButton(
              onPressed: _submit,
              child: const Text('Đăng nhập'),
            ),
          ),
        ],
      ),
    );
  }
}
```

---

## 5. Navigation

```dart
// Push (đẩy màn hình mới)
Navigator.push(
  context,
  MaterialPageRoute(builder: (_) => const DetailScreen()),
);

// Push với animation tùy chỉnh
Navigator.push(
  context,
  PageRouteBuilder(
    pageBuilder: (_, __, ___) => const DetailScreen(),
    transitionsBuilder: (_, animation, __, child) {
      return SlideTransition(
        position: Tween(begin: const Offset(1, 0), end: Offset.zero)
          .animate(animation),
        child: child,
      );
    },
  ),
);

// Pop về màn hình trước
Navigator.pop(context);

// Pop với kết quả trả về
Navigator.pop(context, 'kết quả');

// Chờ kết quả từ màn hình khác
final result = await Navigator.push<String>(
  context,
  MaterialPageRoute(builder: (_) => const PickerScreen()),
);
if (result != null) print('Đã chọn: $result');

// Named routes
Navigator.pushNamed(context, '/detail', arguments: {'id': 1});
```

---

## 6. Theme — Light & Dark Mode

```dart
// main.dart
MaterialApp(
  theme: ThemeData(
    colorScheme: ColorScheme.fromSeed(
      seedColor: Colors.indigo,
      brightness: Brightness.light,
    ),
    useMaterial3: true,
    textTheme: const TextTheme(
      headlineLarge: TextStyle(fontSize: 32, fontWeight: FontWeight.bold),
      bodyMedium: TextStyle(fontSize: 16, height: 1.5),
    ),
    elevatedButtonTheme: ElevatedButtonThemeData(
      style: ElevatedButton.styleFrom(
        minimumSize: const Size(double.infinity, 48),
        shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
      ),
    ),
  ),
  darkTheme: ThemeData(
    colorScheme: ColorScheme.fromSeed(
      seedColor: Colors.indigo,
      brightness: Brightness.dark,
    ),
    useMaterial3: true,
  ),
  themeMode: ThemeMode.system, // theo hệ thống
)

// Truy cập theme trong widget
Widget build(BuildContext context) {
  final colorScheme = Theme.of(context).colorScheme;
  final textTheme = Theme.of(context).textTheme;

  return Container(
    color: colorScheme.primaryContainer,
    child: Text(
      'Tiêu đề',
      style: textTheme.headlineLarge?.copyWith(
        color: colorScheme.onPrimaryContainer,
      ),
    ),
  );
}
```

---

## 7. Animations

```dart
// AnimatedContainer — tự động animate khi property thay đổi
bool _expanded = false;

AnimatedContainer(
  duration: const Duration(milliseconds: 300),
  curve: Curves.easeInOut,
  width: _expanded ? 200 : 100,
  height: _expanded ? 200 : 100,
  color: _expanded ? Colors.blue : Colors.red,
  onTap: () => setState(() => _expanded = !_expanded),
)

// AnimatedOpacity
AnimatedOpacity(
  duration: const Duration(milliseconds: 500),
  opacity: _visible ? 1.0 : 0.0,
  child: Text('Ẩn/hiện'),
)

// AnimationController — kiểm soát thủ công
class _FadeWidgetState extends State<FadeWidget>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(milliseconds: 800),
      vsync: this,
    );
    _animation = CurvedAnimation(
      parent: _controller,
      curve: Curves.elasticOut,
    );
    _controller.forward();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return ScaleTransition(
      scale: _animation,
      child: const FlutterLogo(size: 100),
    );
  }
}
```

---

## 8. FutureBuilder & StreamBuilder

```dart
// FutureBuilder — hiển thị dữ liệu bất đồng bộ
FutureBuilder<List<User>>(
  future: fetchUsers(),
  builder: (context, snapshot) {
    if (snapshot.connectionState == ConnectionState.waiting) {
      return const Center(child: CircularProgressIndicator());
    }
    if (snapshot.hasError) {
      return Center(child: Text('Lỗi: ${snapshot.error}'));
    }
    final users = snapshot.data!;
    return ListView.builder(
      itemCount: users.length,
      itemBuilder: (_, i) => UserTile(user: users[i]),
    );
  },
)

// StreamBuilder — lắng nghe stream liên tục
StreamBuilder<int>(
  stream: timerStream(),
  initialData: 0,
  builder: (context, snapshot) {
    return Text('Đã qua: ${snapshot.data} giây');
  },
)
```

---

## 9. Custom Painter

Vẽ đồ họa tùy chỉnh với Canvas API.

```dart
class WavePainter extends CustomPainter {
  final Color color;
  final double progress; // 0.0 → 1.0

  WavePainter({required this.color, required this.progress});

  @override
  void paint(Canvas canvas, Size size) {
    final paint = Paint()
      ..color = color
      ..style = PaintingStyle.fill;

    final path = Path();
    path.moveTo(0, size.height * 0.5);

    for (double x = 0; x <= size.width; x++) {
      final y = size.height * 0.5 +
          20 * sin((x / size.width * 2 * pi) + progress * 2 * pi);
      path.lineTo(x, y);
    }

    path.lineTo(size.width, size.height);
    path.lineTo(0, size.height);
    path.close();

    canvas.drawPath(path, paint);
  }

  @override
  bool shouldRepaint(WavePainter old) =>
      old.progress != progress || old.color != color;
}

// Sử dụng
CustomPaint(
  size: const Size(double.infinity, 200),
  painter: WavePainter(color: Colors.blue.withOpacity(0.6), progress: _waveProgress),
)
```

---

## 10. BuildContext & InheritedWidget

```dart
// MediaQuery — thông tin màn hình
final size = MediaQuery.of(context).size;
final padding = MediaQuery.of(context).padding; // safe area
print('Chiều rộng: ${size.width}, Chiều cao: ${size.height}');

// Responsive layout
Widget build(BuildContext context) {
  final width = MediaQuery.of(context).size.width;
  final isTablet = width > 600;

  return isTablet
    ? Row(children: [SideBar(), Expanded(child: Content())])
    : Column(children: [Content(), BottomNav()]);
}

// SafeArea — tránh notch, status bar
SafeArea(
  child: Scaffold(
    body: YourContent(),
  ),
)
```

---

## Bài Tập Thực Hành

**Bài 1 — Layout:** Tạo màn hình Profile Card hiển thị ảnh đại diện, tên, email và 3 nút action dùng Stack + Positioned.

**Bài 2 — Form:** Tạo form đăng ký với các trường: họ tên, email, mật khẩu, xác nhận mật khẩu. Validate đầy đủ.

**Bài 3 — Animation:** Tạo màn hình loading với logo Flutter scale từ 0 → 1 khi khởi động app, dùng AnimationController.

**Bài 4 — List:** Build màn hình danh sách sản phẩm dùng GridView.builder, khi bấm vào item navigate đến màn hình chi tiết.

---

*Tiếp theo: [03 · Flame](03_flame.md) — Game Engine trên Flutter*
