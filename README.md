
# MongoDB Sharding

Este código fornece um starter para se trabalhar com sharding no MongoDB, siga o passo a passo abaixo:

## Sumário

  - [Configurando o Servidor](#configurando-o-servidor)
  - [Criando o Shard 1](#criando-o-shard-1)
  - [Criando o Shard 2](#criando-o-shard-2)
  - [Criando o Mongos Router](#criando-o-mongos-router)
  - [Adicionando o Sharding e Dados](#adicionando-o-sharding-e-dados)


#### ♦️ ATENÇÃO: Se você não quiser preservar nenhuma imagem no docker rodando atualmente, acesse como superuser, pare e exclua todos os atuais containers com os seguinte comandos:

    sudo su

    docker stop $(docker ps -a -q) && docker rm $(docker ps -a -q) && docker volume prune


## Configurando o Servidor

1. Iniciar instancia que será config server com replicaset
```
    docker-compose -f config-server/docker-compose.yaml up -d
```

2. Conectar ao config server

    OBS: Nesta etapa, deve-se utilizar o IP da sua máquina (basta rodar `ipconfig` e verificar o IPv4 Address)
```
    mongo mongodb://192.168.100.52:40001
```

3. Iniciar o replica set do config server

    OBS: Lembre de modificar o IP para o seu local
```json
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
```


## Criando o Shard 1
1. Para iniciar instância que será shard1 com replicaset, rode o sequinte comando:
```
    docker-compose -f shard1/docker-compose.yaml up -d
```

2. Conectar ao shard1 server
```
    mongo mongodb://192.168.100.52:50001
```

3. Iniciar o replica set do shard1 server

```json
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
```

## Criando o Shard 2

1.  Para iniciar instância que será shard1 com replicaset, rode o sequinte comando:
```
    docker-compose -f shard2/docker-compose.yaml up -d
```

2. Conectar ao shard2 server
```
    mongo mongodb://192.168.100.52:50004
```

3. Iniciar o replica set do shard2 server

```json
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
```

## Criando o Mongos Router

OBS: Antes de iniciar o container, acesse o arquivo yaml e modifique o IP para o seu local

1. Iniciar instância que será Mongos Router

```
    docker-compose -f mongos/docker-compose.yaml up -d
```

2. Conectar ao mongos

```
    mongo mongodb://192.168.100.52:60000
```

3. Adicionar shard1 e shard2 ao cluster

```json
    sh.status()

    sh.addShard("shard1rs/192.168.100.52:50001,192.168.100.52:50002,192.168.100.52:50003")

    sh.status()

    sh.addShard("shard2rs/192.168.100.52:50004,192.168.100.52:50005,192.168.100.52:50006")

    sh.status()
```


## Adicionando o Sharding e Dados
*Usado como nome do banco de dados "development" para o exemplo*

1. Utilizar o database "development"

    OBS: Não é necessário rodar um comando para criar a collection, basta inserir dentro dela.

```
    use development
```


2. Já conectado ao Mongos Router, habilitar o sharding para a database:

    OBS: Qualquer coleção que não tiver sharding habilitada, irá para o primary shard.

```json
    sh.enableSharding("development")
```

3. Habilitar shard para collection e definir shard key:

```json
    sh.shardCollection("development.movies", { title: "hashed" } )
```

4. Importar dataset movies.json na collection

```json
    mongoimport --host 192.168.1.23:60000 --jsonArray --db development --collection movies --file datasets/movies.json
```

5. Rodar comando para verificar distribuição dos dados

```
    use development
    db.movies.getShardDistribution()
```