# Cài đặt CEPH `Nautilus` Cluster trên CentOS 7
### **Mô hình**
## **1) Cài đặt ban đầu trên các node**
- **B1 :** Update các gói phần mềm và các package cơ bản :
    ```
    # yum update -y
    ```
- **B2 :** Thiết lập hostname :
    ```
    # hostnamectl set-hostname [ceph-1|ceph-2|ceph-3]
    # bash
    ```
- **B3 :** Khai báo file `/etc/hosts` :
    ```
    # echo "127.0.0.1 localhost" > /etc/hosts
    # echo "10.5.11.101 ceph-1" >> /etc/hosts
    # echo "10.5.11.102 ceph-2" >> /etc/hosts
    # echo "10.5.11.103 ceph-3" >> /etc/hosts
    ```
- **B4 :** Thiết lập phân hoạch IP cho node
- **B5 :** Disable **Firewalld** và **SELinux** :
    ```
    # sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
    # sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
    # systemctl disable firewalld
    # systemctl stop firewalld
    # reboot
    ```
## **2) Cài đặt Ceph**
### **2.1) Cài đặt NTP**
#### **2.1.1) Cài đặt trên node `ceph_1`**
- **B1 :** Cài đặt chrony sử dụng làm **NTP** :
    ```
    # yum -y install chrony
    ```
- **B2 :** Sao lưu file cấu hình của `chrony` :
    ```
    # cp /etc/chrony.conf /etc/chrony.conf.bak
    ```
- **B3 :** Set timezone :
    ```
    # timedatectl set-timezone Asia/Ho_Chi_Minh
    ```
- **B4 :** Sửa file cấu hình :
    ```
    # sed -i s'/0.centos.pool.ntp.org/128.138.140.44/'g /etc/chrony.conf
    # sed -i s'/server 1.centos.pool.ntp.org iburst/#server 1.centos.pool.ntp.org iburst/'g /etc/chrony.conf
    # sed -i s'/server 2.centos.pool.ntp.org iburst/#server 2.centos.pool.ntp.org iburst/'g /etc/chrony.conf
    # sed -i s'/server 3.centos.pool.ntp.org iburst/#server 3.centos.pool.ntp.org iburst/'g /etc/chrony.conf
    ```
    > Node `ceph_1` sẽ cập nhật thời gian từ internet hoặc máy chủ **NTP**. Các node còn lại sẽ đồng bộ thời gian từ `ceph_1`. Trong bài lab này sẽ sử dụng địa chỉ NTP của NIST là `128.138.140.44` (có thể thay thế bằng IP của NTP Server trong mạng).
- **B5 :**  Để cho phép các node `ceph` kết nối với `chrony` trên node `ceph_1`, ta sẽ sửa file cấu hình allow subnet chung của các node :
    ```
    # sed -i s'|#allow 192.168.0.0/16|allow 10.5.8.0/22|'g /etc/chrony.conf
    ```
- **B6 :** Khởi động lại `chrony` sau khi sửa file cấu hình :
    ```
    # systemctl start chronyd
    # systemctl enable chronyd
    ```
- **B7 :** Kiểm tra lại trạng thái của `chrony` :
    ```
    # systemctl status chronyd
    ```
    <img src=https://i.imgur.com/lO08Krx.png>
- **B8 :** Kiểm tra xem thời gian đã được đồng bộ chưa :
    ```
    # chronyc sources
    ```
    <img src=https://i.imgur.com/PvcoRI2.png>
#### **2.1.2) Cài đặt trên các node còn lại**
- **B1 :** Cài đặt chrony sử dụng làm **NTP** :
    ```
    # yum -y install chrony
    ```
- **B2 :** Sao lưu file cấu hình của `chrony` :
    ```
    # cp /etc/chrony.conf /etc/chrony.conf.bak
    ```
- **B3 :** Set timezone :
    ```
    # timedatectl set-timezone Asia/Ho_Chi_Minh
    ```
