# Docker Security 
## Ngữ cảnh
Docker giúp phân phối nhanh chóng, nhất quán các ứng dụng; cho phép chạy nhiều khối lượng công việc hơn trên cùng một phần cứng. Docker tiện lợi nhưng nó cũng tiềm ẩn nhiều lỗ hỏng bảo mật, như là nó cho phép kẻ tấn công có thể thực thi code từ xa hay là leo thang đặc quyền…

Do đó, điều quan trọng là phải bảo vệ Docker Engine chống lại các mối đe dọa có thể xảy ra, đặc biệt nếu chúng ta đang chạy một máy chủ Docker trong sản xuất, thương mại.

![Vulnerabilities](./images/Vulnerabilities.png)

Vậy cần phải làm gì để có môi trường Docker an toàn

## Docker Security Cheat Sheet
### 1. Keep Host and Docker up to date
Điều đầu tiên luôn là cập nhật các bản vá mới nhất cho Host và Docker để ngăn chặn các lỗ hỏng đã biết.

### 2. Do not expose the Docker daemon socket (even to the containers)
Docker thường hoạt động như là một client giao tiếp với tiến trình Daemon. Daemon có đặc quyền rất cao, thường là quyền root. Vì vậy nếu user bất kì có quyền truy cập vào Daemon đồng nghĩa với user đó có được quyền root.

### 3. Set User
Cấu hình container cho người dùng không có đặc quyền là cách tốt nhất để ngăn chặn các cuộc tấn công leo thang đặc quyền.

### 4. Limit capabilities
Ta chỉ nên set cho Docker các đặc quyền vừa đủ, không nên cho nó quá nhiều đặc quyền.

### 5. Add –no-new-privileges flag
Luôn chạy Docker với cờ --no-new-privileges để ngăn chặn leo thang đặc quyền

### 6. Disable inter-container communication (--icc=false)
Theo mặc định, inter-container communication (icc) được bật - có nghĩa là tất cả các container có thể giao tiếp với nhau. Điều này có thể làm lộ thông tin không mong muốn cho các container khác.

### 7. Use Linux Security Module (seccomp, AppArmor, or SELinux)
Ta nên sử dụng các security module có sẵn trên linux kernel như là seccomp, AppArmor, SELinux

### 8. Some other rules
- Hạn chế tài nguyên (Tránh tấn công Dos)
- Set file system và volume thành read-only
- Sử dụng các công cụ phân tích tĩnh
- Đặt mức ghi log thành ít nhất là INFO


*Ý thức được sự cần thiết của việc đảm bảo cho môi trường Docker đủ “cứng” để có thể đối mặt với các mối đe dọa, đội ngũ phát triển Docker đã tạo ra Docker Bench như là một công cụ kiểm tra bảo mật mã nguồn mở. Docker Bench for Security là một automated script giúp ta có thể tìm ra các vấn đề với cấu hình của mình. Script scan host để tìm điểm yếu trong việc thiết lập Docker Engine.*

## What does Docker Bench for Security do?
Docker Bench for Security cho phép quản trị viên xây dựng security baseline trong quá trình triển khai Docker. Công cụ quét bảo mật này nên được sử dụng để làm cứng các host configurations.

Docker Bench scan Docker host để tìm các vấn đề cấu hình phổ biến, chẳng hạn như cài đặt lỏng lẻo trong tệp cấu hình, hay là quyền hệ thống và các giá trị mặc định có vấn đề. Công cụ này dựa trên cơ sở dữ liệu về các CVE để kiểm tra các thư viện và tệp thực thi trên hệ thống được đề cập.

Sau khi scan, nó sẽ cung cấp điểm bảo mật. Quản trị viên có thể theo dõi điểm này để đánh dấu các cải tiến theo thời gian. Điểm số đó được tăng lên khi cấu hình Pass và ngược lại, nó sẽ giảm khi bị Warn, và điểm cao hơn rất có thể sẽ bảo mật tốt hơn.

## What needs to be secured?
![Docker Bench for Security](./images/DockerBench.png)

### 1. Host Configuration
Phần này bao gồm các khuyến nghị bảo mật mà ta nên tuân theo để chuẩn bị cho máy host. Nó làm theo các phương pháp tốt nhất về bảo mật cơ sở hạ tầng để xây dựng một nền tảng vững chắc và an toàn để thực hiện các khối lượng công việc trên container.

Phần này tập trung vào các điểm yếu trong quá trình kiểm tra bảo mật trên host. Nó sẽ kiểm tra các thư mục Docker, đảm bảo sử dụng phân vùng dành riêng cho containers và đảm bảo cài đặt phiên bản Docker cập nhật.

![Host Configuration](./images/HostConfiguration.png)

