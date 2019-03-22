# Real World Project: Database Shard Github 
1.Install Docker

sudo apt install docker.io

2.Install Docker Compose

https://docs.docker.com/compose/install/

3.Install mariadb

https://hub.docker.com/_/mariadb

4.Pull MAxscale github and connect it with the existing containers

sudo git clone https://github.com/mariadb-corporation/MaxScale

5.Add phpmyadmin to docker-compose.yml as described in the "docker-compose.yml" section

6.Use phpmyadmin to run a SQL command that shards a database by the id being even or odd.


# docker-compose.yml:
version: '2'
services:
    master:
        image: mariadb:latest
        environment:
            MYSQL_ALLOW_EMPTY_PASSWORD: 'Y'
        volumes:
            - ./sql/master:/docker-entrypoint-initdb.d
        command: mysqld --log-bin=mariadb-bin --binlog-format=ROW --server-id=3000
        ports:
            - "4001:3306"

    slave1:
        image: mariadb:latest
        depends_on:
            - master
        environment:
            MYSQL_ALLOW_EMPTY_PASSWORD: 'Y'
        volumes:
            - ./sql/slave:/docker-entrypoint-initdb.d
        command: mysqld --log-bin=mariadb-bin --binlog-format=ROW --server-id=3001 --log-slave-updates
        ports:
            - "4002:3306"

    slave2:
        image: mariadb:latest
        depends_on:
            - master
        environment:
            MYSQL_ALLOW_EMPTY_PASSWORD: 'Y'
        volumes:
            - ./sql/slave:/docker-entrypoint-initdb.d
        command: mysqld --log-bin=mariadb-bin --binlog-format=ROW --server-id=3002 --log-slave-updates
        ports:
            - "4003:3306"

    maxscale:
        image: mariadb/maxscale:latest
        depends_on:
            - master
            - slave1
            - slave2
        volumes:
            - ./maxscale.cnf.d:/etc/maxscale.cnf.d
        ports:
            - "4006:4006"  # readwrite port
            - "4008:4008"  # readonly port
            - "8989:8989"  # REST API port
    phpmyadmin:
        image: phpmyadmin/phpmyadmin
        container_name: phpmyadmin
        depends_on:
            - master
        environment:
            - PMA_HOST=maxscale
            - PMA_PORT=4006
        restart: always
        ports:
            - 8080:80
        volumes:
            - /sessions
            
            
            
# maxscale.cnf:
[maxscale]
threads=8

[serverONE]
type=server
address=slave1
port=3306

[serverTWO]
type=server
address=slave2
port=3306

[Sharded-Service]
type=service
router=schemarouter
servers=serverONE,serverTWO
user=shard
password=maxpwd

[Sharded-Service-Listener]
type=listener
service=Sharded-Service
protocol=MariaDBClient
port=4000

[MySQL-Monitor]
type=monitor
module=mariadbmon
servers=accounts_west,accounts_east
user=shard
password=maxpwd
monitor_interval=1000
