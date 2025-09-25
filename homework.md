Задание 1:
1. Создать два сервера, установить на них PostgreSQL и подключить их к
одной сети.
2. Настроить репликацию между серверами, чтобы изменения, вносимые
на одном сервере, автоматически реплицировались на другой.
3. Настроить отказоустойчивость, используя репликацию и механизм
автоматического переключения между серверами.
4. Проверить работу отказоустойчивого решения, симулируя отказ одного
из серверов и убедившись, что второй сервер продолжает работу и все
данные сохранены.

ubuntu-serv01 - master 192.168.56.3, ubuntu-serv02 - slave 192.168.56.4, ubuntu-etcd - etcd 192.168.56.2

Устанавливаем PostgreSQL на наши сервера:

  <img width="626" height="196" alt="image" src="https://github.com/user-attachments/assets/bcf6c223-5a70-4ff9-a925-91427b013a1e" />
  <img width="586" height="179" alt="image" src="https://github.com/user-attachments/assets/b0aa8549-f532-40e2-80d3-7239243d05b9" />

На первом сервере подключаемся к бд и создаем роль replicator с атрибутом replication:

  <img width="901" height="223" alt="image" src="https://github.com/user-attachments/assets/6d505635-ea8b-4a8d-870d-5fdd2a7d44e5" />

Добавляю роль в файл pg_hba.conf:

  <img width="790" height="48" alt="image" src="https://github.com/user-attachments/assets/623fba60-7746-471a-b431-cef7669fe73d" />

Правлю /etc/postgresql/14/main/postgresql.conf:

  <img width="1290" height="198" alt="image" src="https://github.com/user-attachments/assets/bc0a2f4b-60fa-450f-98fc-2eed41b82cf6" />

Перезапускаем postgresql:

  <img width="1775" height="435" alt="image" src="https://github.com/user-attachments/assets/cf337a96-4a78-4435-91b2-2d0495c43788" />

pg_create_physical_replication_slot — это системная функция PostgreSQL, которая создаёт слот физической репликации в базе данных.
Слот репликации — это механизм, который позволяет сохранять WAL (Write-Ahead Log) записи на главном сервере, пока они не будут получены репликой. Это предотвращает удаление WAL-файлов до тех пор, пока они необходимы для репликации.
Физическая репликация копирует и применяет записи WAL с главного сервера на реплику байт за байтом. Это обеспечивает точную копию базы данных на реплике.

Создаем слот реплекации:

  <img width="761" height="384" alt="image" src="https://github.com/user-attachments/assets/19e42b49-80dc-452c-b1f8-9e998bbc5e3a" />

На втором сервере останавливаем postgresql

    sudo systemctl stop postgresql
Правим /etc/postgresql/14/main/postgresql.conf

  <img width="879" height="175" alt="image" src="https://github.com/user-attachments/assets/1169a512-cb2a-469b-aa57-4f325592b3cc" />

На сервере ubuntu-serv02 вычищаем /var/lib/postgresql/14/main и создаем резервную копию всего кластера базы данных:

  <img width="1459" height="368" alt="image" src="https://github.com/user-attachments/assets/a70fcdb1-d303-48b5-9812-f0f339eaf962" />

pg_basebackup — утилита PostgreSQL для создания резервной копии всего кластера базы данных.

Параметры команды:

-h 192.168.56.11 — указывает хост (IP-адрес) сервера PostgreSQL, с которого будет производиться резервное копирование.

-p 5432 — порт подключения к серверу PostgreSQL (по умолчанию 5432).

-U replicator — имя пользователя, от которого выполняется подключение к серверу. В данном случае используется пользователь replicator, который должен иметь соответствующие права для создания резервной копии.

-D /var/lib/postgresql/14/main/ — путь, куда будет сохранена резервная копия. В данном случае это директория /var/lib/postgresql/14/main/.

-Fp — указывает формат вывода в простой (plain) формат, что означает сохранение данных в виде одного файла.

-Xs — включает режим streaming (потокового) резервного копирования, что позволяет более эффективно передавать данные.

-P — показывает прогресс копирования, отображая процент выполнения и скорость передачи данных.

