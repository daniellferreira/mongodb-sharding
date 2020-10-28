# mongodb-sharding

//ATENÇÃO: PASSO 0 SOMENTE SE NÃO QUISER PRESERVAR NENHUMA IMAGEM DO DOCKER RODANDO
0 => Acessar como superuser, parar e excluir todos os atuais containers
cd /home/daniel/workspace/unisinos/bd2/sharding/sharding
sudo su
docker stop $(docker ps -a -q) && docker rm $(docker ps -a -q) && docker volume prune


//CONFIG

1 => Iniciar instancia que será config server com replicaset
docker-compose -f config-server/docker-compose.yaml up -d

2 => Conectar ao config server
mongo mongodb://192.168.100.52:40001

3 => Iniciar o replica set do config server
rs.status()
rs.initiate( {
    _id: "cfgrs",
    configsvr: true,
    members: [
      { _id : 0, host : "192.168.100.52:40001" },
      { _id : 1, host : "192.168.100.52:40002" },
      { _id : 2, host : "192.168.100.52:40003" }
    ]
  }
)
rs.status()

//

//SHARD1

4 => Iniciar instancia que será shard1 com replicaset 
docker-compose -f shard1/docker-compose.yaml up -d

5 => Conectar ao shard1 server
mongo mongodb://192.168.100.52:50001

6 => Iniciar o replica set do shard1 server
rs.status()
rs.initiate(
  {
    _id: "shard1rs",
    members: [
      { _id : 0, host : "192.168.100.52:50001" },
      { _id : 1, host : "192.168.100.52:50002" },
      { _id : 2, host : "192.168.100.52:50003" }
    ]
  }
)
rs.status()

//

//SHARD2

7 => Iniciar instancia que será shard2 com replicaset
docker-compose -f shard2/docker-compose.yaml up -d

8 => Conectar ao shard2 server
mongo mongodb://192.168.100.52:50004

9 => Iniciar o replica set do shard2 server
rs.status()
rs.initiate(
  {
    _id: "shard2rs",
    members: [
      { _id : 0, host : "192.168.100.52:50004" },
      { _id : 1, host : "192.168.100.52:50005" },
      { _id : 2, host : "192.168.100.52:50006" }
    ]
  }
)
rs.status()

//

//MONGOS ROUTER

10 => Iniciar instancia que será Mongos Router
docker-compose -f mongos/docker-compose.yaml up -d

11 => Conectar ao mongos
mongo mongodb://192.168.100.52:60000

12 => Adicionar shard1 e shard2 ao cluster
sh.status()
sh.addShard("shard1rs/192.168.100.52:50001,192.168.100.52:50002,192.168.100.52:50003")
sh.status()
sh.addShard("shard2rs/192.168.100.52:50004,192.168.100.52:50005,192.168.100.52:50006")
sh.status()

//

// Collection

13 => Criar database <nome>, criar collection movies

14 => Já conectado ao Mongos Router, habilitar o sharding para a database:
sh.enableSharding("<nome>")

15 => Habilitar shard para collection e definir shard key:
sh.shardCollection("<nome>.movies", { title: "hashed" } )

16 => importar dataset movies.json

17 => rodar comando para verificar distribuição dos dados
use <nome>
db.movies.getShardDistribution()