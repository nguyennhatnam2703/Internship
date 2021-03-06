# Network File System (NFS)

## 1. Giới thiệu

- NFS là hệ thống cung cấp dịch vụ chia sẻ file phổ biến trong hệ thống mạng Linux và Unix.

- NFS cho phép các máy tính kết nối tới 1 phân vùng đĩa trên 1 máy từ xa giống như là local disk. Cho phép việc truyền file qua mạng được nhanh và trơn tru hơn 

- NFS sử dụng mô hình Client-Server. Trên server có các disk chứa các file hệ thống được chia sẻ và một số dịch vụ chạy ngầm (daemon) phục vụ cho việc chia sẻ với Client.

- Cung cấp chức năng bảo mật file và quản lý lưu lượng sử dụng (file system quota).

- Các Client muốn sử dụng các file system được chia sẻ thì sử dụng giao thức NFS để mount các file đó về.

- Khi triển khai hệ thống lớn hoặc chuyên biệt cần áp dụng 3NFS, còn người dùng ngẫu nhiên hoặc nhỏ lẻ thì áp dụng 2NFS, 4NFS.

- Với NFSv4, yêu cầu hệ thống phải có kernel phiên bản từ 2.6 trở lên.

- Xử lý được những file lớn hơn 2GB, đòi hỏi hệ thống phải có phiên bản kernel lớn hơn hoặc bằng 2.4x và glibc từ 2.2x trở lên.

- Client từ phiên bản kernel 2.2.18 trở đi đều hỗ trợ NFS trên nền TCP.

## 2. NFS Server

### 2.1. Các file cấu hình, dịch vụ

Có 3 tập tin cấu hình chính cần chỉnh sửa để thiết lập một máy chủ NFS: `/etc/exports`, `/etc/hosts.allow` và `/etc/hosts.deny`

#### `/etc/exports`

`dir host1(options) host2(options) hostN(options) ...`

- **dir**: Thư mục hoặc file system muốn chia sẻ.

- **host**: Một hoặc nhiều host được cho phép mount dir. Có thể được định nghĩa là một tên, một nhóm sử dụng ký tự , * hoặc một nhóm sử dụng 1 dải địa chỉ mạng /subnetmask ...

- **options**: Định nghĩa 1 hoặc nhiều options khi mount.

Các options:

- **ro**: Thư mục được chia sẻ chỉ đọc được

- **rw**: Client có thể đọc và ghi trên thư mục

- **no_root_squash**: Mặc định, bất kỳ file truy vấn được tạo bởi người dùng (root) máy trạm đều được xử lý tương tự nếu nó được tạo bởi user nobody. Nếu no_root_squash được chọn, user root trên client sẽ giống như root trên server.

- **no_subtree_check**: Nếu chỉ 1 phần của ổ đĩa được chia sẻ, 1 đoạn chương trình gọi là "thầm tra lại việc kiểm tra cây con" được yêu cầu từ phía client (nó là 1 file n m trong phân vùng được chia sẻ). Nếu toàn bộ ổ đĩa được chia sẻ, việc vô hiệu hoá sự kiểm tra này sẽ tăng tốc độ truyền tải.

- **sync**: Thông báo cho client biết 1 file được được ghi xong - tức là nó đã được ghi để lưu trữ an toàn - khi mà NFS hoàn thành việc kiểm soát ghi lên các file hệ thống. Cách xử lý này có thể là nguyên nhân làm sai lệch dữ liệu nếu server khởi động lại.

Ví dụ 1 file cấu hình mẫu:

```
/usr/local *.123.vn(ro)
/home 192.168.1.0/24(rw)
/var/tmp 192.168.1.1(rw) 192.168.1.3(rw)
```

- Dòng thứ nhất: Cho phép tất cả các host với tên miền định dạng "somehost".123.vn được mount thư mục `/usr/local` với quyền chỉ đọc.

- Dòng thứ hai: Cho phép bất kỳ host nào có địa chỉ IP thuộc dải mạng 192.168.1.0/24 được mount thư mục `/home` với quyền đọc và ghi.

