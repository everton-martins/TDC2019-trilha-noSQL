# TDC2019-trilha-noSQL
## Desempate de cluster usando AWS sa-east-1

Neste projeto você encontrará a palestra realizada por mim no TDC 2019, trilha BigData e noSQL.
A apresentação está disponivel neste repositório no arquivo noSQL.pdf

Abaixo segue um complemento da apresentação com os parametros e código para montagem do cluster mongodb com arbitro.

## Pametros

Estes são os parametros que minimamente você deve adicionar ao /etc/mongod.conf:

  `# network interfaces`
  
  `net:`
  
  `     port: 27017`
  
  `     bindIp: 0.0.0.0  # Listen to local interface only, comment to listen on all interfaces. `
  
  
  `replication: `
  
  `     oplogSizeMB: 500`
  
  `     replSetName: RS`
  
  `     enableMajorityReadConcern: false`
  
  
  `security:`
  
  `     authorization: enabled`
  
  `     keyFile: /mongodb/mongodb-keyfile`

## Cluster configure

1. Adicione um endereço DNS ao IP fisico de cada servers. Por exemplo db-mongo-poca.domain.in, db-mongo-pocb.domain.in, db-mongo-pocc.domain.in;
2. Garanta que todos os server possuem conexão entre si pela porta 27017;
3. Garanta que o arquivo `/mongodb/mongodb-keyfile` está disponivel em todos os server com a permissão de leitura apenas (chmod 400 /mongodb/mongodb-keyfile);
4. Ative o mongod em todos os servers `service mongod start`;
5. Conecte-se no primeiro mongo pela cli e execute os comandos abaixo:

  `rs.initiate( {`
  
  `  _id : "RS",`
  
  `members: [`
  
  `  { _id: 0, host: "db-mongo-poca.domain.in:27017" },`
  
  ` { _id: 1, host: "db-mongo-pocb.domain.in:27017" },`
  
  ` ] `
  
  ` }) `
  
  ` rs.conf() `
  
  ` rs.status() `
  
  6. Crie um usuário root
  
  ` use admin`
    
  ` db.createUser( `
    
  ` { `
    
  `  user: "root", `
    
  `  pwd: "superuser", `
    
  `  roles: [{"role":"root","db":"admin"}] `
    
  ` } `
 
  ` ) `
 
 7. Agora você pode adicionar o seu nodo arbitro.
 
  `rs.addArb("db-mongo-pocc.domain.in:27017")`
  
  `rs.config()`
  
  Ao executar o config novamente você verá que ele está com os parametros Vote=1, priority=0, arbiterOnly=true. Agora vamos torna-lo hidden para os clientes que conectarem no cluster.
  
  `cfg = rs.conf()`
  
  `cfg.members[2].hidden = true`
  
  `rs.reconfig(cfg)`


## Extras


### Read Concern "majority"
  
  Esta feature garante que o dado lido foi escrito na maioria dos membros do cluster. No exemplo acima ela foi desabilitada pois existem apenas 2 membros que possuem dados, e em caso de queda de um deles haverá pressão de IO no nodo restante.

  Indico a leitura da documentação a seguir: https://docs.mongodb.com/manual/reference/read-concern/


### Write Concern

  É utilizado para garantir escrita do dado nos nodos do cluster, é composto de três parametros:
  
  `W`: que define a quantidade de nodos que receberão este dado, antes que o cluster devolva um ACK para a aplicação. Pode conter:
  
    um valor numérico de 0 a N: que indica a quantidade de nodos que receberão a escrita, antes de retornar o ACK;
    
    uma string "majority": que significa que o dados será escrito na maioria dos nodos, antes de retornar o ACK;
    
    uma string que define o conjunto de nodos por tag (https://docs.mongodb.com/manual/tutorial/configure-replica-set-tag-sets/#configure-custom-write-concern)
    
    
  `J`: true ou false, que indica que o valor deve ser escrito em disco (journal), antes do ACK.
  
  
  `wtimeout`: time out da escrita.

  Indico a leitura da documentação a seguir: https://docs.mongodb.com/manual/reference/write-concern/


### Priority

  Existem casos em que é importante ter um nodo com maior prioridade para se tornar o primário, por exemplo, quando temos um nodo com maior capacidade, ou está mais 'proximo' da aplicação que realiza escrita.
  
  Neste caso podemos configurá-lo com maior prioridade.
  
  `cfg = rs.conf()`
  
  `cfg.members[1].priority = 2`
  
  `rs.reconfig(cfg)`

  https://docs.mongodb.com/manual/tutorial/adjust-replica-set-member-priority/


### nonVote, secondary only

  Se configurarmos um membro como Vote=0, ele obrigatóriamente não poderá assumir como primário e sua prioridade será 0 também.

  Essa configuração se aplica em caso de termos um membro em uma região remota, com grande latência, que suporta LAG, e atende, por exemplo, as leituras de uma aplicação específica.


### settings.heartbeatTimeoutSecs & settings.electionTimeoutMillis

  Respectivamente tempo para determinar que um membro está inacessível e tempo para determinar que o primary está fora e eleger um novo primary. Valores altos tornarão o failover lento, baixo valores podem causar failover desnecessários.
