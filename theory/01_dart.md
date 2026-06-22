# 01 · Dart — Ngôn Ngữ Nền Tảng

> Dart là ngôn ngữ lập trình kiểu tĩnh, biên dịch native, được Google phát triển. Đây là ngôn ngữ duy nhất chạy được trên Flutter.

---

## 1. Kiểu Dữ Liệu Cơ Bản

```dart
// Số nguyên và số thực
int age = 25;
double height = 1.75;
num score = 9.5;        // num = int hoặc double

// Chuỗi
String name = 'Khoa';
String greeting = "Xin chào $name!";                // string interpolation
String multiLine = '''
  Đây là chuỗi
  nhiều dòng
''';

// Boolean
bool isStudent = true;

// Null Safety (Dart ≥ 2.12)
String? nickname;         // có thể null
String username = 'khoa'; // không thể null — bắt buộc có giá trị
```

---

## 2. Collections

```dart
// List (mảng có thứ tự)
List<String> fruits = ['táo', 'cam', 'xoài'];
fruits.add('chuối');
fruits.removeAt(0);
print(fruits.length);       // 3
print(fruits[0]);           // cam

// Spread operator
List<int> a = [1, 2, 3];
List<int> b = [0, ...a, 4]; // [0, 1, 2, 3, 4]

// Set (tập hợp không trùng lặp)
Set<String> colors = {'đỏ', 'xanh', 'vàng'};
colors.add('đỏ'); // không thêm vì đã tồn tại

// Map (key-value)
Map<String, int> scores = {
  'Toán': 9,
  'Lý': 8,
  'Hóa': 7,
};
scores['Văn'] = 10;
print(scores['Toán']); // 9
```

---

## 3. Null Safety

Null Safety là tính năng quan trọng nhất của Dart hiện đại — giúp tránh lỗi NullPointerException.

```dart
// ? — kiểu nullable
String? email;
print(email?.length);       // null (không crash)
print(email ?? 'chưa có');  // 'chưa có'

// ! — khẳng định không null (dùng cẩn thận!)
String? data = fetchData();
print(data!.toUpperCase());  // crash nếu data là null

// late — khai báo trễ, đảm bảo sẽ có giá trị trước khi dùng
late String initData;
void init() {
  initData = 'đã khởi tạo';
}

// Null-aware assignment
String? config;
config ??= 'default'; // chỉ gán nếu config đang null

// Conditional member access
User? user;
String? city = user?.address?.city; // an toàn, không crash
```

---

## 4. Hàm (Functions)

```dart
// Hàm thông thường
int add(int a, int b) {
  return a + b;
}

// Arrow function (một biểu thức)
int multiply(int a, int b) => a * b;

// Named parameters (tham số đặt tên)
void greet({required String name, String title = 'bạn'}) {
  print('Xin chào $title $name!');
}
greet(name: 'Khoa');             // Xin chào bạn Khoa!
greet(name: 'Nam', title: 'anh'); // Xin chào anh Nam!

// Optional positional parameters
String format(String text, [String? prefix]) {
  return prefix != null ? '$prefix $text' : text;
}

// Higher-order function
List<int> numbers = [1, 2, 3, 4, 5];
List<int> evens = numbers.where((n) => n.isEven).toList(); // [2, 4]
List<int> doubled = numbers.map((n) => n * 2).toList();    // [2, 4, 6, 8, 10]
int total = numbers.reduce((a, b) => a + b);               // 15
```

---

## 5. Lập Trình Hướng Đối Tượng (OOP)

```dart
// Class cơ bản
class Animal {
  final String name;
  int _age;              // private (prefix _)

  Animal(this.name, this._age);  // constructor ngắn gọn

  // Named constructor
  Animal.unknown() : name = 'unknown', _age = 0;

  // Getter
  int get age => _age;

  // Setter
  set age(int value) {
    if (value >= 0) _age = value;
  }

  void speak() => print('$name phát ra âm thanh');

  @override
  String toString() => 'Animal($name, $_age tuổi)';
}

// Kế thừa
class Dog extends Animal {
  final String breed;

  Dog(super.name, super.age, this.breed);

  @override
  void speak() => print('$name: Gâu gâu!');
}

// Abstract class
abstract class Shape {
  double get area;
  double get perimeter;
}

class Circle extends Shape {
  final double radius;
  Circle(this.radius);

  @override
  double get area => 3.14 * radius * radius;

  @override
  double get perimeter => 2 * 3.14 * radius;
}

// Mixin — tái sử dụng code mà không cần kế thừa
mixin Flyable {
  void fly() => print('Đang bay!');
}

mixin Swimmable {
  void swim() => print('Đang bơi!');
}

class Duck extends Animal with Flyable, Swimmable {
  Duck(super.name, super.age);
}

// Interface (implicit — mọi class đều là interface)
class Printer {
  void print(String text) {}
}

class ConsolePrinter implements Printer {
  @override
  void print(String text) => print(text);
}
```

---

## 6. Lập Trình Bất Đồng Bộ (Async/Await)

