## Execute localmente 
Clone o projeto  
* HTTPS
~~~bash  
  git clone https://github.com/leomoritz/fullcycle-consul.git
~~~

* SSH

~~~bash
git@github.com:leomoritz/fullcycle-consul.git
~~~

Executa o container

~~~bash  
  docker compose up -d
~~~

Entra no container via sh

~~~bash  
docker exec -it consul01 sh
~~~

Executa o consult em modo dev

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

## Consulta via servidor DNS

Instalando o dig (distribuição alpine)

~~~bash  
apk -u add bind-tools
~~~

Consultando todos hosts no servidor DNS

~~~bash  
dig @localhost -p 8600
~~~

Consultando host específico no servidor DNS

~~~bash  
dig @localhost -p 8600 consul01.node.consul
~~~