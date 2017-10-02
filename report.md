## Báo Cáo Docker Swarm
1, Lý thuyết, cấu trúc mô hình.
1.1, Docker Swarm là gì ?

- Docker swarm là một phần mềm hỗ trợ việc quản lý các container hoặc các hệ thống Container Orchestration. Nó là một cluster nơi mà người dùng quản lý các Docker Engines hoặc các node nơi mà các service được deploy và chạy. 
- Khi khởi chạy Docker swarm mode, người dùng sẽ sử dụng dịch vụ orchestration của Docker.
- Docker Swarm quản lý các Docker Engine ở chế độ swarm, các Docker Engine có thể được đưa vào những swarm mới hoặc join vào các swarm có sẵn.
- Docker swarm có những tính năng để hỗ trợ việc quản lý các container khi chạy trên môi trường phân tán và để chắc chắn các container trong một cluster hoạt động ổn định.
- Docker Swarm có khả năng khởi chạy các container trên nhiều máy (cluster) hoặc trên một máy duy nhất (standalone)

1.2, Các node trong swarm là gì ? Cách thức làm việc của các node.

- Các node trong Docker Swarm là các máy chạy Docker Engine được join vào swarm. Các node này có thể được gọi là một Docker Node.
- Để có thể deploy một ứng dụng trên swarm, người dùng cần phải định nghĩa các service đến manager node. Manager node này sẽ chuyển *service* này thành các *task* và đưa xuống cho các node bên dưới để thực thi
- Docker Swarm còn được coi là một dạng native cluster do các node trong cluster kể cả là manager node cũng sẽ hoạt động như một worker node. Các worker node được giao task từ manager node sẽ thông báo về cho manager node tình trạng hiện tại của task đang chạy trên node đó thông qua một *agent* ở mỗi node để cho manager node có thể đảm bảo trạng thái của mỗi node bên dưới nó.

1.3, Các service và task trong Docker Swarm.

