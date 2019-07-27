# Tìm hiểu về Wazuh
========================


## I. Tổng quan
### 1.Giới thiệu về Wazuh
Wazuh là 1 project mã nguồn dùng cho việc bảo vệ an ninh. Được xây dựng từ các thành phần : OSSEC HIDS, OpenSCAP và Elastic Stack.

![wazuh-00](https://github.com/builonghaikma198/ghichep-SOC/blob/master/images/wazuh-00.png?raw=true)


* OSSEC HIDS : host-based Intrusion Detection System (HIDS) được dùng cho việc phát hiện xâm nhập, hiển thị và giám stas. Nó dựa vào 1 multi-platform agent cho việc đẩy dữ liệu hệ thống (log message, file hash và phát hiện bất thường) tới 1 máy quản lý trung tâm, nơi sẽ phân tích và xử lý, dựa trên các cảnh báo an ninh. Các agent truyền event data event data tới máy quản lý trung tâm thông qua kênh được bảo mật và xác thực. OSSEC HIDS cung cấp syslog server trung tâm và hệ thống giám sát không cần agent, cung cấp việc giám sát tới các event và thay đổi trên các thiết bị không cài được agent như firewall, switch, router, access point, thiết bị mạng....

* OpenSCAP OpenSCAP là 1 OVAL (Open Vulnerability Assesment Language) và XCCDF (Extensible Configuration Checklist Description Format) được dùng để kiểm tra cấu hình hệ thống và phát hiện các ứng dụng dễ bị tấn công. Nó được biết đến như là một công cụ được thiết kế để kiểm tra việc tuân thủ an ninh của hệ thống sử dụng các tiêu chuẩn an ninh dùng cho môi trường doanh nghiệp.

* ELK Stack Sử dụng cho việc thu thập, phân tihcs, index, store, search và hiển thị dữ liệu log.
### 2. Các thành phần
#### 2.1 Wazuh agent
Chạy trên : Windows, Linux, Solaris, BSD hoặc MAC OS. Dùng thu thập các dạng khác nhau của dữ liệu hệ thống và ứng dụng. Dữ liệu được chuyển tới Wazuh server thông qua 1 kênh được mã hóa và xác thực. Để thiết lập kênh này, 1 quá trình đăng ký bao gồm pre-shared key duy nhất được thiết lập.

Các agent có thể dùng để giám sát server vật lý, máy ảo, cloud instance (AWS, Azure hoặc Google cloud). Các các cài đặt pre-compile agent có sẵn cho các OS : Linux, AIX, Solaris, Windows và Darwin (Mac OS X).

Trên các OS Unix-based, agent chạy trên multiple process, các process này liên lạc với nhau thông qua local Unix domain socket. 1 trong các process này phụ trách việc liên lạc và gửi dữ liệu tới Wazuh server. Trên Windows, chỉ có 1 agent process chạy trên multiple task sử dụng mutexes.

Các agent task hoặc process khác nhau được dùng để giám sát hệ thống theo các cách khác nhau (giám sát sự thay đổi về file, đọc log, quét các thay đổi hệ thống).

Sơ đồ sau thể hiện các internal task và process diễn ra trên các agent level.
![alt](file://media/138932080.png)


Tất cả các process agent có mục tiêu và thiết lập khác nhau.
* **Rootcheck** : Thực hiện các task liên quan đến phát hiện về `Rootkits`, `malware` và các bất thường của hệ thống. Nó chạy 1 số công cụ kiểm tra an ninh cơ bản dựa vào các file cấu hình hệ thống.

* **Log Collector**: Dùng để đọc và thu thập các log message, bao gồm các các file flat log như Windows event log và thậm chí là Windows Event Channel. Nó cũng được cấu hình để chạy định kỳ và bắt 1 số output của các câu lệnh cụ thể.

* **Syscheck** : Process này thực hiện file integrity monitoring (FIM) (Giám sát tính toàn vẹn của file). Nó cũng có thể giám sát registry key trên Windows. Nó sẽ bắt các thay đổi về nội dung file, quyền và các thuộc tính khác, cũng như phát hiện việc tạo và xóa file.

* **Agent Daemon** : Process nhận dữ liệu được tạo hoặc được thu thập bởi tất cả các thành phần agent khác. Nó nén, mã hóa và phân phối dữ liệu tới server thông qua kênh được xác thực. Process này chạy trên "chroot" enviroment được cô lập, có nghĩa rằng nó sẽ hạn chế truy cập tới các hệ thống được giám sát. Điều này cải thiện được an toàn cho agent vì process đó là process duy nhất kết nối tới mạng.

Chú giải :
 
* Rootkits : Phần mềm hoặc công cụ phần mềm che giấu sự tồn tại của 1 phần mềm khác, thường là virus xâm nhập vào hệ thống.

* Malware : Mọi loại mã gây hại trên máy tính người dùng : spyware, trojan, virus...

#### 2.2 Wazuh server
Thành phần server phụ trách việc phân tích dữ liệu nhận từ agent, tạo các ngưỡng cảnh báo khi 1 event ánh xạ với rule (phát hiện xâm nhập, thay đổi file, cấu hình không tương thích với policy, rootfit...)

![wazuh-02](file://media/1582778743.png)


Server thông thường chạy các thành phần agent với mục tiêu giám sát chính nó. Một số thành phần server chính là :

* **Registration service** : Được dùng để register agent mới được việc cung cấp và phân phối các key xác thực pre-shared, các key này là độc nhất với mỗi agent. Process này chạy như 1 network service và hỗ trợ việc xác thực qua TLS/SSL với 1 fixed password.

* **Remote daemon service** : Service này nhận dữ liệu từ agent. Nó sử dụng pre-shared key để xác thực định danh của mỗi agent và mã hóa giao tiếp với chúng.

* **Analysis daemon** : Process này thực hiện việc phân tích dữ liệu. Nó sử dụng các bộ giải mã để nhận dạng thông tin được xử lý (các Windows event, SSHD logs...) và sau đó giải nén các yếu tố dữ liệu thích hợp từ log message (source ip, event id, user...) Sau đó, bằng cách sử dụng các rule được định nghĩa bằng cách pattern đặc biệt trên bộ giải mã, nó sẽ tạo các ngưỡng cảnh báo thậm chí ra lệnh để thực hiện các biện pháp đối phó như chặn IP trên firewall.

* **RESTful API** : Cung cấp interface để quản lý và giám sát cấ hình và trạng thái triển khai của các agent. Nó cũng được dùng bởi Wazuh web interface (Kibana).

Wazuh tích hợp với Elastic stack để cung cấp các log message đã được giải mã và đánh index bởi Elasticsearch, cũng như là 1 web console real-time cho việc cảnh báo và phân tích log. Wazuh web interface (chạy trên Kibana) có thể dùng để quản lý và giám sát hạ tầng Wazuh

Một Elasticsearch index là một tập hợp các document có một chút các đặc trưng tương tự nhau (như các trường chung hoặc các yêu cầu về data retention được chia sẻ). Wazuh sử dụng 3 index khác nhau, được tạo hàng ngày và lưu trữ các dạng event khác nhau :

* **Wazuh-alert** : Index cho các cảnh báo được sinh ra bởi Wazun server mỗi khi một event ứng với rule tạo ra.
* **Wazuh-events** : Index cho tất cả các event (archive data) được nhận từ các agent, bất kể có ứng với rule hay không.
* **Wazuh-monitoring** : Index cho dữ liệu liên quan đến trạng thái agent. Nó được dùng bởi web interface cho việc hiển thị agent đã hoặc đang "Active", "Disconnect" hoặc "Never connected".

Với các index trên, document là các cảnh báo, archived event hoặc status event riêng lẻ.

Một Elasticsearcg index được chia tới 1 hoặc nhiều shard, và mỗi shard có thể có 1 hoặc nhiều replica. Mỗi primary và replica shard là 1 Lucene index đơn lẻ. Vì vậy 1 Elasticsearch index được tạo bởi nhiều Lucene index. Khi 1 tìm kiếm chạy trên 1 Elasticsearch index, search đó được xử lý trên các shard song song, và kết quả được merge lại. Việc chia nhỏ các Elasticsearch tới nhiều shard và replica cũng được dùng với Elasticsearch cluster với mục tiêu là mở rộng việc tìm kiếm và HA. Một Elasticsearch cluster single-node thường chỉ có 1 shard mỗi index và không có replica.

## II. Cài đặt Wazuh
### 1. Cài đặt Wazuh manager
Bước đầu tiên để thiết lập Wazuh là thêm kho lưu trữ Wazuh vào máy chủ của bạn.

 1.1 Để thực hiện quy trình này, các gói phát hành ,curl,apt-transport-https và lsb-release phải được cài đặt trên hệ thống của bạn.Nếu bạn chưa cài thì hãy cài chúng theo lệnh bên dưới

    apt-get update
    apt-get install curl apt-transport-https lsb-release gnupg2

1.2 Cài GPG key:

 `curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | apt-key add -`

1.3 Thêm repo

`echo "deb https://packages.wazuh.com/3.x/apt/ stable main" | tee -a /etc/apt/sources.list.d/wazuh.list`

1.4 Cập  nhật thông tin gói

`apt-get update`

### 2. Cài đặt Wazuh Manager
Trên terminal của bạn

`apt-get install wazuh-manager`

Khi hoàn thành cài đặt, bạn có thể kiểm tra dịch vụ với :

`systemctl status wazuh-manager`

hoặc

`service wazuh-manager status`

### 3. Cài đặt Wazuh API

NodeJS> = 4.6.1 là bắt buộc để chạy API Wazuh. Nếu bạn chưa cài đặt NodeJS hoặc phiên bản của bạn cũ hơn 4.6.1, Tôi khuyên bạn nên thêm repo NodeJS bằng lệnh sau:

    curl -sL https://deb.nodesource.com/setup_8.x | bash -
sau đó cài NodeJS:

     apt-get install nodejs
Cài đặt Wazuh API:

     apt-get install wazuh-api

KIểm tra trạng thái hoạt động :

    systemctl status wazuh-api
hoặc

    service wazuh-api status

#### Mở port cho phép Wazuh agent kết nối:
    firewall-cmd --add-port=1515/tcp
    firewall-cmd --add-port=1515/tcp --permanent
    firewall-cmd --add-port=55000/tcp
    firewall-cmd --add-port=55000/tcp --permanent
    firewall-cmd --add-port=1514/udp
    firewall-cmd --add-port=1514/udp --permanent
    firewall-cmd --add-port=514/tcp
    firewall-cmd --add-port=514/tcp --permanent
    firewall-cmd --add-port=514/udp
    firewall-cmd --add-port=514/udp --permanent

### 4. Cài đặt filebeat

Filebeat là công cụ trên máy chủ Wazuh chuyển tiếp cảnh báo và các sự kiện được lưu trữ một cách an toàn cho Elaticsearch.

#### 4.1 Cài đặt Elastic repo và GPG key :

    apt-get install curl apt-transport-https
    curl -s https://artifacts.elastic.co/GPG-KEY-elasticsearch | apt-key add -
    echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | tee /etc/apt/sources.list.d/elastic-7.x.list
    apt-get update

#### 4.2 Cài đặt Filebeat:
     apt-get install filebeat=7.2.0

#### 4.3  Tải tệp cấu hình Filebeat từ Wazuh repository. Đây là cấu hình được thiết lập sẵn để chuyển tiếp cảnh báo Wazuh tới Elaticsearch:
     curl -so /etc/filebeat/filebeat.yml https://raw.githubusercontent.com/wazuh/wazuh/v3.9.3/extensions/filebeat/7.x/filebeat.yml
#### 4.4 Tải cảnh bảo cho Elastichsearch:
     curl -so /etc/filebeat/wazuh-template.json https://raw.githubusercontent.com/wazuh/wazuh/v3.9.3/extensions/elasticsearch/7.x/wazuh-template.json
#### 4.5 Chỉnh sửa tệp `/etc/filebeat/filebeat.yml` và thay thế `YOUR_ELASTIC_SERVER_IP` với địa chỉ hoặc hostname của Elasticsearch server. Ví dụ : 
     output.elasticsearch:
      hosts: ['http://YOUR_ELASTIC_SERVER_IP:9200']
      indices:
        index: 'wazuh-alerts-3.x-%{+yyyy.MM.dd}'

#### 4.6 Kích hoạt và khởi động Filebeat server: 
     systemctl daemon-reload
     systemctl enable filebeat.service
     systemctl start filebeat.service

### 5. Cài đặt Elasticseach

#### 5.1 Chuẩn bi:

Cài đặt Elastic repo và GPG key : 

     apt-get install curl apt-transport-https
     curl -s https://artifacts.elastic.co/GPG-KEY-elasticsearch | apt-key add -
     echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | tee /etc/apt/sources.list.d/elastic-7.x.list
     apt-get update

#### 5.2 Cài Elasticsearch

    apt-get install elasticsearch=7.2.0
#### Thêm ruler firewall
     firewall-cmd --add-port=9200/tcp
     firewall-cmd --add-port=9200/tcp --permanent

#### 5.3 Kích hoạt và khơi động Elastichsearch:
    systemctl daemon-reload
    systemctl enable elasticsearch.service
    systemctl start elasticsearch.service

#### 5.4 Kiểm tra dịch vụ Elastichsearch
     curl -X GET http://localhost:9200

#### 5.5 Load Wazuh Elasticsearch Template:

    curl https://raw.githubusercontent.com/wazuh/wazuh/v3.9.3/extensions/elasticsearch/7.x/wazuh-template.json | curl -X PUT "http://localhost:9200/_template/wazuh" -H 'Content-Type: application/json' -d @-
    {"acknowledged":true}

### 6. Kibana

Kibana là một giao diện web linh hoạt và trực quan để khai thác và trực quan hóa các sự kiện và tài liệu lưu trữ được lưu trữ trong Elaticsearch

#### 6.1 Cài đặt Kibana

    apt-get install kibana=7.2.0

#### 6.2 Cài đặt Wazuh app plugin cho Kibana:

    sudo -u kibana /usr/share/kibana/bin/kibana-plugin install https://packages.wazuh.com/wazuhapp/wazuhapp-3.9.3_7.2.0.zip

#### 6.3 Kich hoạt và khơi động Kibana :

    systemctl daemon-reload
    systemctl enable kibana.service
    systemctl start kibana.service

#### 6.4 Cho phép truy cập Kibana web interface (port 5601)
    firewall-cmd --add-port=5601/tcp
    firewall-cmd --add-port=5601/tcp --permanent

### 7. Logstash
#### 7.1 Thêm file repo
    apt-get install curl apt-transport-https
    curl -s https://artifacts.elastic.co/GPG-KEY-elasticsearch | apt-key add -
    echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | tee /etc/apt/sources.list.d/elastic-7.x.list
    apt-get update

#### 7.2 Cài đặt Logstash

    apt-get install logstash=1:7.2.0-1
    systemctl daemon-reload 
    systemctl enable logstash

#### 7.3 Download Wazuh config và template file cho Logstash:

    curl -so /etc/logstash/conf.d/01-wazuh.conf https://raw.githubusercontent.com/wazuh/wazuh/v3.9.3/extensions/logstash/7.x/01-wazuh-remote.conf

#### 7.4 Restart Logstash
     systemctl restart logstash

#### 7.5 Cấu hình FIlebeat, thay đổi events từ Elastichsearch sang Logstash

Disable the Elasticsearch output in `/etc/filebeat/filebeat.yml.`

    #output.elasticsearch:
    #hosts: ['http://YOUR_ELASTIC_SERVER_IP:9200']
     #pipeline: geoip
     #indices:
      -#index: 'wazuh-alerts-3.x-%{+yyyy.MM.dd}'

Add Logstash output in `/etc/filebeat/filebeat.yml.`

    output.logstash.hosts: ["YOUR_LOGSTASH_SERVER_IP:5000"]

#### 7.6 Restart Filebeat:

     systemctl restart filebeat

#### 7.7  Kiểm tra xem Logstash có thể truy cập được từ Filebeat không.
     filebeat test output
Ví dụ :

    logstash: 172.16.1.2:5000...
    connection...
     parse host... OK
     dns lookup... OK
     addresses: 172.16.1.2
    dial up... OK
    TLS... WARN secure connection disabled
    talk to server... OK



