# BÀI TẬP VẬN DỤNG CHUYÊN SÂU: THIẾT KẾ GIẢI PHÁP PROMPTING TỰ ĐỘNG HÓA

**Bối cảnh áp dụng:** Tự động hóa quy trình trích xuất thông tin đặt hàng từ tin nhắn lộn xộn của khách hàng (Zalo/Facebook) sang dữ liệu chuẩn **JSON** để đẩy thẳng vào hệ thống quản lý của cửa hàng TechShop.

---

## BƯỚC 1: THIẾT KẾ GIẢI PHÁP (INPUT / OUTPUT)

* **Input (Đầu vào):** Đoạn văn bản tin nhắn tự nhiên của khách hàng gửi cho shop (thường viết tắt, thiếu dấu, thiếu thông tin hoặc sai định dạng).
* **Output (Đầu ra):** Chuỗi dữ liệu định dạng **JSON** chuẩn xác gồm các trường: `ten_khach`, `so_dien_thoai`, `san_pham`, `dia_chi`, `trang_thai_du_lieu` và `ghi_chu_loi`.
* **Cơ chế bẫy lỗi (Xử lý dữ liệu trống/sai lệch):**
  * Nếu thiếu bất kỳ thông tin bắt buộc nào (Tên, SĐT, Địa chỉ, Sản phẩm) $\rightarrow$ Điền giá trị `null`.
  * Nếu SĐT không đủ 10 số hoặc sai định dạng $\rightarrow$ Đánh dấu `"trang_thai_du_lieu": "CAN_KIEM_TRA"` và ghi rõ lý do vào trường `ghi_chu_loi`.

---

## BƯỚC 2: VIẾT PROMPT HOÀN CHỈNH & KẾT QUẢ CHẠY THỬ

### 1. Master Prompt (Kết hợp CREATE + Few-shot + Ép định dạng JSON)

> **System / Prompt:**
> * **Character:** Đóng vai là Hệ thống AI Trích xuất Dữ liệu Đơn hàng tự động của chuỗi bán lẻ TechShop.
> * **Request:** Hãy đọc tin nhắn đặt hàng của khách dưới đây, trích xuất các thông tin quan trọng và xuất ra duy nhất một khối mã JSON hợp lệ.
> * **Rules & Edge Cases (Ràng buộc bẫy lỗi):**
>   1. Nếu thiếu thông tin nào, bắt buộc gán giá trị `null` (tuyệt đối không tự suy diễn).
>   2. Kiểm tra số điện thoại: Nếu không phải 10 chữ số bắt đầu bằng số 0, gán `"trang_thai_du_lieu": "LOI_DU_LIEU"` và ghi rõ lỗi ở `"ghi_chu_loi"`. Nếu đủ và đúng, gán `"HOP_LE"`.
> * **Examples (Few-shot mẫu):**
>   * *Input:* "Shop ơi ship cho anh Tuấn 1 con chuột Logitech G102 tới 12 Nguyễn Trãi Q1 nhé, sđt 0901234567"
>   * *Output:*
>     ```json
>     {
>       "ten_khach": "Anh Tuấn",
>       "so_dien_thoai": "0901234567",
>       "san_pham": "Chuột Logitech G102",
>       "dia_chi": "12 Nguyễn Trãi, Q1",
>       "trang_thai_du_lieu": "HOP_LE",
>       "ghi_chu_loi": null
>     }
>     ```
> * **Type of Output:** Chỉ trả về đúng định dạng JSON, không viết thêm bất kỳ lời giải thích nào bên ngoài.
> 
> **Dữ liệu cần xử lý (Input thực tế):**  
> *"Chào shop, lấy cho chị Lan 2 thanh RAM 8GB DDR4 giao về số 5 Lê Duẩn Đà Nẵng nha. Số chị 098765432"*

---

### 2. Kết quả phản hồi thực tế từ AI (Output)

```json
{
  "ten_khach": "Chị Lan",
  "so_dien_thoai": "098765432",
  "san_pham": "2 thanh RAM 8GB DDR4",
  "dia_chi": "Số 5 Lê Duẩn, Đà Nẵng",
  "trang_thai_du_lieu": "LOI_DU_LIEU",
  "ghi_chu_loi": "Số điện thoại chỉ có 9 chữ số, thiếu 1 số so với quy chuẩn."
}
```