- Service trong Docker Swarm đảm nhận chức năng định danh các task hoạt động cho manager node hoặc các worker node.
- Khi khởi tạo một service trong swarm mode, người dùng sẽ xác định container imager nào sẽ được sử dụng và câu lệnh nào sẽ chạy bên trong container đó.
- Các container images được chia sẻ và upload trên [DockerHub](https://hub.docker.com) 
- Có 2 kiểu service là:
  - **Replicated Service**: ở chế độ này thì swarm manager sẽ xác định số lượng các bản sao mà một task của một service sẽ được chạy trên các nodes dựa theo sự mong muốn của người dùng.
  - **Global Services**: ở chế độ này swarm manager sẽ chạy một task cho một service trên tất cả các node có khả năng chạy được trong cluster
 - Một task có nhiệm vụ xác định Docker container và câu lệnh sẽ được chạy bên trong container đó. Nó là khả năng sắp xếp các unit của swarm.
 - Manager node sẽ đưa các task này cho các worker node theo số lượng replicas được set trong service scale. Khi một task được đưa cho một worker node, nó sẽ không thể chuyển sang node khác, nó chỉ có thể trên node được manager node chỉ định hoặc FAIL. 
 
 1.3, Load Balancing.
 - Swarm manager sử dụng network *ingress load balancing* để đưa các service ra bên ngoài swarm. Swarm manager có thể tự động đưa các service lên các Port chưa được sử dụng trong khoảng 30000-32767. Hoặc người dùng cũng có thể tự động xác định port mà service sẽ được gán vào.
 - Các ứng dụng bên ngoài có thể truy cập vào service thông qua các PublishedPort ở bất kỳ node nào trong cluster kể cả node đó có service này hay không. 
 - Tất cả các node trong swarm đều sẽ kết nối đến route ingress để kết nối đến các task đang hoạt động.
 - Bên trong swarm mode, mỗi một service đều sẽ được gán cho một DNS nội bộ để truy cập. Swarm manager sẽ sử dụng khả năng load balancing nội bộ để gửi các yêu cầu đến các service trong cluster dựa theo DNS của service đó.

1.2, Mô hình giữa docker engine bình thường và docker swarm.
![]()
![]()

1.3, Mô hình cơ chế hoạt động của swarm.
![]()
![]()

2, Các chức năng bên trong Docker Swarm.
2.1, Docker Swarm Discovery
- Trong một cụm cluster, các Docker Engine sẽ kết nối và đăng kí đến một *distributed store* làm nơi để chứa các thông tin về chính nó.
- Swarm manager sẽ lấy data cho Swarm State từ distributed store đó, qua đó nó sẽ đọc thông tin trong state để kết nối đến Docker Engine.
![]()

- Nếu có một node bị fail health check với distributed store, Docker Swarm Manager sẽ nhận được thông tin đó và ngay lập tức rebalance container trên Docker Engine đó sang một Docker Engine khác.
![]()

2.2, Docker Swarm Scheduling.
- Docker Swarm Manager có khả năng đặt kế hoạch để đảm bảo chắc chắn rằng các nguồn lực hoạt động cho các container.
- Swarm manager sẽ xây dựng kế hoạch cho các container trong cluster dựa theo các tài nguyên mà nó có như CPU, memory, etc,... và các constraints, affinities.
- Scheduling là chức năng cho phép swarm manager khả năng quản lý thêm các container đã có hoặc được thêm mới vào cluster và hỗ trợ khả năng mở rộng cluster.
- Trong Scheduling có 2 điều cần chú ý sau:
-- Strategies:
--- **SPREAD**: Đây là default setting được dùng để cân bằng các container qua các nodes trong cluster dựa theo tài nguyên CPU, RAM cũng như các container đang chạy. Lợi ích của Spread là nếu node chứa các container bị fails thì sẽ có ít container bị mất.
![]()
---**BinPack**: Khi sử dụng strategy này, Swarm manager đảm bảo các node chứa đầy các container cho đến khi full và chuyển qua node mới trong cùng cluster. Lợi ích của BinPack là sử dụng một số lượng nhỏ các kiến trúc và có nhiều không gian cho những container lớn hơn chạy trên những máy không được sử dụng
![]()
---**Random**: Strategies này sẽ chọn ngẫu nhiên các node chứa container

-- ***Filter*** :
Swarm manager sẽ dựa theo 5 filter dưới đây để đặt schedule cho container. Nó cũng sẽ xóa bỏ các node bị fails Filter cũng như không tuân theo các constraints.

***NODE FILTER***
---**Constraint**: Cũng được biết đến như node tags, constraint là một cặp key/values thuộc về một node. Người dùng có thể chọn một hoặc nhiều cặp key/value khi xây dựng một container.
---**Health**: filter sẽ ngăn cản việc scheduling container trên node mà function này không chạy được.

***CONTAINER FILTER*** 
---**Affinity**: Đảm bảo container chạy trên các node cùng một network, filter này sẽ nói với container chạy theo giá trị tiếp theo dựa vào identifier, image, label.
---**Port**: Với filter này, port được coi như là một nguồn tài nguyên độc nhất. Khi container cố gắng kết nối đến port đã được sử dụng, nó sẽ chuyển sang node tiếp theo trong cluster.
---**Dependency**: Khi container dựa trên một container khác, filter này sẽ schedules chúng trên cùng một node.

2.3, Docker Swarm High Avaibility
- Chức năng này tăng tính chịu lỗi cho cluster và cũng để đảm bảo rằng mọi thứ hoạt động bình thưởng kể cả khi có một manager trong cluster bị down.
- Ngoài ra nó cũng đảm bảo rằng các worker node bên dưới chỉ hoạt động với một schedule duy nhất với một nguồn duy nhất, cũng như đảm bảo rằng các network không kết nối được sẽ không ảnh hưởng đến cluster.
- Chức năng này hoạt động thông qua một *external storage* bên ngoài chứa key để xác định Leader cho cluster.
![]()
- Nếu như node Leader trong nhóm các manager node bị down, *external storage* sẽ ghi nhận điều đó và thông báo đến các manager node còn lại trong cluster. Sau đó các manager này sẽ được *external storage* chọn một làm Leader tiếp theo của cluster.
- Các node manager còn lại sẽ tiếp tục forward các request đến Leader mới đó thông qua notify message từ *external storage*
- Điều này đảm bảo khả năng chịu lỗi và tính sẵn sàng cao cho cluster.
![]()

2.3, Docker Swarm Network
- Trong docker swarm có 2 kiểu traffice khác nhau là:
--**Control and management plane traffic**: đây là traffic luôn luôn bị được mã hóa vì nó chứa các swarm managerment message ví dụ như các request tham gia hoặc rời bỏ swarm.
--**Application data plane traffic** Đây là traffic của các container và các giao thức kết nối ra các clients bên ngoài.

**3 loại network với swarm service**

- Docker sử dụng *bride network* là network default được đinh nghĩa bằng Docker Daemon. Network này khi người dùng cài đặt Docker sẽ tạo nên một interface mang tên là Docker0. Interfaces này là interfaces của networkdefaults.
- Trong swarm mode, Docker Swarm sử dụng một dạng network khác của bridge là ***docker_gwbridge***. Network này là một bridge network kết nối  ***overlay networks*** bao gồm cả ***ingress network*** đến mạng vật lý của Docker Daemon. Mặc định là các container, service đang chạy đều kết nối đến ***docker_gwbridge*** network của Docker Daemon host.
- ***docker_gwbridge*** là mạng được tạo tự động khi người dùng sử dụng swarm mode. Network này được Docker cho phép người dùng chỉnh sửa.
- Docker Swarm mode sử dụng ***Overlay Network*** để quản lý việc giao tiếp giữa các Docker daemons trong swarm.
- Người dùng có thể tạo ra overlay network bằng câu lệnh

```sh
docker network create --driver overlay <network name>
```
![]()
- Overlay Network có thể được các service sử dụng, để tạo ra giao tiếp giữa các service với nhau.
- ***ingress network*** là một overlay network đặc biệt có nhiệm vụ cân bằng tải giữa các service node. Khi swarm node nhận được request từ một published port, nó sẽ chuyển request này đến module ```IPVS```, module này sẽ kiểm tra các IP tham gia vào hoạt động của service đó, chọn một trong các IP này rồi điều hướng request đến IP đó qua ***ingress network***.
- ***ingress network*** này cũng được tự động tạo ra mỗi khi người dùng khởi tạo swarm mode. Từ bản Docker17.05 người dùng có thể cấu hình network này.
![]()

- Ngoài ra, Docker Swarm Network còn hỗ trợ việc sử dụng nhiều interfaces khác nhau để điều khiển luồng dữ liệu và traffice trong swarm.