-R — копирует файл recovery.conf в директорию резервной копии, что необходимо для настройки репликации.

–slot=replica_slot — указывает имя слота репликации, который будет использоваться для потокового резервного копирования. Слот репликации помогает сохранить данные на первичном сервере до завершения процесса копирования

Запускаем postgresql:

    sudo systemctl start postgresql

Создаю тестовую базу с таблицами и заполняю значениями:

  <img width="909" height="822" alt="image" src="https://github.com/user-attachments/assets/3b190817-6a62-41c8-90bd-894bade92fc5" />

Проверяем на ubuntu-serv02 наличие БД:

  <img width="731" height="495" alt="image" src="https://github.com/user-attachments/assets/7068171e-d1a8-4ca4-99fe-3fa1c11d26f3" />

Добавляю еще студентов на первом сервере/проверяю на втором сервере:

  <img width="922" height="901" alt="image" src="https://github.com/user-attachments/assets/1ae3b464-a277-4045-9d57-e6c3901ecd7d" />

  <img width="750" height="781" alt="image" src="https://github.com/user-attachments/assets/287b2d3c-3e7b-4e10-80a3-bd422969ca27" />

Пробую insert на втором сервере:

  <img width="838" height="112" alt="image" src="https://github.com/user-attachments/assets/3ff86231-982b-4e40-87ed-7be9e19cd496" />

Далее я переключился на дополнительное видео про HAProxy и повторил практику. База там уже была другая.

Создал и настроил еще одну машину ubuntu-etcd.

Устанавливаю etcd-server на ubuntu-etcd:

  <img width="559" height="232" alt="image" src="https://github.com/user-attachments/assets/640dda1b-20b2-466c-89ad-d1eda423d81e" />

Правлю файл /etc/default/etcd:

    sudo vim /etc/default/etcd
  <img width="751" height="271" alt="image" src="https://github.com/user-attachments/assets/c053a07b-44ca-483a-9c90-c12332a18419" />


Перезапускаю:

    sudo systemctl restart etcd
Проверяю, что все работает:

  <img width="1770" height="722" alt="image" src="https://github.com/user-attachments/assets/0d3e18af-d6a9-4aa9-b2cf-d8680a587d38" />

