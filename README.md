# Real World Project: Database Shard Github 

# docker-compose.yml:

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
