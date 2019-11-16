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

