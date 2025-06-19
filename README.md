# Домашнее задание по лекции "ELK"

### Задание 1
Cisco config ![alt text](img/1.png)

Cisco pkt (https://github.com/drumlast/sflt-39-dr-keepalived/blob/main/pkt/hsrp_advanced_a_ivanov.pkt)

### Задание 2
Конфиг keepalived MASTER

```
global_defs {
    script_user root
    enable_script_security
}

vrrp_script check_web {
    script "/etc/keepalived/check_web.sh"
    interval 3
    weight -10
}

vrrp_instance VI_1 {
    state MASTER
    interface ens33
    virtual_router_id 155
    priority 100
    advert_int 3

    authentication {
        auth_type PASS
        auth_pass 1234
    }

    virtual_ipaddress {
        192.168.245.155/24
    }

    track_script {
        check_web
    }
}
```

Конфиг keepalived BACKUP

```
global_defs {
    script_user root
    enable_script_security
}

vrrp_script check_web {
    script "/etc/keepalived/check_web.sh"
    interval 3
    weight -10
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens33
    virtual_router_id 155
    priority 90
    advert_int 3

    authentication {
        auth_type PASS
        auth_pass 1234
    }

    virtual_ipaddress {
        192.168.245.155/24
    }

    track_script {
        check_web
    }
}
```

bash-скрипт проверки веб-сервера

```
#!/bin/bash

PORT=80
URL="/var/www/html/index.html"

# Check if port enabled

nc -z 127.0.0.1 $PORT
if [ $? -ne 0 ]; then
    exit 1
fi

# Check if file index.html exists

if [ ! -f "$URL" ]; then
    exit 1
fi

exit 0
```

Статус keepalived до:
![alt text](img/2-1_vm1.png)
![alt text](img/2-2_vm2.png)
![alt text](img/2-3_before_vm1.png)
![alt text](img/2-4_before_vm2.png)

Статус keepalived после:
![alt text](img/2-5_after_vm1.png)
![alt text](img/2-6_after_vm2.png)