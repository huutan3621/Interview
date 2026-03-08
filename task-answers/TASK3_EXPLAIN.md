# Task 3 – Merchant Product Flow

- Luồng quản lý sản phẩm của Merchant và hiển thị sản phẩm cho khách bên B2C: Merchant tạo sản phẩm => sản phẩm lên store => khách hàng đặt => hệ thống chọn courier giao

---

# Backend

## Điều kiện tạo sản phẩm

Merchant phải có:

status = APPROVED

Nếu chưa được duyệt thì không được tạo sản phẩm

---

## Kiểm tra role

API tạo sản phẩm chỉ cho:

MERCHANT_OWNER

Nếu không đúng role thì từ chối

---

## Merchant chỉ tạo sản phẩm cho store của mình

Kiểm tra:

merchantId == currentUserMerchantId

Không đúng thì reject request

---

## API danh sách sản phẩm

API:

GET /products

Có hỗ trợ pagination:

GET /products?page=1&pageSize=10

---

## Điều kiện courier nhận đơn

Courier phải:

status = APPROVED  
status = ONLINE

Nếu chưa duyệt hoặc đang offline thì không nhận đơn

---

# Frontend Merchant

Merchant có thể:

Create Product  
Update Product  
Delete Product

Trang quản lý hiển thị danh sách sản phẩm của merchant

Form tạo sản phẩm gồm:

name  
description  
price  
images  
category

---

# B2C Storefront

Khách hàng có thể:

Xem danh sách sản phẩm  
Xem chi tiết sản phẩm  
Thêm vào giỏ hàng  
Đặt hàng

Flow:

User thêm sản phẩm => tạo order => hệ thống chọn courier giao

---

# Technical Notes

- Kiểm tra role để tránh user tạo sản phẩm lung tung
- Kiểm tra merchant status để đảm bảo merchant đã được duyệt
- Dùng pagination để tránh trả quá nhiều dữ liệu
- Chỉ courier APPROVED + ONLINE mới nhận đơn
