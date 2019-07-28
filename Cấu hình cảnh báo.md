# Report
========================

## 1. Định nghĩa ngưỡng cảnh báo.
Mỗi event trên Wazuh Agent được đặt severity level mặc định là 1. Tất cả các event level này tăng lên sẽ tạo ra cảnh báo trên Wazuh Manager.

Cấu hình
Ngưỡng cảnh báo được cấu hình trên ossec.conf file sử dụng ` XML tag. 

    <ossec_config>
     <alerts>
      <log_alert_level>6</log_alert_level>
     </alerts>
    </ossec_config>
Cấu hình này sẽ đặt severity level nhỏ nhất để tạo cảnh báo, mà sẽ được lưu trữ tại alerts.log và/hoặc trên các file alerts.json

Khi bất kỳ giá trị nào bị thay đổi trong ossec.conf file, service cần phải restart để có tác dụng :

    systemctl restart wazuh-manager

## 2. Cấu hình syslog output
Wazuh có thể cấu hình để gửi các cảnh báo tới syslog như sau (trong file `osssec.conf), 

    <ossec_config>
     <syslog_output>
       <level>9</level>
         <server>192.168.1.241</server>
     </syslog_output>
     <syslog_output>
         <server>192.168.1.240</server>
      </syslog_output>
    </ossec_config>
Cấu hình trên sẽ gửi các cảnh báo tới 192.168.1.240 và nếu level cảnh báo >9, cũng sẽ gửi tới 192.168.1.241.

Sau khi cấu hình trong file ossec.conf, client-syslog phải được bật.

    /var/ossec/bin/ossec-control enable client-syslog
    systemctl restart wazuh-manager

## 3. Tự động tạo báo cáo
Bạn có thể cấu hình để tự động tạo báo cáo hàng ngày với option report trong osssec.conf. Chi tiết xem tại Report. Để cấu hình gửi email thông báo , tham khảo tại cấu hình email và cấu hình SMTP server

    <ossec_config>
      <reports>
        <category>syscheck</category>
        <title>Daily report: File changes</title>
      <email_to>example@test.com</email_to>
     </reports>
    </ossec_config>
Cấu hình trên sẽ gửi báo cáo hàng ngày của tất cả các syscheck alert tới email example@test.com

Các rule có thể được lọc bằng level, source, username, rule id... :

    <ossec_config>
      <reports>
        <level>10</level>
        <title>Daily report: Alerts with level higher than 12</title>
        <email_to>example@test.com</email_to>
      </reports>
    </ossec_config>
Cấu hình trên sẽ gửi báo cáo với tất cả các rule với level cảnh báo lớn 12


## 4. Cấu hình mail cảnh báo
Wazuh có thể gửi cảnh báo tới một hoặc nhiều email khi có rule được đặt hoặc với báo cáo event hàng ngày.

Mail ví dụ :

    From: Wazuh <you@example.com>         5:03 PM (2 minutes ago)
    to: me
    -----------------------------
    Wazuh Notification.
    2017 Mar 08 17:03:05
    Received From: localhost->/var/log/secure
    Rule: 5503 fired (level 5) -> "PAM: User login failed."
    Src IP: 192.168.1.37
    Portion of the log(s):
    Mar  8 17:03:04 localhost sshd[67231]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=192.168.1.37
    uid: 0
    euid: 0
    tty: ssh
    --END OF NOTIFICATION

### 4.1 Các option email thông thường
Để cấu hình Wazuh gửi email cảnh báo, các thiết lập email phải được cấu hình trong section global ở osssec.conf file :

    <ossec_config>
     <global>
        <email_notification>yes</email_notification>
        <email_to>me@test.com</email_to>
        <smtp_server>mail.test.com..</smtp_server>
        <email_from>wazuh@test.com</email_from>
     </global>
     ...
    </ossec_config>


Khi các giá trị trên được cấu hình, email_alert_level cần được đặt với level cảnh báo nhỏ nhất để tạo email. Mặc định, level được được là 7 :

    <ossec_config>
      <alerts>
      <email_alert_level>10</email_alert_level>
     </alerts>
      ...
    </ossec_config>
Ví dụ này sẽ đặt level nhỏ nhất tới 10. Chi tiết tham khảo tại alerts section

Sau khi alert_level được cấu hình. Wazuh cần restart :

    systemctl status wazuh-manager

## 4.2 Với các option của Granular email
Wazuh cũng sử dụng các cấu hình chi tiết cho các cảnh báo email. 

Chú ý : Level nhỏ nhất được cấu hình trong section alert sẽ ghi đè các cấu hình này. VD, nếu bạn cấu hình hệ thống gửi khi rule 526 được trigger, nhưng rule có level nhỏ được được chỉ định, cảnh báo sẽ không được gửi.

Cảnh báo email dựa trên level và agent

    <email_alerts>
      <email_to>you@example.com</email_to>
        <event_location>server1</event_location>
     <do_not_delay />
    </email_alerts>

Cấu hình này sẽ gửi email tới you@example.com khi rule trigger trên server1.

Trường event_location có thể được cấu hình để giám sát một log chỉ định, hostname hoặc IP.

Cảnh báo dựa trên rule ID

    <email_alerts>
      <email_to>you@example.com</email_to>
        <rule_id>515, 516</rule_id>
      <do_not_delay /> 
     </email_alerts>
Cấu hình này sẽ gửi khi rule 515 hoặc 516 được trigger với agent bất kỳ

Cảnh báo dựa trên group

    <email_alerts>
       <email_to>you@example.com</email_to>
      <group>pci_dss_10.6.1</group>
    </email_alerts>
Cấu hình này sẽ gửi cảnh báo khi bất kì rule nào là 1 phần của group `pci_dss_10.6.1` được trigger trên bất kỳ Wazuh device nào được giám sát.

Bắt buộc gửi cảnh bảo bới email
Có thể buộc gửi 1 email cảnh báo trên rule đã khai báo bên dưới với level cảnh báo nhỏ nhất. Để làm được việc này, bạn cần sử dụng một trong các option bên dưới đây
 
     alert_by_email : luôn luôn cảnh báo bằng email
     no_email_alert: không bao giờ cảnh báo bằng email
     no_log : không log lại alert này

    <rule id="502" level="3">
    <if_sid>500</if_sid>
    <options>alert_by_email</options>
     <match>Ossec started</match>
    <description>Ossec server started.</description>
    </rule>

Cấu hình này sẽ gửi email mỗi khi rule 502 được trigger tương ứng với level nhỏ nhất được đặt.

## 5. Cấu hình SMTP server
Có thể sử dụng Postfix để thực hiện tính năng xác thực email. Các bước cài đặt và cấu hình như sau :

Cài đặt postfix

    apt-get install postfix mailutils libsasl2-2 ca-certificates libsasl2-modules -y

 Cấu hình Postfix trong file /etc/postfix/main.cf, thêm các dòng sau vào cuối file :

    relayhost = [smtp.gmail.com]:587
    smtp_sasl_auth_enable = yes
    smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
    smtp_sasl_security_options = noanonymous
    smtp_tls_CAfile = /etc/ssl/certs/thawte_Primary_Root_CA.pem
    smtp_use_tls = yes

 Cấu hình email và password

    echo [smtp.gmail.com]:587 USERNAME@gmail.com:PASSWORD > /etc/postfix/sasl_passwd
    postmap /etc/postfix/sasl_passwd
    chmod 400 /etc/postfix/sasl_passwd

Bảo mật DB password

    chown root:root /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db
    chmod 0600 /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db

Restart Postfix

    systemctl reload postfix

Test cấu hình

    echo "Test mail from postfix" | mail -s "Test Postfix" you@example.com

Cấu hình Wazuh tại /var/ossec/etc/ossec.conf :

    <global>
      <jsonout_output>yes</jsonout_output>
      <alerts_log>yes</alerts_log>
      <logall>no</logall>
      <logall_json>no</logall_json>
      <email_notification>yes</email_notification>
      <smtp_server>smtp.example.wazuh.com</smtp_server>
      <email_from>ossecm@example.wazuh.com</email_from>
      <email_to>recipient@example.wazuh.com</email_to>
      <email_maxperhour>12</email_maxperhour>
    </global>

Restart wazuh manager

    systemctl restart wazuh-manager