- **B4 :** Sửa file cấu hình :
    ```
    # sed -i s'/0.centos.pool.ntp.org/ceph-1/'g /etc/chrony.conf
    # sed -i s'/server 1.centos.pool.ntp.org iburst/#server 1.centos.pool.ntp.org iburst/'g /etc/chrony.conf
    # sed -i s'/server 2.centos.pool.ntp.org iburst/#server 2.centos.pool.ntp.org iburst/'g /etc/chrony.conf
    # sed -i s'/server 3.centos.pool.ntp.org iburst/#server 3.centos.pool.ntp.org iburst/'g /etc/chrony.conf
    ```
- **B6 :** Khởi động lại `chrony` sau khi sửa file cấu hình :
    ```
    # systemctl start chronyd
    # systemctl enable chronyd
    ```
- **B7 :** Kiểm tra lại trạng thái của `chrony` :
    ```
    # systemctl status chronyd
    ```
    <img src=https://i.imgur.com/J8vStsm.png>
    <img src=https://i.imgur.com/fZCKdOj.png>
- **B8 :** Kiểm tra xem thời gian đã được đồng bộ chưa :
    ```
    # chronyc sources
    ```
    <img src=https://i.imgur.com/efzQDEM.png>
    <img src=https://i.imgur.com/Zgg3aY0.png>
### **2.2) Tạo user để cài đặt `Ceph` trên cả 3 node**
- **B1 :** Tạo user `ceph_user` với password "`Password123`" :
    ```
    # useradd ceph_user; echo 'Password123' | passwd ceph_user --stdin
    ```
- **B2 :** Cấp quyền sudo cho user `ceph_user` :
    ```
    # echo "ceph_user ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ceph_user
    # chmod 0440 /etc/sudoers.d/ceph_user
    ```
### **2.3) Khai báo repo cho `ceph`**
- **B1 :** Chỉnh sửa file repo :
    ```
    # vi /etc/yum.repos.d/ceph.repo
    ```
    - Thêm vào đoạn sau :
        ```ini
        [ceph]
        name=Ceph packages for $basearch
        baseurl=https://download.ceph.com/rpm-nautilus/el7/x86_64/
        enabled=1
        priority=2
        gpgcheck=1
        gpgkey=https://download.ceph.com/keys/release.asc

        [ceph-noarch]
        name=Ceph noarch packages
        baseurl=https://download.ceph.com/rpm-nautilus/el7/noarch
        enabled=1
        priority=2
        gpgcheck=1
        gpgkey=https://download.ceph.com/keys/release.asc

        [ceph-source]
        name=Ceph source packages
        baseurl=https://download.ceph.com/rpm-nautilus/el7/SRPMS
        enabled=0
        priority=2
        gpgcheck=1
        gpgkey=https://download.ceph.com/keys/release.asc
        ```
- **B2 :** Update lại các repo :
    ```
    # yum update -y
    ```
### **2.4) Cài đặt `ceph-deploy`**
- Có thể cài đặt `Ceph` theo 3 cách, bao gồm: CEPH manual, `ceph-deploy` và `ceph-ansible`. Ở đây ta sẽ sử dụng `ceph-deploy`, một công cụ để triển khai CEPH.
- Node cài đặt `ceph-deploy` được gọi là node `admin` .
- Chỉ cần đứng trên `ceph_1` thực hiện, một số thao tác trên node `ceph_2`, `ceph_3` sẽ thực hiện từ xa ngay trên `ceph_1`.
- **B1 :** Cài đặt `ceph-deploy` :
    ```
    # yum install -y epel-release ceph-deploy
    ```
- **B2 :** Chuyển sang user `ceph_user` :
    ```
    # su - ceph_user
    ```
- **B3 :** Tạo ssh key cho user `ceph_user` :
    ```
    $ ssh-keygen -t rsa
    ```
- **B4 :** Copy public key vừa tạo sang các node :
    ```
    $ ssh-copy-id ceph_user@ceph-1
    $ ssh-copy-id ceph_user@ceph-2
    $ ssh-copy-id ceph_user@ceph-3
    ```
- **B5 :** Tạo thư mục chứa các file cấu hình khi cài đặt `ceph` :
    ```
    $ cd ~
    $ mkdir ceph-cluster
    $ cd ceph-cluster
    ```
