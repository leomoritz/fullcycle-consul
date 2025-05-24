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

**bind:** ip do servidor. Para saber o ip, basta digitar ~ifconfig~ e o IP será apresentado no eth0 inet addr.

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

Para ligar um client em um server de forma automática
~~~bash
consul agent -bind=172.19.0.3 -data-dir=/var/lib/consul -config-dir=/etc/consul.d -retry-join=172.19.0.4
~~~

É possível fazer vários retry join caso um dos servers não esteja disponível, ele tenta o próximo
Para ligar um client em um server de forma automática
~~~bash
consul agent -bind=172.19.0.3 -data-dir=/var/lib/consul -config-dir=/etc/consul.d -retry-join=172.19.0.4 -retry-join=172.19.0.5
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

~~~bash
; <<>> DiG 9.16.44 <<>> @localhost -p 8600 nginx.service.consul
; (2 servers found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 51336
;; flags: qr aa rd; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;nginx.service.consul.          IN      A

;; ANSWER SECTION:
nginx.service.consul.   0       IN      A       172.19.0.5
nginx.service.consul.   0       IN      A       172.19.0.2

;; Query time: 0 msec
;; SERVER: 127.0.0.1#8600(127.0.0.1)
;; WHEN: Sat May 24 13:43:57 UTC 2025
;; MSG SIZE  rcvd: 81
~~~

5. Outras formas de buscar o serviço por meio do catalogo:
~~~bash
curl localhost:8500/v1/catalog/services
consul catalog nodes -service nginx
~~~

**Para mais informações e outros comandos, acessar documentação:** https://developer.hashicorp.com/consul/api-docs/catalog

## Implementando healthchecks nos serviços
* Ref: https://developer.hashicorp.com/consul/docs/register/health-check/vm

**Existem várias formas de checar se um serviço está disponível:**

* **Script:** As verificações de script invocam um aplicativo externo que executa a verificação de integridade, sai com um código de saída apropriado e, potencialmente, gera uma saída. As verificações de script são um dos tipos mais comuns de verificação.
* **HTTP:** As verificações HTTP fazem uma solicitação HTTP GET para a URL especificada e aguardam o tempo especificado. As verificações HTTP são um dos tipos mais comuns de verificação.
* **TCP:** As verificações TCP tentam se conectar a um IP ou nome de host e porta via TCP e aguardam o tempo especificado.
* **UDP:** As verificações UDP enviam datagramas UDP para o IP ou nome de host e porta especificados e aguardam o tempo especificado.
* **TTL**: As verificações de tempo de vida (TTL) são verificações passivas que aguardam atualizações do serviço. Se a verificação não receber uma atualização de status antes do período especificado, a verificação de integridade entra em um estado crítico.
* **Docker:** As verificações do Docker dependem de aplicativos externos empacotados com um contêiner Docker que são acionados por chamadas ao endpoint da API Exec do Docker.
* **gRPC:** As verificações gRPC sondam aplicativos que suportam o protocolo padrão de verificação de integridade gRPC.
* **H2ping:** As verificações de H2ping testam um endpoint que utiliza http2. A verificação se conecta ao endpoint e envia um quadro de ping.
* **Alias:** As verificações de alias representam o estado de integridade de outro nó ou serviço registrado.

Se a sua rede for executada em um ambiente Kubernetes, você poderá sincronizar as informações de integridade do serviço com as verificações de integridade do Kubernetes. Consulte Configurar verificações de integridade (https://developer.hashicorp.com/consul/docs/register/health-check/k8s) para o Consul no Kubernetes para obter mais detalhes.

**Registrar:**
Após definir as verificações de integridade, você deve registrar o serviço que as contém no Consul. Consulte Registrar Serviços e Verificações de Integridade (https://developer.hashicorp.com/consul/docs/register/service/vm) para obter mais informações. Se o serviço já estiver registrado, você pode recarregar o arquivo de configuração do serviço para implementar sua verificação de integridade. Consulte Recarregar (https://developer.hashicorp.com/consul/commands/reload) para obter mais informações.

**Exemplo utilizando consulclient01:**
Foi adicionado um novo elemento **check** no json, conforme exemplo abaixo:
~~~json
{
  "service": {
    "id": "nginx",
    "name": "nginx",
    "tags": ["web"],
    "port": 80,
    "check": {
      "id": "nginx",
      "name": "HTTP 80",
      "http": "http://localhost",
      "interval": "10s",
      "timeout": "1s"
    }
  }
}
~~~

Em seguida, é necessário acessar o client e atualizar o consul, conforme exemplo abaixo:
~~~bash
docker exec -it consulclient01 sh
consul reload
~~~

Neste exato momento, o consulclient01 irá logar um alerta do healthcheck, visto que o serviço está sem o nginx instalado:
~~~log
2025-05-24T17:25:54.797Z [WARN]  agent: Check is now critical: check=nginx
~~~

Sendo assim, vamos instalar o nginx no client onde implementamos o healthcheck:
~~~bash
apk add nginx
~~~

Em seguida, vamos subir o pid do nginx:
~~~bash
mkdir /run/nginx # caso ainda não exista
nginx # comando para subir
ps # comando para verificar se o nginx está executando (pid)
~~~

Após isso, é possível identificar os PIDs do nginx em execução:
~~~bash
PID   USER     TIME  COMMAND
    1 root      0:00 {docker-entrypoi} /usr/bin/dumb-init /bin/sh /usr/local/bin/docker-entrypoint.sh tail -f /dev/null
    7 root      0:00 tail -f /dev/null
   19 root      0:00 sh
   49 root      1:08 consul agent -bind=172.19.0.5 -data-dir=/var/lib/consul -config-dir=/etc/consul.d
  104 root      0:00 sh
  143 root      0:00 nginx: master process nginx
  144 nginx     0:00 nginx: worker process
  145 nginx     0:00 nginx: worker process
  146 nginx     0:00 nginx: worker process
  147 nginx     0:00 nginx: worker process
  148 root      0:00 ps
~~~

Além disso, podemos garantir que está acessível
~~~bash
curl localhost
~~~

Contudo, o healthcheck pode ainda estar falhando com a mensagem 
~~~log 
[WARN]  agent: Check is now critical: check=nginx
~~~
Neste caso, seguir os seguintes passos:

1. Instalar o vim caso não tenha e criar um diretório:
~~~bash
apk add vim
mkdir /usr/share/nginx/html -p
~~~

2. Acessar o arquivo conf do nginx através do vim onde poderemos ver que ele está retornando 404 pra qualquer coisa:
~~~bash
vim /etc/nginx/http.d/default.conf
~~~

~~~conf
server {
    listen 80 default_server;
    listen [::] 80 default_server;

    #Everthing is a 404
    location / {
        return 404;
    }

    # You may need this to prevent return 404 recursion.
    location = /404.html {
        internal;
    }
}
~~~

3. Será necessário editar o arquivo:
~~~conf
server {
    listen 80 default_server;
    listen [::] 80 default_server;

    root /usr/share/nginx/html;

    # You may need this to prevent return 404 recursion.
    location = /404.html {
        internal;
    }
}
~~~

4. Em seguida, será necessário editar o index.html no diretório root:
~~~bash
vim /usr/share/nginx/html/index.html
~~~

~~~html
<h1>Hello World</h1>
~~~

5. Agora, basta reiniciar o nginx
~~~bash
nginx -s reload
curl localhost # caso queira conferir se atualizou
~~~

6. Por fim, podemos conferir que o healthcheck parou de alertar
~~~log
2025-05-24T17:51:25.097Z [WARN]  agent: Check is now critical: check=nginx
2025-05-24T17:51:35.113Z [INFO]  agent: Synced check: check=nginx
~~~

E além disso, podemos voltar a enxergar o client entre os serviços
~~~bash
# Comando
 dig @localhost -p 8600 nginx.service.consul

# Saída
; <<>> DiG 9.16.44 <<>> @localhost -p 8600 nginx.service.consul
; (2 servers found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 22809
;; flags: qr aa rd; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;nginx.service.consul.          IN      A

;; ANSWER SECTION:
nginx.service.consul.   0       IN      A       172.19.0.2
nginx.service.consul.   0       IN      A       172.19.0.5

;; Query time: 0 msec
;; SERVER: 127.0.0.1#8600(127.0.0.1)
;; WHEN: Sat May 24 17:54:53 UTC 2025
;; MSG SIZE  rcvd: 81
~~~

## Subindo server via arquivo pra evitar comandos bash
É possível configurar um arquivo para subir os servers, segue exemplo abaixo onde os IPs podem mudar de acordo com os servers:
~~~json
{   
    "server":true,
    "bind_addr":"172.19.0.4",
    "bootstrap_expect":3,
    "data_dir":"/tmp",
    "retry_join":["172.19.0.3", "172.19.0.6"]
}
~~~
* Arquivo foi criado em servers/server01/server.json

Após isso, precisamos mapear o volume no docker-compose no server01:
~~~yaml
volumes:
        - ./servers/server01:/etc/consul.d
~~~

Em seguida, derrubar o docker-compose e subir novamente
~~~bash
docker-compose down
docker-compose up -d
~~~

Por fim, basta acessar o server e executar o comando consul
~~~bash
docker exec -it consulserver01 sh
consul agent -config-dir=/etc/consul.d
~~~

OBS: Lembrar de conferir os IPs configurados no arquivo e também de executar os servers configurados no retry_join.