### 2. Docker Daemon Configuration
Phần này sẽ kiểm tra Docker’s socket có bị lộ qua một kết nối không an toàn hay không. Lưu lượng mạng giữa các container trên mạng bridge mặc định nên bị hạn chế và loại bỏ registries không an toàn.

Phần này cũng tìm kiếm các đặc quyền không phù hợp cho các containers. Containers không nên có được các đặc quyền mới, vì điều này có thể cho phép kẻ tấn công phát triển vượt quá container.

![Docker Daemon Configuration](./images/DockerDaemonConfiguration.png)

### 3. Docker Daemon Configuration Files
Các tệp cấu hình Docker Daemon có độ nhạy cảm cao và có thể cho phép kẻ tấn công kiểm soát tất cả các containers trên host.

Phần này bao gồm các quyền hạn và quyền sở hữu các tệp và thư mục liên quan đến Docker. Giữ an toàn cho các tệp và thư mục có thể chứa các tham số nhạy cảm là điều quan trọng để Docker daemon hoạt động chính xác và an toàn.

![Docker Daemon Configuration Files](./images/DockerDaemonConfigurationFiles.png)

### 4. Container Images and Build File
Container base images và build files chi phối các nguyên tắc cơ bản về cách một cá thể container từ một images cụ thể sẽ hoạt động. Đảm bảo rằng ta đang sử dụng base images và các build files thích hợp có thể rất quan trọng để xây dựng cơ sở hạ tầng dựa trên container.

Docker Bench thực hiện kiểm tra cơ bản các Dockerfiles cho các image hiện có. Nó sẽ tìm kiếm container users chuyên dụng, sự hiện diện của các lệnh HEALTHCHECK và việc sử dụng Content Trust để xác minh tính toàn vẹn của dữ liệu.

Phần kiểm tra này cũng sẽ phát ra các cảnh báo nhắc nhở về các bước làm cứng image cơ bản. Sử dụng base images đáng tin cậy, áp dụng các bản vá bảo mật mới và tránh cài đặt các gói không cần thiết. Các biện pháp này giúp loại bỏ các lỗ hổng bên trong các containers.

![Container Images and Build File](./images/ContainerImagesandBuildFile.png)

### 5. Container Runtime
Container Runtime sẽ kiểm tra các containers đang hoạt động. Phần này bao gồm hơn 30 bài tests, từ tính khả dụng của SELinux và AppArmor đến việc sử dụng các tùy chọn mạng và mount file system thích hợp.

Containers không được có thêm đặc quyền hoặc can thiệp vào host system.

DockerBench cũng tìm kiếm các SSH server chạy bên trong các container. Điều này không được phép xảy ra, vì ta nên tránh truy cập trực tiếp vào container. Tốt hơn hết là sử dụng trình điều khiển docker từ host để tương tác với các container.

Nó cũng xem xét việc sử dụng CPU và giới hạn bộ nhớ. Một container không giới hạn có thể tiêu tốn quá nhiều tài nguyên và gây ra tình trạng hết bộ nhớ trên host.

Và cuối cùng, networking checks sẽ gắn cờ các port không cần thiết trong container.

![Container Runtime](./images/ContainerRuntime.png)

### 6. Docker Security Operations
Phần này bao gồm một số vấn đề bảo mật hoạt động liên quan đến triển khai Docker. Đây là những phương pháp tốt nhất nên được tuân theo nếu có thể. Hầu hết các đề xuất trong phần này chỉ đóng vai trò là lời nhắc nhở cho các tổ chức nên mở rộng các chính sách và thực tiễn tốt nhất về bảo mật.

### 7. Docker Swarm Configuration
Phần này bao gồm các khuyến nghị bảo mật mà ta nên tuân theo để chuẩn bị cho máy host. Nó làm theo các phương pháp tốt nhất về bảo mật cơ sở hạ tầng để xây dựng một nền tảng vững chắc và an toàn để thực hiện các khối lượng công việc trên container.

Phần này tập trung vào các điểm yếu trong quá trình kiểm tra bảo mật trên host. Nó sẽ kiểm tra các thư mục Docker, đảm bảo sử dụng phân vùng dành riêng cho containers và đảm bảo cài đặt phiên bản Docker cập nhật.

![Docker Swarm Configuration](./images/DockerSwarmConfiguration.png)

## Tool: https://github.com/docker/docker-bench-security
### Cách sử dụng: 
    git clone https://github.com/docker/docker-bench-security
    cd docker-bench-security
    sudo ./docker-bench-security.sh

## Video demo: https://youtu.be/Mp0qlgveWCE

## Tài liệu tham khảo:
- https://www.cisecurity.org/benchmark/docker
- https://www.aquasec.com/cloud-native-academy/docker-container/docker-cis-benchmark/
- https://www.howtogeek.com/devops/how-to-automate-docker-security-audits-with-docker-bench-for-security/