- **B6 :** Khai báo các node **Ceph** trong cluster :
    ```
    $ ceph-deploy new ceph-1 ceph-2 ceph-3
    ```
    - Lệnh trên sẽ sinh ra các file cấu hình trong thư mục hiện tại, kiểm tra bằng lệnh `ls – lah`
        <img src=https://i.imgur.com/Jk6mjoF.png>

- **B7 :** Khai báo thêm các tùy chọn cho việc triển khai, vận hành **Ceph** vào file `ceph.conf` này trước khi cài đặt các gói cần thiết cho ceph trên các node. Lưu ý các tham số về network. Ta sẽ dụng vlan `10.5.8.0/22` cho đường truy cập của các client (Hay gọi là `ceph public`). Vlan `10.10.230.0/24` cho đường replicate dữ liệu, các dữ liệu sẽ được sao chép & nhân bản qua vlan này.
    ```
    $ echo "public network = 10.5.8.0/22" >> ceph.conf
    $ echo "cluster network = 172.16.69.0/24" >> ceph.conf
    $ echo "osd objectstore = bluestore"  >> ceph.conf
    $ echo "mon_allow_pool_delete = true"  >> ceph.conf
    $ echo "osd pool default size = 3"  >> ceph.conf
    $ echo "osd pool default min size = 1"  >> ceph.conf
    ```
- **B8 :** Cài đặt phiên bản **Ceph `Octopus`** lên các node `ceph-1`, `ceph-2`, `ceph-3`:
    ```
    $ ceph-deploy install --release nautilus ceph-1 ceph-2 ceph-3 --repo-url https://download.ceph.com/rpm-nautilus/el7
    ```
    ```
    ...
    [ceph-1][DEBUG ] ceph version 14.2.12 (2f3caa3b8b3d5c5f2719a1e9d8e7deea5ae1a5c6) nautilus (stable)
    ...
    [ceph-2][DEBUG ] ceph version 14.2.12 (2f3caa3b8b3d5c5f2719a1e9d8e7deea5ae1a5c6) nautilus (stable)
    ...
    [ceph-3][DEBUG ] ceph version 14.2.12 (2f3caa3b8b3d5c5f2719a1e9d8e7deea5ae1a5c6) nautilus (stable)
    ```
- **B9 :** Thiết lập thành phần MON cho **Ceph** trên cả 3 node :
    ```
    $ ceph-deploy mon create-initial
    ```
    - Kết quả sinh ra các file trong thư mục hiện tại :
        <img src=https://i.imgur.com/AAtg97z.png>
- **B10 :** Thực hiện copy file `ceph.client.admin.keyring` sang các node trong cluster. File này sẽ được copy vào thư mục `/etc/ceph/` trên các node :
    ```
    $ ceph-deploy admin ceph-1 ceph-2 ceph-3
    ```
- **B11 :** Đứng trên node `ceph_1` phân quyền cho file `/etc/ceph/ceph.client.admin.keyring` cho cả 03 node :
    ```
    $ ssh ceph_user@ceph-1 'sudo chmod +r /etc/ceph/ceph.client.admin.keyring'
    $ ssh ceph_user@ceph-2 'sudo chmod +r /etc/ceph/ceph.client.admin.keyring'
    $ ssh ceph_user@ceph-3 'sudo chmod +r /etc/ceph/ceph.client.admin.keyring'
    ```
### **2.5) Khai báo OSD cho các node**
- **B1 :** Show các disk có trên các node :
    ```
    $ ceph-deploy disk list ceph-1 ceph-2 ceph-3
    ```
- **B2 :** Xóa hết dữ liệu trên các disk :
    ```
    $ ceph-deploy disk zap ceph-1 /dev/vdb
    $ ceph-deploy disk zap ceph-1 /dev/vdc
    $ ceph-deploy disk zap ceph-1 /dev/vdd

    $ ceph-deploy disk zap ceph-2 /dev/vdb
    $ ceph-deploy disk zap ceph-2 /dev/vdc
    $ ceph-deploy disk zap ceph-2 /dev/vdd

    $ ceph-deploy disk zap ceph-3 /dev/vdb
    $ ceph-deploy disk zap ceph-3 /dev/vdc
    $ ceph-deploy disk zap ceph-3 /dev/vdd
    ```