```dart
import 'dart:async';

// Future — đại diện cho giá trị sẽ có trong tương lai
Future<String> fetchUserName(int id) async {
  await Future.delayed(Duration(seconds: 1)); // giả lập delay
  return 'User #$id';
}

// Sử dụng async/await
void loadUser() async {
  try {
    String name = await fetchUserName(1);
    print('Tên: $name');
  } catch (e) {
    print('Lỗi: $e');
  }
}

// Chạy nhiều Future song song
Future<void> loadAll() async {
  final results = await Future.wait([
    fetchUserName(1),
    fetchUserName(2),
    fetchUserName(3),
  ]);
  print(results); // [User #1, User #2, User #3]
}

// Stream — luồng dữ liệu liên tục
Stream<int> countdown(int from) async* {
  for (int i = from; i >= 0; i--) {
    await Future.delayed(Duration(seconds: 1));
    yield i;
  }
}

void listenToCountdown() async {
  await for (int count in countdown(5)) {
    print(count);
  }
}

// StreamController
StreamController<String> controller = StreamController<String>();
Stream<String> stream = controller.stream;

stream.listen(
  (data) => print('Nhận: $data'),
  onError: (e) => print('Lỗi: $e'),
  onDone: () => print('Hoàn thành'),
);

controller.add('Hello');
controller.add('World');
controller.close();
```

---

## 7. Generics

```dart
// Generic class
class Stack<T> {
  final List<T> _items = [];

  void push(T item) => _items.add(item);

  T pop() {
    if (_items.isEmpty) throw StateError('Stack rỗng');
    return _items.removeLast();
  }

  bool get isEmpty => _items.isEmpty;
  int get size => _items.length;
}

// Sử dụng
final intStack = Stack<int>();
intStack.push(1);
intStack.push(2);
print(intStack.pop()); // 2

// Generic function
T findFirst<T>(List<T> list, bool Function(T) predicate) {
  return list.firstWhere(predicate);
}

int firstEven = findFirst([1, 3, 4, 6], (n) => n.isEven); // 4
```

---

## 8. Extension Methods

Thêm method vào class có sẵn mà không cần sửa source code.

```dart
extension StringExtension on String {
  bool get isValidEmail => contains('@') && contains('.');
  String get capitalize => '${this[0].toUpperCase()}${substring(1)}';
  String truncate(int maxLength) =>
      length > maxLength ? '${substring(0, maxLength)}...' : this;
}

extension ListExtension<T> on List<T> {
  T? get secondOrNull => length >= 2 ? this[1] : null;
}

// Sử dụng
print('khoa@email.com'.isValidEmail);  // true
print('hello'.capitalize);             // Hello
print('một chuỗi dài'.truncate(5));    // một c...

List<int> nums = [10, 20, 30];
print(nums.secondOrNull); // 20
```

---

## 9. Pattern Matching & Records (Dart 3.0+)

```dart
// Records — tuple có tên
(String, int) getUser() => ('Khoa', 22);

var (name, age) = getUser();
print('$name - $age tuổi');

({String name, String city}) getProfile() =>
    (name: 'Khoa', city: 'Hà Nội');

var profile = getProfile();
print(profile.name); // Khoa

// Switch expression
String grade(int score) => switch (score) {
  >= 90 => 'A',
  >= 80 => 'B',
  >= 70 => 'C',
  >= 60 => 'D',
  _ => 'F',
};

// Pattern matching với sealed class
sealed class Shape {}
class Circle extends Shape { final double radius; Circle(this.radius); }
class Rectangle extends Shape { final double w, h; Rectangle(this.w, this.h); }

double area(Shape shape) => switch (shape) {
  Circle(:final radius) => 3.14 * radius * radius,
  Rectangle(:final w, :final h) => w * h,
};
```

---

## 10. Một Số Tip Thực Tế

```dart
// Cascade notation (..)
final user = User()
  ..name = 'Khoa'
  ..age = 22
  ..email = 'khoa@email.com';

// Spread trong constructor
final merged = {...map1, ...map2};

// Collection if/for
final items = [
  'always',
  if (isLoggedIn) 'profile',
  for (var item in extras) item,
];

// typedef — đặt tên cho kiểu phức tạp
typedef Callback<T> = void Function(T value);
typedef Json = Map<String, dynamic>;

void doSomething(Callback<String> onDone) {
  onDone('xong rồi!');
}

// Const constructor — compile-time constant
class Config {
  final String baseUrl;
  const Config(this.baseUrl);
}

const config = Config('https://api.example.com'); // không tạo object mới ở runtime
```

---

## Bài Tập Thực Hành

**Bài 1 — Null Safety:** Viết hàm `parseAge(String? input)` trả về `int?`, trả về null nếu input không hợp lệ.

**Bài 2 — OOP:** Tạo class `BankAccount` với các method `deposit`, `withdraw`, `getBalance`. Đảm bảo số dư không âm.

**Bài 3 — Async:** Viết hàm giả lập gọi API trả về `Future<List<String>>`, xử lý cả trường hợp lỗi.

**Bài 4 — Generic:** Tạo generic class `Pair<A, B>` lưu hai giá trị khác kiểu, có method `swap()` trả về `Pair<B, A>`.

---

*Tiếp theo: [02 · Flutter](02_flutter.md) — Xây dựng giao diện với Widget Tree*
