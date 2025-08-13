# Search Builder — Hướng dẫn sử dụng

## 1. Giới thiệu

`Search Builder` hỗ trợ tạo điều kiện lọc dữ liệu động cho LINQ thông qua một lớp định nghĩa các trường tìm kiếm.
Bạn chỉ cần khai báo các thuộc tính và gán `[FieldMap]` với:

* Tên cột hoặc nhiều cột trong DB.
* Loại điều kiện (`KindField`).
* Toán tử so sánh (`Operation`).

Hệ thống sẽ tự sinh biểu thức (`Expression`) để truyền vào `.Where(...)` khi truy vấn EF Core.

---

## 2. Định nghĩa class Search

Ví dụ: `SearchNhapKho`
```csharp
public class SearchNhapKho
{
  [FieldMap("Ma,Kho.Ten", KindField.Search, Operation.Contains)]
  public string? KeySearch { get; set; }
  
  [FieldMap("Kho.Ten", KindField.Search, Operation.Contains)]
  public string? TuKhoa { get; set; }
  
  [FieldMap("Ma", KindField.Filter, Operation.EqualTo)]
  public string? Ma { get; set; }
  
  [FieldMap("Id", KindField.Filter, Operation.EqualTo)]
  public int? Id { get; set; }
  
  [FieldMap("SoLuong", KindField.Filter, Operation.GreaterThan)]
  public int? SoLuongMin { get; set; }
  
  [FieldMap("SoLuong", KindField.Filter, Operation.LessThan)]
  public int? SoLuongMax { get; set; }
  
  [FieldMap("NgayNhap", KindField.Filter, Operation.Between)]
  public Range<DateTime?>? NgayNhapTuDen { get; set; }  
  
  [FieldMap("TrangThai", KindField.Filter, Operation.In)]
  public List<int>? TrangThaiList { get; set; }  
}
```

## 3. Sử dụng trong hàm Search
```
var predicate = DynamicSearch.Build\<NhapKho, SearchNhapKho>(search);

var result = \_context.NhapKhos
.Include(x => x.Kho)
.Where(predicate)
.ToList();

```

## 4. Request JSON mẫu
```json
{
"keySearch": "string",
"tuKhoa": "string",
"ma": "string",
"id": 0,
"soLuongMin": 0,
"soLuongMax": 0,
"ngayNhapTuDen": {
"from": "2025-08-13T12:13:16.065Z",
"to": "2025-08-13T12:13:16.065Z"
},
"trangThaiList": \[0]
}
```

## 💡 Ghi nhớ

* **`KindField.Search`** → Các điều kiện nối bằng **OR**.
* **`KindField.Filter`** → Các điều kiện nối bằng **AND**.
* **`Operation`** → Xác định cách so sánh.

---

## 5. Tham chiếu

### 5.1. `Range<T>`

| Thuộc tính | Mô tả            |
| ---------- | ---------------- |
| `From`     | Giá trị bắt đầu  |
| `To`       | Giá trị kết thúc |

---

### 5.2. `KindField` — Quan hệ giữa điều kiện

| Giá trị  | Quan hệ trong LINQ |
| -------- | ------------------ |
| `Search` | OR                 |
| `Filter` | AND                |

---

### 5.3. `Operation` — Các phép so sánh hỗ trợ

| Tên                  | Mô tả                             | Ghi chú                 |
| -------------------- | --------------------------------- | ----------------------- |
| `Contains`           | Tương tự `.Contains()` trong LINQ |                         |
| `EqualTo`            | Tương tự `==`                     |                         |
| `StartsWith`         | Tương tự `.StartsWith()`          |                         |
| `EndsWith`           | Tương tự `.EndsWith()`            |                         |
| `GreaterThan`        | `>`                               |                         |
| `LessThan`           | `<`                               |                         |
| `GreaterThanOrEqual` | `>=`                              |                         |
| `LessThanOrEqual`    | `<=`                              |                         |
| `In`                 | Chứa trong danh sách              | Kiểu phải là `List<T>`  |
| `Between`            | Trong khoảng                      | Kiểu phải là `Range<T>` |
