https://github.com/netson/ubuntu-unattended

## Định nghĩa
- Là một công cụ để tạo file ISO cài đặt Ubuntu tự động trả lời
- Tạo từ các file cài đặt cần thiết trong ISO có nguồn "sạch", không modify

## Cách làm việc:
- Chọn phiên bản muốn tạo (14,16,...)
- Tải các file cài đặt cần thiết
- Tải file preseed, chứa những câu trả lời tự động phục vụ việc cài đặt
- Lựa chọn ngôn ngữ/vùng
- Lựa chọn bàn phím
- Vô hiệu hóa tài khoản root đăng nhập
- Phân vùng ổ cứng: LVM, full disk hay single partition
- Cài đặt mkpasswd để hash các password trong thông tin đăng nhập
- Cài đặt genisoimage để tạo file ISO
- Mount file ISO vừa tải về vào thư mục tạm
- Sao chép toàn bộ các file gốc để làm việc
- Set ngôn ngữ cài đặt
- Thêm/Cập nhật thông tin vào file preseed
- Thêm tùy chọn Autoinstall vào Menu cài đặt
- Tạo file ISO
- Xóa các file tạm