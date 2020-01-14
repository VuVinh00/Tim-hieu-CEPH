## File-based Storage 

**File-storage**: là hệ thống cấp bậc các file và folder được quản lý bởi hệ thống lưu trữ. Mỗi file đều có những thuộc tính riêng như : người tạo, người có thể truy cập, kích thước được lưu trữ thành metadata trong hệ thống. Khi ta cần truy cập đến tệp dữ liệu thì ta cần phải biết chính xác đường dẫn (path) để đi từ tập nguồn đến tập đích

## Block storage

**Block storage** là lưu trữ liên tục dưới dạng khối (block), mỗi khối này có kích thước bằng nhau. 1 ổ đĩa thông thường, mảng RAID hoặc USB là một ví dụ của block storage

Với block storage, file được cắt nhỏ với kích thước bằng kích thước độ lớn của 1 block. Mỗi file này sẽ có địa chỉ riêng và ứng dụng lấy block bằng các lời gọi SCSI tới địa chỉ ( không như filesystem quyết định đâu là nơi lưu trữ dữ liệu)

Thiết bị block storage thường được đính kèm và sử dụng đọc ghi chỉ với 1 máy hoặc 1 máy ảo trong 1 thời gian ( nghĩa là chỉ có 1 máy thao tác với 1 dữ liệu ở 1 thời điểm )

## Object storage

**Object storage** là lưu trữ storage hướng đến đối tượng, lưu trữ và truy xuất dữ liệu bằng cách sử dụng API http. 1 lần đối tượng được tạo và ghi, nó không thể thay đổi mà chỉ có thể copy hoặc xóa. Vì nó không cung cấp cho việc sửa 1 lần của dữ liệu, nó coi mỗi đối tượng được lưu trữ là 1 đơn vị. Không giống như file, đối tượng ở đây được lưu trữ theo cấu trúc phẳng.

1 lần ghi các đối tượng có thể đọc được với nhiều máy khác

## So sánh các dạng Storage

<img src="https://github.com/vjnkvt/Images/blob/master/sidebyside_comparison.png">
