# Flutter Products App

แอปพลิเคชัน Flutter สำหรับแสดงรายการสินค้าจาก DummyJSON API ด้วยการออกแบบ UI แบบ minimal และสวยงาม

## หลักการทำงาน

แอปนี้ใช้ **FutureBuilder** เป็นหัวใจหลักในการจัดการ asynchronous operations สำหรับดึงข้อมูลจาก API และแสดงผลอย่างมีประสิทธิภาพ

### การจัดการสถานะ (State Management)

แอปจัดการ 4 สถานะหลัก:
1. **Loading State** - แสดง CircularProgressIndicator ขณะรอข้อมูล
2. **Error State** - แสดงข้อความและไอคอน error เมื่อเกิดข้อผิดพลาด
3. **Empty State** - แสดงข้อความเมื่อไม่มีข้อมูลสินค้า
4. **Success State** - แสดงรายการสินค้าในรูปแบบ ListView

## อธิบายการทำงานของโค้ด

### 1. Product Model
```dart
class Product {
  final int id;
  final String title;
  final String description;
  final double price;
  final double rating;
  final String thumbnail;
  final String brand;
  final int stock;

  factory Product.fromJson(Map<String, dynamic> json) {
    return Product(
      id: json['id'],
      title: json['title'],
      description: json['description'],
      price: json['price'].toDouble(),
      rating: json['rating'].toDouble(),
      thumbnail: json['thumbnail'],
      brand: json['brand'] ?? 'Unknown',
      stock: json['stock'],
    );
  }
}
```
- สร้างโมเดลข้อมูลสินค้าเพื่อจัดเก็บข้อมูลจาก API
- ใช้ `factory Product.fromJson()` แปลงข้อมูล JSON response เป็น Product object
- จัดการกรณี `brand` เป็น null ด้วย null operator `??`

### 2. ProductsService - การเรียกใช้ API
```dart
class ProductsService {
  static const String apiUrl = 'https://dummyjson.com/products';

  static Future<List<Product>> fetchProducts() async {
    final response = await http.get(Uri.parse(apiUrl));

    if (response.statusCode == 200) {
      final data = json.decode(response.body);
      final List<dynamic> productsJson = data['products'];
      return productsJson.map((json) => Product.fromJson(json)).toList();
    } else {
      throw Exception('Failed to load products');
    }
  }
}
```

**ขั้นตอนการเรียก API:**
1. **HTTP GET Request** - ใช้ `http.get()` เรียก DummyJSON API
2. **Status Code Check** - ตรวจสอบ response status (200 = สำเร็จ)
3. **JSON Parsing** - แปลง response body จาก String เป็น Map
4. **Data Extraction** - ดึงข้อมูล products array จาก response
5. **Object Mapping** - แปลงแต่ละ item เป็น Product object
6. **Error Handling** - throw Exception เมื่อ status code ไม่เป็น 200

### 3. FutureBuilder Implementation
```dart
FutureBuilder<List<Product>>(
  future: ProductsService.fetchProducts(),
  builder: (context, snapshot) {
    if (snapshot.connectionState == ConnectionState.waiting) {
      return CircularProgressIndicator(); // Loading state
    }
    
    if (snapshot.hasError) {
      return ErrorWidget(); // Error state
    }
    
    if (!snapshot.hasData || snapshot.data!.isEmpty) {
      return EmptyWidget(); // Empty state
    }
    
    return ListView.builder(); // Success state
  },
)
```

**การทำงานของ FutureBuilder:**
- **future**: กำหนด async function ที่จะเรียกใช้
- **builder**: function ที่สร้าง widget ตาม snapshot state
- **ConnectionState.waiting**: สถานะกำลังรอผลลัพธ์
- **snapshot.hasError**: ตรวจสอบว่าเกิด error หรือไม่
- **snapshot.hasData**: ตรวจสอบว่ามีข้อมูลหรือไม่
- **snapshot.data**: เข้าถึงข้อมูลที่ได้จาก Future

### 4. ProductCard Widget
```dart
class ProductCard extends StatelessWidget {
  final Product product;

  @override
  Widget build(BuildContext context) {
    return Container(
      // การออกแบบการ์ด
      child: Row(
        children: [
          // รูปภาพสินค้า
          Image.network(product.thumbnail),
          // ข้อมูลสินค้า
          Expanded(child: ProductInfo()),
        ],
      ),
    );
  }
}
```
- แสดงข้อมูลสินค้าในรูปแบบการ์ด horizontal layout
- ใช้ `Row` จัดเรียง: รูปภาพ + ข้อมูลสินค้า
- รองรับ Image loading states และ error handling
- แสดงข้อมูล: ชื่อ, แบรนด์, คำอธิบาย, ราคา, rating, stock status

