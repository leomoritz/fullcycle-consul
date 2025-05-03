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