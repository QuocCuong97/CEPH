# Cài đặt RadosGW
### **1) Cài đặt RadosGW**
- **B1 :** Đứng trong thư mục `ceph_cluster` của user `ceph_user` cài đặt **RadosGW** :
    ```
    $ ceph-deploy install --rgw ceph-1 ceph-2 ceph-3
    ```
- **B2 :** Deploy **RadosGW** :
    ```
    $ ceph-deploy rgw create ceph-1 ceph-2 ceph-3
    ```
    ```
    ...
    [ceph_deploy.rgw][INFO  ] The Ceph Object Gateway (RGW) is now running on host ceph-1 and default port 7480
    ...
    [ceph_deploy.rgw][INFO  ] The Ceph Object Gateway (RGW) is now running on host ceph-2 and default port 7480
    ...
    [ceph_deploy.rgw][INFO  ] The Ceph Object Gateway (RGW) is now running on host ceph-3 and default port 7480
    ```
- **B3 :** Kiểm tra lại trạng thái của **Ceph cluster** :
    ```
    $ ceph -s
    ```
    <img src=https://i.imgur.com/B9gW6gU.png>
### **2) Cấu hình RadosGW client**
- Chỉnh sửa file cấu hình `ceph` trên cả 3 node :
    ```
    # vi /etc/ceph/ceph.conf
    ```
    - Thêm đoạn sau vào cuối file (chú ý điều chỉnh host cho phù hợp):
    ```
    [client.rgw.ceph-1]
    host = ceph-1
    debug_rgw = 5
    rgw frontends = beast port=80
    ```