- **B3 :** Tạo các **OSD** :
    ```
    $ ceph-deploy osd create ceph-1 --data /dev/vdb
    $ ceph-deploy osd create ceph-1 --data /dev/vdc
    $ ceph-deploy osd create ceph-1 --data /dev/vdd

    $ ceph-deploy osd create ceph-2 --data /dev/vdb
    $ ceph-deploy osd create ceph-2 --data /dev/vdc
    $ ceph-deploy osd create ceph-2 --data /dev/vdd

    $ ceph-deploy osd create ceph-3 --data /dev/vdb
    $ ceph-deploy osd create ceph-3 --data /dev/vdc
    $ ceph-deploy osd create ceph-3 --data /dev/vdd
    ```
- **B4 :** Kiểm tra trạng thái cụm **CEPH Cluster** :
    ```
    $ ceph -s
    ```
    <img src=https://i.imgur.com/wcAAOf6.png>
### **2.6) Cấu hình manager và dashboad cho Ceph cluster**
- **B1 :** Cài đặt các package hỗ trợ :
    ```
    $ sudo yum install -y python-jwt python-routes
    ```
- **B2 :** Cài đặt `ceph-mgr-dashboard` và `ceph-grafana-dashboard` :
    ```
    $ sudo rpm -Uvh https://download.ceph.com/rpm-nautilus/el7/noarch/ceph-grafana-dashboards-14.2.12-0.el7.noarch.rpm
    $ sudo rpm -Uvh https://download.ceph.com/rpm-nautilus/el7/noarch/ceph-mgr-dashboard-14.2.12-0.el7.noarch.rpm
    ```
- **B3 :** Kích hoạt `ceph-mgr` và `ceph-dashboard` :
    ```
    $ ceph-deploy mgr create ceph-1 ceph-2 ceph-3
    $ ceph mgr module enable dashboard --force
    ```
    - Kiểm tra lại các module đã được kích hoạt :
        ```
        $ ceph mgr module ls
        ```
        ```
        {
            "always_on_modules": [
                "balancer",
                "crash",
                "devicehealth",
                "orchestrator_cli",
                "progress",
                "rbd_support",
                "status",
                "volumes"
            ],
            "enabled_modules": [
                "dashboard",
                "iostat",
                "restful"
            ],
            ...
        }
        ```
- **B4 :** Tạo self-signed cert cho dashboard :
    ```
    $ sudo ceph dashboard create-self-signed-cert
    ```
- **B5 :** Tạo tài khoản cho `ceph-dashboard` :
    ```
    $ ceph dashboard ac-user-create cephadmin Password123 administrator
    ```
    ```json
    {"username": "cephadmin", "lastUpdate": 1604056616, "name": null, "roles": ["administrator"], "password": "$2b$12$SF4b28AE13RKtIS.bU1p1.p09EIhE/LkEKR0/tbZhzO9VXr1vQuXK", "email": null}
    ```
- **B6 :** Kiểm tra xem `ceph-dashboard` đã được cài thành công chưa đồng thời lấy URL truy cập :
    ```
    $ ceph mgr services
    ```
    ```json
    {
        "dashboard": "https://ceph-1:8443/"
    }
    ```
    > Đến đây đã hoàn tất việc cài đặt **Ceph cluster**. Có thể kiểm tra lại trạng thái cụm bằng lệnh `ceph -s` :
        
    <img src=https://i.imgur.com/FgZtbaN.png>
- **B7 :** Truy cập `ceph-dashboard` bằng địa chỉ IP của node `ceph-1`, đăng nhập với tài khoản vừa tạo :
    ```
    https://10.5.11.101:8443
    ```
    <img src=https://i.imgur.com/a8guMeN.png>

    - Giao diện dashboard :

    <img src=https://i.imgur.com/clt0Zvv.png>