Ceph có nhiều công cụ triển khai được ra đời với mục đích làm cho Ceph dễ cài đặt và quản lý. Trong đó, Ceph-deploy là một tool đơn giản và dễ hiểu (ít nhất đổi với người có kiến thức với Ceph). Ceph-deploy đã không còn được mantained, thậm chí không hoạt động trên một số distro mới như RHEL/CentOS 8.

### Cephadm

Mục tiêu là Cephadm là cung cấp đầy đủ tính năng, mạnh mẽ, cung cấp việc cài đặt và quản lý cho bất cứ ai không chạy Ceph trong Kubernetes. Mục tiêu của Cephadm bao gồm:

- Deploy all components in containers: Sử dụng containers để làm đơn giản hóa việc phụ thuộc package trên các bản phân phối khác nhau. 

- Tight intergration with the orchestrator API: 

- No dependency on management tools: Các tool như Salt hay Ansible tuyệt vời khi triển khai ở quy mô lớn, nhưng nó làm Ceph phụ thuộc vào tool, có nghĩa là người dùng sẽ phải tìm hiểu thêm một phần mềm. Quan trọng hơn là triển khai sẽ phức tạp hơn, khó debug hơn

- Minimal OS dependencies: Cephadm requires Python3, LVM và container runtime( Podman hoặc Docker). Tất cả Linux distro hiện nay đều có thể làm được

- Isolate clusters from each other: 

- Automated upgrades: Ceph có thể upgrade một cách an toàn và tự động

- Easy migration from "legacy" deployment tools: Cephadmn cho phép convert  Ceph cluster đang tồn tại được deploy bởi các tool đang tồn tại như ceph-ansible, ceph-deploy, DeepSea hay các tool tương tự 

Mục tiêu là để tập chung sự chú ý của Ceph developer và người dùng chỉ trên 2 nền tảng cho việc triển khai và quản lý Ceph, cephadm dùng cho "bare metal" và Rook để chạy Ceph trong Kubernetes

### Deploy Ceph Cluster sử dụng Cephadm

#### Requirements

- Systemd
- Podman hoặc Docker để chạy containers
- Chrony hoặc NTP để đồng bộ thời gian
- LVM2

#### Install Cephadm

Câu lệnh **cephadmn** có thể bootstrap một cluster mới, khởi động contrainerized shell để làm việc với Ceph CLI

Sử dụng **curl** để lấy phiên bản script mới nhất để cài đặt cephadm:

```
curl --silent --remote-name --location https://github.com/ceph/ceph/raw/octopus/src/cephadm/cephadm
chmod +x cephadm
```

Cài đặt packages cho phiên bản **Octopus**:

```
./cephadm add-repo --release octopus
./cephadm install
```

#### Bootstrap cluster mới

Để bootstrap cluster ta dùng câu lệnh:

```
mkdir -p /etc/ceph
cephadm bootstrap --mon-ip *<mon-ip>*
```

Output khi thành công sẽ giống như:

```
INFO:cephadm:Ceph Dashboard is now available at:

             URL: https://cephadm:8443/
            User: admin
        Password: lb777day77

INFO:cephadm:You can access the Ceph CLI with:

        sudo /usr/sbin/cephadm shell --fsid f5429e0c-7737-11ea-b0fe-fa163e9a2068 -c /etc/ceph/ceph.conf -k /etc/ceph/ceph.client.admin.keyring

INFO:cephadm:Please consider enabling telemetry to help improve Ceph:

        ceph telemetry on

For more information see:

        https://docs.ceph.com/docs/master/mgr/telemetry/

INFO:cephadm:Bootstrap complete.
```

Lệnh này sẽ:

- Tạo một monitor và manager daemon cho cluster mới trên localhost
- Generate một SSH key mới cho Ceph cluster và add key vào ``/root/.ssh/authorized_keys`` 
- Viết một minimal configuration để giao tiếp với cluster mới vào ``/etc/ceph/ceph.conf``
- Viết một bản copy của public key vào ``/etc/ceph/ceph.pub``

Default bootstrap sẽ làm một số việc cho đa số người dùng. Ta có thể xem một số options để bootstrap cluster bằng câu lệnh ``cephadm bootstrap -h``

#### Enable Ceph CLI

**Cephadm** không yêu cầu Ceph package cài đặt trên host. Để truy cập vào câu lệnh **ceph**:

- Câu lệnh ``cephadm shell`` sẽ chạy 1 bash shell trong container được cài đặt tất cả Ceph packages

```
cephadm shell
```

Ta có thể dùng alias để truy cập nhanh vào Ceph CLI ví dụ:

```
alias ceph='cephadm shell'
```

#### Add hosts to the cluster

Để thêm host mới cho cluster, thực hiện 2 bước sau:

1. Install SSH public key của cluster trong host mới:

``ssh-copy-id -f -i ceph.pub root@*<new-host>*``

2. Truy cập vào cephadm shell và nói cho Ceph node mới là một phần của cluster:

``ceph orch host add *newhost*``

#### Deploy additional montitor

Khi Ceph biết địa chỉ IP subnet monitor có thể sử dụng nó có thể tự động deploy và scale monitor. Mặc định, Ceph giả định các monitor khác sử dụng cùng subnet với subnet IP của monitor đầu tiên

Nếu Ceph monitor (hoặc toàn bộ cluster) chạy trên một subnet duy nhất thì mặc định **cephadm** sẽ tự động thêm vào 5 monitor khi add host mới vào cluster.

- Ta có thể cấu hình IP subnet được sử dụng bởi monitors bằng CIDR format:

```
ceph config set mon public_network 10.5.0.0/16*
```
- Nếu ta muốn điều chỉnh số mon tự thêm vào mặc định xuống 3 mon:

```
ceph orch apply mon 3
```
- Ta có thể control host monitor bằng cách sử dụng ``host labels``. Để set ``mon`` label dành cho host

```
ceph orch host label add *<hostname>* mon
```
- Để view các hosts hiện tại và labels:

```
[ceph: root@node1 /]# ceph orch host ls
HOST   ADDR   LABELS   STATUS
node1  node1  mon
node2  node2  osd
node3  node3  osd mon
```
- Triển khai monitor dựa trên các label:

```
ceph orch apply mon label:mon
```

- Disable tự động triển khai monitor:

```
ceph orch apply mon --unmanaged
```

- Add thêm từng mon:

```
ceph orch daemon add mon *<host1:ip-or-network1> [<host1:ip-or-network-2>...]*
```

Ví dụ deploy thêm một monitor trên ``node2`` sử dụng địa chỉ IP ``10.5.10.172``

```
ceph orch apply mon --unmanaged
ceph orch daemon add mon node2:10.5.10.172
```

#### Deploy OSDs

Một danh mục các devices trên các hosts trong cluster có thể được hiển thị với:

```
ceph orch device ls
```

Một storage device được coi là ``available`` nếu đáp ứng các điều kiện sau:

- Device không có partitions
- Device không có LVM state
- Device không được mounted
- Device không chứa filesystem
- Device không chứa Ceph BlueStore OSD
- Device phải có kích thước lớn hơn 5GB

Ceph sẽ từ chối cung cấp OSD trên các device not available

Một số cách để tạo OSDs:

- Tạo OSD trên các device available và không được sử dụng:

```
ceph orch apply osd --all-available-devices
```

- Tạo OSD bằng cách chỉ định device cụ thể trên một host cụ thể, ví dụ device ``/dev/vdb`` trên host ``node2``

```
ceph orch daemon add osd node2:/dev/vdb
```
