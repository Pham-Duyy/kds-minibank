# MiniBank – Thiết kế API

Phiên bản: 0.1 · Người viết: Phạm Khương Duy · Ngày: 26/11/2026

## 1. Quy ước chung

- Định dạng dữ liệu: JSON (`Content-Type: application/json`)
- Xác thực: header `Authorization: Bearer <token>`
- Số tiền: số nguyên, đơn vị VND (không dùng số thập phân)
- Thời gian: dạng `2026-11-26T09:00:00`
- `{id}` trong URL là số định danh, ví dụ `/accounts/{id}` → `/accounts/1`

## 2. Danh sách API

### Khách hàng

| # | Chức năng | Method | URL | Thành công |
|---|---|---|---|---|
| 1 | Xem danh sách khách hàng | GET | /customers | 200 |
| 2 | Tìm khách hàng theo tên | GET | /customers?name={tên} | 200 |
| 3 | Xem thông tin một khách hàng | GET | /customers/{id} | 200 |
| 4 | Đăng ký khách hàng mới | POST | /customers | 201 |
| 5 | Xóa khách hàng | DELETE | /customers/{id} | 204 |
| 6 | Xem các tài khoản của khách hàng | GET | /customers/{id}/accounts | 200 |

### Tài khoản

| # | Chức năng | Method | URL | Thành công |
|---|---|---|---|---|
| 7 | Xem danh sách tài khoản | GET | /accounts | 200 |
| 8 | Xem một tài khoản | GET | /accounts/{id} | 200 |
| 9 | Mở tài khoản mới | POST | /accounts | 201 |
| 10 | Cập nhật toàn bộ tài khoản | PUT | /accounts/{id} | 200 |
| 11 | Khóa / mở khóa tài khoản | PATCH | /accounts/{id} | 200 |
| 12 | Xóa tài khoản | DELETE | /accounts/{id} | 204 |

### Giao dịch

| # | Chức năng | Method | URL | Thành công |
|---|---|---|---|---|
| 13 | Nạp tiền | POST | /accounts/{id}/deposits | 201 |
| 14 | Rút tiền | POST | /accounts/{id}/withdrawals | 201 |
| 15 | Chuyển tiền | POST | /transfers | 201 |
| 16 | Xem lịch sử giao dịch theo tháng | GET | /accounts/{id}/transactions?month=2026-11 | 200 |
| 17 | Xem chi tiết một giao dịch | GET | /transactions/{id} | 200 |

## 3. Chi tiết API

### 3.1. Chuyển tiền

`POST /transfers`

Request body:

```json
{
  "fromAccountId": 1,
  "toAccountId": 2,
  "amount": 500000,
  "description": "Tra tien an trua"
}
```

Response thành công (201):

```json
{
  "transactionId": 1001,
  "type": "TRANSFER",
  "fromAccountId": 1,
  "toAccountId": 2,
  "amount": 500000,
  "status": "SUCCESS",
  "createdAt": "2026-11-26T09:00:00"
}
```

Lỗi có thể gặp: `INVALID_AMOUNT`, `SOURCE_ACCOUNT_NOT_FOUND`, `DESTINATION_ACCOUNT_NOT_FOUND`, `INSUFFICIENT_BALANCE`, `ACCOUNT_ACCESS_DENIED`

### 3.2. Rút tiền

`POST /accounts/{id}/withdrawals`

Request body:

```json
{
  "amount": 300000,
  "description": "Rut tien mat"
}
```

Response thành công (201):

```json
{
  "transactionId": 1002,
  "accountId": 1,
  "type": "WITHDRAWAL",
  "amount": 300000,
  "status": "SUCCESS",
  "createdAt": "2026-11-26T09:00:00"
}
```

Lỗi có thể gặp: `INVALID_AMOUNT`, `ACCOUNT_NOT_FOUND`, `INSUFFICIENT_BALANCE`

### 3.3. Nạp tiền

`POST /accounts/{id}/deposits`

Request body:

```json
{
  "amount": 2000000,
  "description": "Nop tien mat"
}
```

Response thành công (201):

```json
{
  "transactionId": 1003,
  "accountId": 1,
  "type": "DEPOSIT",
  "amount": 2000000,
  "status": "SUCCESS",
  "createdAt": "2026-11-26T09:00:00"
}
```

Lỗi có thể gặp: `INVALID_AMOUNT`, `ACCOUNT_NOT_FOUND`

### 3.4. Khóa / mở khóa tài khoản

`PATCH /accounts/{id}`

Request body (khóa: `LOCKED`, mở khóa: `ACTIVE`):

```json
{
  "status": "LOCKED"
}
```

Response thành công (200): thông tin tài khoản sau khi cập nhật.

## 4. Mã lỗi

| errorCode | Status | Khi nào xảy ra |
|---|---|---|
| INVALID_JSON | 400 | Body gửi lên không phải JSON hợp lệ |
| INVALID_AMOUNT | 400 | Số tiền nhỏ hơn hoặc bằng 0 |
| UNAUTHENTICATED | 401 | Chưa đăng nhập hoặc token hết hạn |
| ACCOUNT_ACCESS_DENIED | 403 | Người đăng nhập không có quyền với tài khoản này |
| ACCOUNT_NOT_FOUND | 404 | Tài khoản không tồn tại |
| SOURCE_ACCOUNT_NOT_FOUND | 404 | Tài khoản gửi không tồn tại |
| DESTINATION_ACCOUNT_NOT_FOUND | 404 | Tài khoản nhận không tồn tại |
| EMAIL_ALREADY_EXISTS | 409 | Email khách hàng đã được dùng |
| INSUFFICIENT_BALANCE | 409 | Số dư không đủ để rút hoặc chuyển tiền |

Mẫu response lỗi:

```json
{
  "status": "ERROR",
  "errorCode": "INSUFFICIENT_BALANCE",
  "message": "So du tai khoan khong du de thuc hien giao dich",
  "timestamp": "2026-11-26T09:00:00"
}
```

Quy tắc: không bao giờ trả stack trace hoặc thông tin bên trong server cho client.

## 5. Câu hỏi còn mở

- Chuyển tiền khi `fromAccountId` trùng `toAccountId` thì xử lý thế nào? Cần thêm mã lỗi không?