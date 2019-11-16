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
  
  
  `security:`
  
  `     authorization: enabled`
  
  `     keyFile: /mongodb/mongodb-keyfile`

## Cluster configure

1. Adicione um endereço DNS ao IP fisico de cada servers. Por exemplo db-mongo-poca.domain.in, db-mongo-pocb.domain.in, db-mongo-pocc.domain.in;
2. Garanta que todos os server possuem conexão entre si pela porta 27017;
3. Garanta que o arquivo `/mongodb/mongodb-keyfile` está disponivel em todos os server com a permissão de leitura apenas (chmod 400 /mongodb/mongodb-keyfile);
4. Ative o mongod em todos os servers `service mongod start`;
5. Conecte-se no primeiro mongo pela cli e execute os comandos abaixo:
