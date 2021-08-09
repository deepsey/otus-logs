# Домашнее задание по теме "Сбор и анализ логов"

## 1. Поднимаем в Vagrant два сервера: otus-web (192.168.100.64) и otus-log (192.168.100.61).

## 2. Для построения системы централизованного хранения логов будем использовать journald.

## 3. На центральном сервере otus-log:

#### 3.1. Устанавливаем пакет systemd-journal-gateway

        # yum install systemd-journal-gateway
        
#### 3.2. Настраиваем пассивный режим работы демона systemd-journal-remote
        
        # mkdir -p /var/log/journal/remote
        # chown systemd-journal-remote:systemd-journal-remote /var/log/journal/remote
      
#### 3.3. Редактируем конфиг демона
 
        # vi /lib/systemd/system/systemd-journal-remote.service
        
========================================================================================

    This file is part of systemd.  
    systemd is free software; you can redistribute it and/or modify it  
    under the terms of the GNU Lesser General Public License as published by  
    the Free Software Foundation; either version 2.1 of the License, or  
    (at your option) any later version.  
========================================================================================  

    [Unit]  
    Description=Journal Remote Sink Service  
    Requires=systemd-journal-remote.socket  
  
    [Service]  
    ExecStart=/usr/lib/systemd/systemd-journal-remote \  
          --listen-http=-3 \  
          --output=/var/log/journal/remote/  
    User=systemd-journal-remote  
    Group=systemd-journal-remote  
    PrivateTmp=yes  
    PrivateDevices=yes  
    PrivateNetwork=yes  
    WatchdogSec=10min  
  
    [Install]
    Also=systemd-journal-remote.socket  
=========================================================================================  
  
#### 3.4. Смотрим конфиг сокета:
    
    vi /lib/systemd/system/systemd-journal-remote.socket
        
=========================================================================================

    # This file is part of systemd.  
      
    #  systemd is free software; you can redistribute it and/or modify it  
    #  under the terms of the GNU Lesser General Public License as published by  
    #  the Free Software Foundation; either version 2.1 of the License, or  
    #  (at your option) any later version.  

    [Unit]  
    Description=Journal Remote Sink Socket  
  
    [Socket]  
    ListenStream=19532  
  
    [Install]  
    WantedBy=sockets.target  
========================================================================================  

#### 3.5. Перезагружаем конфиги демонов и рестартим сервис systemd-journal-remote
    
        # systemctl daemon-reload
        # systemctl restart systemd-journal-remote
        
## 4. На удаленном сервере otus-web:

#### 4.1. Устанавливаем пакет systemd-journal-gateway  
 
        # yum install systemd-journal-gateway  

#### 4.2. Редактируем конфиг /etc/systemd/journal-upload.conf, прописывая в него адрес центрального сервера 192.168.100.61  

       # vi /etc/systemd/journal-upload.conf  

========================================================================================  

    [Upload]  
    URL=http://192.168.100.61:19532  
    # ServerKeyFile=/etc/ssl/private/journal-upload.pem  
    # ServerCertificateFile=/etc/ssl/certs/journal-upload.pem  
    # TrustedCertificateFile=/etc/ssl/ca/trusted.pem  
=======================================================================================  
 
#### 4.3. Стартуем сервис  
    
        # systemctl start systemd-journal-upload  
  
## 5. Проверяем содержимое директории на центральном сервере  
  
  
       [root@otus-log ~]# ls /var/log/journal/remote  
       remote-192.168.100.64.journal  
  
  
