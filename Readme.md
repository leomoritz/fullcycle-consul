## Execute localmente 
Clone o projeto  
* HTTPS
~~~bash  
git clone https://github.com/leomoritz/fullcycle-consul.git
~~~

* SSH

~~~bash
git clone git@github.com:leomoritz/fullcycle-consul.git
~~~

Executa o container

~~~bash  
docker compose up -d
~~~

Entra no container via sh

~~~bash  
docker exec -it consul01 sh
~~~

Executa o consul em modo dev (apenas para ambiente de testes, não deve ser usado em produção)

~~~bash  
consul agent -dev
~~~

Verifica quais hosts fazem parte do cluster 

~~~bash  
consul members
~~~

Pegando informações do server através da API do consul

~~~bash  
curl localhost:8500/v1/catalog/nodes
~~~

## Consulta informações dos servers via servidor DNS

Instalando o dig (comando abaixo para distribuições alpine)

~~~bash  
apk -u add bind-tools
~~~

Consultando todos hosts no servidor DNS

~~~bash  
dig @localhost -p 8600
~~~

Consultando host específico no servidor DNS (é obrigatório incluir *.node.consul* após o nome do server que se deseja consultar)

~~~bash  
dig @localhost -p 8600 consul01.node.consul
~~~

## Criando um cluster

Executa o consul em modo server

~~~bash
mkdir /etc/consul.d
mkdir /var/lib/consul
consul agent -server -bootstrap-expect=3 -node=consulserver01 -bind=172.19.0.3 -data-dir=/var/lib/consul -config-dir=/etc/consul.d
~~~

**node:** nome do servidor

**bind:** ip do servidor

**-data-dir:** local no servidor onde ficarão os arquivos do consul

**-config-dir:** local no servidor onde ficarão os arquivos de configuração do consul

Liga um server com outro(s) server(s) consul que também foi executado em modo server. Por exemplo:

~~~bash
consul join 172.19.0.4
~~~

## Criando um client

Executa o consul em modo client

~~~bash
mkdir /var/lib/consul
consul agent -bind=172.19.0.3 -data-dir=/var/lib/consul -config-dir=/etc/consul.d
~~~

Liga um client com um dos servers do cluster criado anteriormente. Por exemplo:

~~~bash
consul join 172.19.0.2
~~~

## Registrando um serviço

1. Foi feito o registro do serviço *nginx* em **clients/consul01/services.json**. 
2. Depois disso, entrar no client e rodar o comando abaixo para:

~~~bash
consul reload
~~~

3. Com isso, um log de [INFO] será apresentado no client:
~~~log
2025-05-13T10:41:13.457Z [INFO]  agent: Synced node info
2025-05-13T10:41:13.463Z [INFO]  agent: Synced service: service=nginx
~~~

4. Após isso, é possível consultar o serviço registrado no consul em qualquer um dos servers ou clients com os comandos abaixo:
~~~bash
apk -u add bind-tools
dig @localhost -p 8600 nginx.service.consul
~~~

5. Outras formas de buscar o serviço por meio do catalogo:
~~~bash
curl localhost:8500/v1/catalog/services
consul catalog nodes -service nginx
~~~

**Para mais informações e outros comandos, acessar documentação:** https://developer.hashicorp.com/consul/api-docs/catalog