На ubuntu-serv01 и ubuntu-serv02 cоздаю символическую ссылку на все исполняемые файлы PostgreSQL для интеграции с Patroni:

    sudo ln -s /usr/lib/postgresql/14/bin/* /usr/sbin

Останавливаю postgres:

    sudo systemctl stop postgresql

Устанавливаю все необходимые python-пакеты и patroni на оба сервера, чтобы patroni мог работать с postgres:

    sudo apt install -y python3 python3-pip python3-testresources python3-psycopg2 patroni etcd-client

Настраиваем на ubuntu-server01 patroni:

    sudo vim /etc/patroni/config.yml


    scope: postgres
    namespace: /db/
    name: node1
    restapi:
        listen: 192.168.56.3:8008
        connect_address: 192.168.56.3:8008
    
    etcd:
        host: 192.168.56.2:2379
    
    bootstrap:
      dcs:
        ttl: 30
        loop_wait: 10
        retry_timeout: 10
        maximum_lag_on_failover: 1048576
    
        postgresql:
          use_pg_rewind: true
          use_slots: true
          parameters:
    
        initdb:
         - encoding: UTF8
         - data-checksums
    
        pg_hba:
        - host replication replicator 127.0.0.1/32 md5
        - host replication replicator 192.168.56.3/0 md5
        - host replication replicator 192.168.56.4/0 md5
        - host all all 0.0.0.0/0 md5
    
        users:
          admin:
            password: admin
            options:
              - createrole
              - createdb
    
    postgresql:
      listen: 192.168.56.3:5432
      connect_address: 192.168.56.3:5432
      data_dir: /data/patroni
      pgpass: /tmp/pgpass
    
      authentication:
        replication:
          username: replicator
          password: admin@123
    
        superuser:
          username: postgres
          password: admin@123
    
      parameters:
          unix_socket_directories: '.'
    
    tags:
        nofailover: false
        noloadbalance: false
        clonefrom: false
        nosync: false 

На втором сервере аналогично, только меням name на node2 и ip с 192.168.56.3 (ubuntu-serv01) на 192.168.56.4 (ubuntu-serv02).

Создаем на обоих серверах директорию /data/patroni и настраиваем права:

  <img width="626" height="124" alt="image" src="https://github.com/user-attachments/assets/9c7510c5-c81a-4b5e-b8c4-69bf6333913b" />
  <img width="648" height="171" alt="image" src="https://github.com/user-attachments/assets/33baa063-4779-4fe3-b9b1-18c615ce23f6" />

Создаем на обоих серверах patroni.service:

    sudo vim /etc/systemd/system/patroni.service

    [Unit]
    Description=High availability PostgreSQL Cluster
    After=syslog.target network.target
    
    [Service]
    Type=simple
    User=postgres
    Group=postgres
    ExecStart=/usr/bin/patroni /etc/patroni/config.yml
    KillMode=process
    TimeoutSec=30
    Restart=no
    
    [Install]
    WantedBy=multi-user.targ                          
На обоих серверах выполняем reload daemon:

    sudo systemctl daemon-reload

Запускаем patroni и првоеряем статус:

    sudo systemctl start petroni
    sudo systemctl status patroni

Смотрим, что patroni видит наш cluster:

  <img width="701" height="179" alt="image" src="https://github.com/user-attachments/assets/8440fd73-1249-4ed7-9060-9ba0814e8568" />

Так как ресурсы у меня ограничены, сделаю основную ВМ 192.168.56.1 в роли proxy:

Устанавливаем на ней haproxy:

  <img width="723" height="218" alt="image" src="https://github.com/user-attachments/assets/c2f9bb86-4fe8-4f53-934f-e33fb5675d6a" />

Добавляем конфиг для haproxy:

    sudo vim /etc/haproxy/haproxy.cfg

Текст конфига:

    global
          maxconn 100
          log 127.0.0.1 local2
    
    defaults
            mode tcp
            retries 2
            timeout client 30m
            timeout connect 4s
            timeout server 30m
            timeout check 5s
    
    listen stats
        mode http
        bind *:7000
        stats enable
        stats uri /
    
    listen primary
        bind *:5000
        option httpchk      OPTIONS /primary
        http-check expect status 200
        default-server inter 3s fall 3 rise 2 on-marked-down shutdown-sessions
        server node1 192.168.56.3:5432 maxconn 100 check port 8008
        server node2 192.168.56.4:5432 maxconn 100 check port 8008
    
    listen replica
        bind *:5001
        option httpchk      OPTIONS /replica
        http-check expect status 200
        default-server inter 3s fall 3 rise 2 on-marked-down shutdown-sessions
        server node1 192.168.56.3:5432 maxconn 100 check port 8008
        server node2 192.168.56.4:5432 maxconn 100 check port 8008
                                                             

Запускаю haproxy и проверяю:

    sudo systemctl restart haproxy
    sudo systemctl status haproxy

  <img width="1887" height="733" alt="image" src="https://github.com/user-attachments/assets/77ed9b5f-03cc-4f79-b551-732890385538" />

Создаю быд:

  <img width="745" height="505" alt="image" src="https://github.com/user-attachments/assets/79ad0937-8586-4d62-92e0-5485816a8994" />

На первом сервере останавливаю patroni и проверяю:

  <img width="589" height="116" alt="image" src="https://github.com/user-attachments/assets/cc291665-66bd-48bb-9a37-b6e9553cf967" />
  <img width="699" height="366" alt="image" src="https://github.com/user-attachments/assets/1f20c8d8-c529-4e1d-b22e-93d7e643b270" />

Запускаю patroni на первом сервере и проверяю:

  <img width="545" height="62" alt="image" src="https://github.com/user-attachments/assets/6e1e32af-7db6-4348-ab4d-57a5faf987c7" />
  <img width="701" height="515" alt="image" src="https://github.com/user-attachments/assets/68cd5a40-7528-49f5-a901-5b2557909d42" />