## 6. Смотрим логи на центральном сервере   


       [root@otus-log ~]# journalctl --file /var/log/journal/remote/remote-192.168.100.64.journal  
  
  
  
       май 04 09:13:14 otus-web systemd-journal[222]: Runtime journal is using 4.0M (max allowed 11.7M, trying to leave 17.6M free of 113.4M available → current limit 11.7M).  
       май 04 09:13:14 otus-web systemd-journald[91]: Received SIGTERM from PID 1 (systemd).  
       май 04 09:13:14 otus-web kernel: random: crng init done  
       май 04 09:13:14 otus-web kernel: type=1404 audit(1620119592.544:2): enforcing=1 old_enforcing=0 auid=4294967295 ses=4294967295  
       май 04 09:13:14 otus-web kernel: SELinux: 2048 avtab hash slots, 112811 rules.  
       май 04 09:13:14 otus-web kernel: SELinux: 2048 avtab hash slots, 112811 rules.  
       май 04 09:13:14 otus-web kernel: SELinux:  8 users, 14 roles, 5052 types, 316 bools, 1 sens, 1024 cats  
       май 04 09:13:14 otus-web kernel: SELinux:  130 classes, 112811 rules  
       май 04 09:13:14 otus-web kernel: SELinux:  Completing initialization.  
       май 04 09:13:14 otus-web kernel: SELinux:  Setting up existing superblocks.  
       май 04 09:13:14 otus-web kernel: type=1403 audit(1620119592.800:3): policy loaded auid=4294967295 ses=4294967295  
       май 04 09:13:14 otus-web systemd[1]: Successfully loaded SELinux policy in 281.118ms.  
       май 04 09:13:14 otus-web kernel: ip_tables: (C) 2000-2006 Netfilter Core Team  
       май 04 09:13:14 otus-web systemd[1]: Inserted module 'ip_tables'  
       май 04 09:13:14 otus-web systemd[1]: Relabelled /dev, /run and /sys/fs/cgroup in 30.793ms.  
       май 04 09:13:14 otus-web kernel: tsc: Refined TSC clocksource calibration: 3008.791 MHz  
       май 04 09:13:14 otus-web systemd-journal[222]: Journal started  
       май 04 09:13:13 otus-web systemd[1]: systemd 219 running in system mode. (+PAM +AUDIT +SELINUX +IMA -APPARMOR +SMACK +SYSVINIT +UTMP +LIBCRYPTSETUP +GCRYPT +GNUTLS +ACL +XZ +LZ4 -SECCOMP +BLKID +ELFUTILS +KMOD   
       май 04 09:13:13 otus-web systemd[1]: Detected virtualization kvm.  
       май 04 09:13:13 otus-web systemd[1]: Detected architecture x86-64.  
       май 04 09:13:13 otus-web systemd[1]: Set hostname to <otus-web>.  
       май 04 09:13:14 otus-web systemd[1]: Starting Flush Journal to Persistent Storage...  
       май 04 09:13:14 otus-web systemd[1]: Started Read and set NIS domainname from /etc/sysconfig/network.  
       май 04 09:13:14 otus-web systemd[1]: Activated swap /swapfile.  
       май 04 09:13:14 otus-web systemd[1]: Reached target Swap.  
       май 04 09:13:14 otus-web kernel: Adding 2097148k swap on /swapfile.  Priority:-2 extents:1 across:2097148k FS  
       май 04 09:13:14 otus-web systemd[1]: Started udev Coldplug all Devices.  
       май 04 09:13:15 otus-web systemd[1]: Started Flush Journal to Persistent Storage.  
       май 04 09:13:15 otus-web systemd[1]: Started Create Static Device Nodes in /dev.  
       май 04 09:13:15 otus-web systemd[1]: Reached target Local File Systems (Pre).  
       май 04 09:13:15 otus-web systemd[1]: Starting udev Kernel Device Manager...  

  
## 7. Устанавливаем и запускаем nginx на otus-web

    # yum install nginx  
    # systemctl start nginx  
   
## 8. Играемся с конфигурацией nginx, смотрим логи на центральном сервере:

    # journalctl -u nginx --file /var/log/journal/remote/remote-192.168.100.64.journal  
  
    май 18 10:22:19 otus-web nginx[32744]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok  
    май 18 10:22:19 otus-web nginx[32744]: nginx: configuration file /etc/nginx/nginx.conf test is successful  
    май 18 10:22:19 otus-web systemd[1]: Failed to parse PID from file /run/nginx.pid: Invalid argument  
    май 18 10:22:19 otus-web systemd[1]: Started The nginx HTTP and reverse proxy server.  
    май 18 10:24:23 otus-web systemd[1]: Stopping The nginx HTTP and reverse proxy server...  
    май 18 10:24:23 otus-web systemd[1]: Stopped The nginx HTTP and reverse proxy server.  
    май 18 10:24:23 otus-web systemd[1]: Starting The nginx HTTP and reverse proxy server...  
    май 18 10:24:25 otus-web nginx[313]: nginx: [emerg] host not found in "default_server" of the "listen" directive in /etc/nginx/nginx.conf:39  
    май 18 10:24:25 otus-web nginx[313]: nginx: configuration file /etc/nginx/nginx.conf test failed  
    май 18 10:24:25 otus-web systemd[1]: nginx.service: control process exited, code=exited status=1  
    май 18 10:24:25 otus-web systemd[1]: Failed to start The nginx HTTP and reverse proxy server.  
    май 18 10:24:25 otus-web systemd[1]: Unit nginx.service entered failed state.  
    май 18 10:24:25 otus-web systemd[1]: nginx.service failed.  
    май 18 10:25:26 otus-web systemd[1]: Starting The nginx HTTP and reverse proxy server...  
    май 18 10:25:26 otus-web nginx[325]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok  
    май 18 10:25:26 otus-web nginx[325]: nginx: configuration file /etc/nginx/nginx.conf test is successful  
    май 18 10:25:27 otus-web systemd[1]: Failed to parse PID from file /run/nginx.pid: Invalid argument  
    май 18 10:25:27 otus-web systemd[1]: Started The nginx HTTP and reverse proxy server.  
  
  

## 9. Настраиваем хранение логов аудита на центральном сервере.

#### 9.1. На otus-web:  
  
   9.1.1. Устанавливаем пакет audispd-plugins  
 
        # yum install audispd-plugins  
 
   9.1.2. Настраиваем входящий в пакет плагин audisp-remote  
 
       # vi /etc/audisp/plugins.d/au-remote.conf
       
===================================================================================

    # This file controls the audispd data path to the  
    # remote event logger. This plugin will send events to  
    # a remote machine (Central Logger).  
  
    active = yes  
    direction = out  
    path = /sbin/audisp-remote  
    type = always  
    #args =  
    format = string  
===================================================================================  
  
В этом файле меняем значение параметра active на yes, чтобы активировать отправку логов 
аудита в удаленное хранилище.  
  
   9.1.3. В файле /etc/audisp/audisp-remote.conf изменим значение параметра remote_server  
        на адрес центрального сервера 192.168.100.61  
          
        # vi /etc/audisp/audisp-remote.conf
          
===================================================================================   

    # This file controls the configuration of the audit remote
    # logging subsystem, audisp-remote.
    #  
  
    remote_server = 192.168.100.61
    port = 60

===================================================================================  
  
#### 9.2. На центральном сервере:  
      
  9.2.1. Чтобы принимать логи аудита с удаленных хостов, в файле /etc/audit/auditd.conf  
           устанавливаем следующие параметры:  
             
           
    tcp_listen_port = 60  
    tcp_listen_queue = 5  
    tcp_max_per_addr = 1  
    ##tcp_client_ports = 1024-65535  
    tcp_client_max_idle = 0  
  
  9.2.2. Перезапускаем auditd
    
         # service auditd condrestart
        
 9.3. Проходим процедуру аутентификации на otus-web, на otus-log смотрим логи аудита:
    
    [root@otus-log ~]# aureport -au
    
=======================================================================================    

    Authentication Report  
    ============================================  
    # date time acct host term exe success event  
    ============================================  
    1. 04.05.2021 09:04:02 vagrant 10.0.2.2 ? /usr/sbin/sshd yes 42  
    2. 04.05.2021 09:04:02 vagrant 10.0.2.2 ? /usr/sbin/sshd yes 43  
    3. 04.05.2021 09:04:02 vagrant 10.0.2.2 ssh /usr/sbin/sshd yes 46  
    4. 04.05.2021 09:04:28 vagrant 10.0.2.2 ? /usr/sbin/sshd yes 517  
    5. 04.05.2021 09:04:28 vagrant 10.0.2.2 ? /usr/sbin/sshd yes 518  
    6. 04.05.2021 09:04:28 vagrant 10.0.2.2 ssh /usr/sbin/sshd yes 521  
    7. 04.05.2021 09:14:29 vagrant 10.0.2.2 ? /usr/sbin/sshd yes 742  
    8. 04.05.2021 09:14:29 vagrant 10.0.2.2 ? /usr/sbin/sshd yes 743  
    9. 04.05.2021 09:14:29 vagrant 10.0.2.2 ssh /usr/sbin/sshd yes 746  
    10. 04.05.2021 09:16:03 vagrant 10.0.2.2 ? /usr/sbin/sshd yes 38  
    11. 04.05.2021 09:16:03 vagrant 10.0.2.2 ? /usr/sbin/sshd yes 39  
    12. 04.05.2021 09:16:03 vagrant 10.0.2.2 ssh /usr/sbin/sshd yes 42  
    13. 04.05.2021 09:29:25 vagrant 10.0.2.2 ? /usr/sbin/sshd yes 1119  
    14. 04.05.2021 09:29:25 vagrant 10.0.2.2 ? /usr/sbin/sshd yes 1120  
    15. 04.05.2021 09:29:25 vagrant 10.0.2.2 ssh /usr/sbin/sshd yes 1123  
    16. 14.05.2021 06:55:55 vagrant 10.0.2.2 ? /usr/sbin/sshd yes 2933  
    17. 14.05.2021 06:55:55 vagrant 10.0.2.2 ? /usr/sbin/sshd yes 2934  
    18. 14.05.2021 06:55:55 vagrant 10.0.2.2 ssh /usr/sbin/sshd yes 2937  
    19. 18.05.2021 10:14:49 ? otus-web tty1 /usr/bin/login no 3255  
    20. 18.05.2021 10:17:39 vagrant otus-web tty1 /usr/bin/login yes 3267  
    21. 18.05.2021 10:21:35 ? otus-web tty1 /usr/bin/login no 3298  
    22. 18.05.2021 10:21:53 vagrant otus-web tty1 /usr/bin/login yes 3300  
