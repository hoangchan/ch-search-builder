# Search Builder — User Guide

## 1. Introduction

`Search Builder` helps create dynamic filtering conditions for LINQ through a class that defines searchable fields.
You only need to declare properties and decorate them with `[FieldMap]` specifying:

* Column name(s) in the database.
* Condition type (`KindField`).
* Comparison operator (`Operation`).

The system will automatically generate an expression (`Expression`) to pass into `.Where(...)` when querying with EF Core.

---

## 2. Defining the Search Class

Example: `SearchNhapKho`
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
---

## 3. Usage in the Search Method
```csharp
var predicate = DynamicSearch.Build\<NhapKho, SearchNhapKho>(search);

var result = \_context.NhapKhos
.Include(x => x.Kho)
.Where(predicate)
.ToList();
```
---

## 4. Sample JSON Request
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
  "trangThaiList": [0]
}
```
 

## 💡 Notes

* **`KindField.Search`** → Conditions are joined using **OR**.
* **`KindField.Filter`** → Conditions are joined using **AND**.
* **`Operation`** → Defines the comparison logic.

---

## 5. Reference

### 5.1. `Range<T>`

| Property | Description |
| -------- | ----------- |
| `From`   | Start value |
| `To`     | End value   |

---

### 5.2. `KindField` — Relation Between Conditions

| Value    | LINQ Relation |
| -------- | ------------- |
| `Search` | OR            |
| `Filter` | AND           |

---

### 5.3. `Operation` — Supported Comparison Operators

| Name                 | Description                      | Notes                   |
| -------------------- | -------------------------------- | ----------------------- |
| `Contains`           | Similar to `.Contains()` in LINQ |                         |
| `EqualTo`            | Equivalent to `==`               |                         |
| `StartsWith`         | Similar to `.StartsWith()`       |                         |
| `EndsWith`           | Similar to `.EndsWith()`         |                         |
| `GreaterThan`        | `>`                              |                         |
| `LessThan`           | `<`                              |                         |
| `GreaterThanOrEqual` | `>=`                             |                         |
| `LessThanOrEqual`    | `<=`                             |                         |
| `In`                 | Value exists in list             | Type must be `List<T>`  |
| `Between`            | Value is within range            | Type must be `Range<T>` |