- Dòng thứ ba: Cho phép 2 host được mount thư mục `/var/tmp` với quyền đọc và ghi.

#### `/etc/hosts.allow` và `/etc/hosts/deny`

Hai tập tin này giúp xác định các máy tính trên mạng có thể sử dụng các dịch vụ trên máy của bạn. Mỗi dòng trong nội dung file chứa duy nhất 1 danh sách gồm 1 dịch vụ và 1 nhóm các máy tính. Khi server nhận được yêu cầu từ client, các công việc sau sẽ được thực thi:

- Kiểm tra file hosts.allow - nếu client phù hợp với 1 quy tắc được liệt kê tại đây thì nó có quyền truy cập.

- Nếu client không phù hợp với 1 mục trong hosts.allow server chuyển sang kiểm tra trong hosts.deny để xem thử client có phù hợp với 1 quy tắc được liệt kê trong đó hay không (hosts.deny). Nếu phù hợp thì client bị từ chối truy cập.

- Nếu client phù hợp với các quy tắc không được liệt kê trong cả 2 file thì nó sẽ được quyền truy cập.

Ví dụ: Muốn chặn hoặc cho phép một host hoặc network thì thêm vào file deny hoặc allow.

`portmap: 10.10.10.5, 10.10.10.0/24`

### 2.2. Các dịch vụ có liên quan

Để sử dụng dịch vụ NFS, cần có các daemon sau:

- **portmap**: Quản lý các kết nối, dịch vụ chạy trên port 2049 và 111 ở cả server và client.

- **NFS**: Khởi động các tiến trình RPC (Remote Procedure Call) khi được yêu cầu để phục vụ cho chia sẻ file, dịch vụ chỉ chạy trên server.

- **NFS lock**: Sử dụng cho client khoá các file trên NFS server thông qua RPC.

#### Khởi động portmapper

NFS phụ thuộc vào tiến trình ngầm quản lý các kết nối (portmap hoặc rpc.portmap), chúng cần phải được khởi động trước.

Nó nên được đặt tại /sbin nhưng đôi khi trong /usr/sbin. Hầu hết các bản phân phối Linux gần đây đều khởi động dịch vụ này trong boot scripts nhưng vẫn phải đảm bảo nó được khởi động đầu tiên trước khi bạn làm việc với NFS (chỉ cần gõ lệnh `netstat -anp | grep portmap` để kiểm tra).

#### Các tiến trình ngầm 

Dịch vụ NFS được hỗ trợ bởi 5 tiến trình ngầm:

- **rpc.nfsd**: Thực hiện hầu hết mọi công việc.

- **rpc.lockd** và **rpc.statd**: Quản lý việc khoá các file.

- **rpc.mountd**: Quản lý các yêu cầu gắn kết lúc ban đầu.

- **rpc.rquotad**: Quản lý các hạn mức truy cập file của người sử dụng trên server được truy xuất.

- **lockd**: Được gọi theo yêu cầu của nfsd.

- **statd**: Cần phải được khởi động riêng.

Tuy nhiên trong các bản phân phối Linux gần đây đều có kịch bản khởi động cho các tiến trình trên. Tất cả các tiến trình này đều nằm trong gói nfs-utils, nó có thể được lưu giữ trong `/sbin` hoặc `/usr/sbin`. Nếu bản phân phối của bạn không tích hợp chúng trong kịch bản khởi động, thì bạn nên tự thêm chúng vào, cấu hình theo thứ tự sau:

```
rpc.portmap
rpc.mountd
rpc.nfsd 
```

### Các chế độ mount 

- Mount cứng: là ghi trực tiếp vào file /etc/fstab

- Mount mềm: là mount bằng lệnh mount thông thường và bị mất khi máy tính được khởi động lại.

## Tham khảo

http://nfs.sourceforge.net/nfs-howto/
https://github.com/hocchudong/ghichep-nfs/blob/master/NDChien_Baocao_NFS.md