### 5. การจัดการรูปภาพ
```dart
Image.network(
  product.thumbnail,
  fit: BoxFit.cover,
  loadingBuilder: (context, child, loadingProgress) {
    if (loadingProgress == null) return child;
    return CircularProgressIndicator(); // แสดง loading ขณะโหลดรูป
  },
  errorBuilder: (context, error, stackTrace) {
    return Icon(Icons.image_not_supported_outlined); // แสดงไอคอนเมื่อ error
  },
)
```

**การจัดการ Image States:**
- **Loading**: แสดง CircularProgressIndicator ขณะโหลด
- **Success**: แสดงรูปภาพเมื่อโหลดสำเร็จ
- **Error**: แสดงไอคอน fallback เมื่อโหลดไม่สำเร็จ

### 6. Stock Status Indicator
```dart
Container(
  decoration: BoxDecoration(
    color: product.stock > 0 ? Colors.green[50] : Colors.red[50],
    borderRadius: BorderRadius.circular(12),
    border: Border.all(
      color: product.stock > 0 ? Colors.green[200]! : Colors.red[200]!,
    ),
  ),
  child: Text(
    product.stock > 0 ? 'Stock: ${product.stock}' : 'Out of stock',
    style: TextStyle(
      color: product.stock > 0 ? Colors.green[700] : Colors.red[700],
    ),
  ),
)
```
- ใช้ conditional styling เปลี่ยนสีตามสถานะ stock
- สีเขียว: มี stock, สีแดง: หมด stock

## Flow การทำงาน

1. **App Launch** → MyApp widget เริ่มต้น MaterialApp
2. **ProductsScreen Init** → สร้าง Scaffold และ FutureBuilder
3. **API Call Trigger** → FutureBuilder เรียก `ProductsService.fetchProducts()`
4. **HTTP Request** → ส่ง GET request ไปยัง `https://dummyjson.com/products`
5. **Response Processing** → แปลง JSON response เป็น `List<Product>`
6. **UI Update** → FutureBuilder rebuild widget ตาม snapshot state
7. **ListView Rendering** → สร้าง ProductCard สำหรับแต่ละสินค้า
8. **Image Loading** → โหลดรูปภาพแบบ asynchronous

## วิธีการเรียกใช้ API

### API Endpoint
```
GET https://dummyjson.com/products
```

### Response Structure
```json
{
  "products": [
    {
      "id": 1,
      "title": "Product Name",
      "description": "Product Description",
      "price": 9.99,
      "rating": 4.5,
      "stock": 99,
      "brand": "Brand Name",
      "thumbnail": "https://image-url.com/image.jpg"
    }
  ],
  "total": 194,
  "skip": 0,
  "limit": 30
}
```

### HTTP Client Configuration
```dart
import 'package:http/http.dart' as http;
import 'dart:convert';

// การเรียกใช้
final response = await http.get(Uri.parse(apiUrl));
final data = json.decode(response.body);
```

## ข้อกำหนดระบบ

- Flutter SDK
- HTTP package: `http: ^1.1.0`
- Internet permission (Android/iOS)

## คุณสมบัติเด่น

✅ **Asynchronous Data Loading** - FutureBuilder จัดการ API calls  
✅ **Error Handling** - จัดการ network และ parsing errors  
✅ **Loading States** - UX indicators สำหรับทุกสถานะ  
✅ **Image Optimization** - lazy loading พร้อม error fallback  
✅ **Responsive Layout** - adaptive design สำหรับหน้าจอต่างๆ  
✅ **Clean Architecture** - แยก concerns ระหว่าง UI, Service, Model  

## การจัดการ State

แอปใช้ **Stateless Architecture** ร่วมกับ **FutureBuilder** ซึ่งเหมาะสำหรับ:
- ข้อมูลที่ไม่เปลี่ยนแปลงบ่อย
- การดึงข้อมูลครั้งเดียวเมื่อเริ่มต้น
- UI ที่ไม่ต้องการ complex state management

FutureBuilder จัดการ lifecycle ของ async operations และ rebuild UI อัตโนมัติเมื่อสถานะเปลี่ยนแปลง