# Docker Compose
[Docker Compose - CLI](https://docs.docker.com/compose/reference/overview/)
[Install Docker Compose](https://docs.docker.com/compose/install)

É uma ferramenta para definir e rodar aplicações docker multi-container

Por padrão já vem instalado no Mac E Win, mas no Linux é necessário instalar o binário

- Nome e Extensão padrão: docker-compose.yml


|Compose file format	|Docker Engine release|
|---------------------|---------------------|
|3.8                  |19.03.0+             |
|3.7                  |18.06.0+             |
|3.6                  |18.02.0+             |
|3.5                  |17.12.0+             |
|3.4                  |17.09.0+             |
|3.3                  |17.06.0+             |
|3.2                  |17.04.0+             |
|3.1                  |1.13.1+              |
|3.0                  |1.13.0+              |
|2.4                  |17.12.0+             |
|2.3                  |17.06.0+             |
|2.2                  |1.13.0+              |
|2.1                  |1.12.0+              |
|2.0                  |1.10.0+              |
|1.0                  |1.9.1.+              |


## Rodando o promeiro "docker-compose up"
[Docker Hub - sapk/cloud9](https://hub.docker.com/r/sapk/cloud9)

Primeiramente vamos criar o arquivo `docker-compose.yml` com a seguinte estrutura, mas é preciso verificar a indentação, pois pode ser interpretado de forma incorreta pelo docker o que irá gerar um erro

```bash
# versão
version: '3'

# Iniciamos o serviço
services:
# nome do container
  cloud9:
# Nome da imagem
    image: sapk/cloud9
# Volume na máquina
    volumes:
      - .:/workspace
# Portas que vamos utilizar
    ports:
      - "8181:8181"
# Usuário e senha
    command: --auth username:password
```

Agora vamos inicia a máquina

```bash
$ docker-compose up -d
```
OU
```bash
$ docker-compose -f <nameFile.yml> up -d
```

Agora já temos nosso conteiner iniciado, agora vamos parar esse container

```bash
$ docker-compose down
```
OU
```bash
$ docker-compose -f <nameFile.yml> down
```

##### Comandos Úteis
```bash
$ docker-compose up
$ docker-compose -f <nameFile.yml> up
$ docker-compose up -d
$ docker-compose -f <nameFile.yml> up -d
$ docker-compose down
$ docker-compose -f <nameFile.yml> down
$ docker-compose -f <nameFile.yml> snameFiletop
```

Agora vamos criar um container com Node.JS

###### docker-compose.yml
```bash
version: '3'

services:
  nodejs:
    image: paulorcvieira/ubuntu-sshd-nvm
    volumes:
      - .:/workspace
    ports:
      - "22"
      - "3030"
```

E vamos subir esse container

```bash
$ docker-compose up -d
```

## Conhecendo o "docker-compose up --scale"
[docker-compose up --scale](https://docs.docker.com/compose/reference/up/#scale)

Podemos rodar vários serviços ao mesmo tempo

- Padrão: --scale SERVICE=NUM

Vamos subir 5 seviços, com portas sendo mapeadas pelo próprio docker

###### docker-compose.yml
```bash
version: '3'

services:
  nodejs:
    image: paulorcvieira/ubuntu-sshd-nvm
    volumes:
      - .:/workspace
    ports:
      - "22"
      - "3030"
```

Agora pra subir os seviços

```bash
$ docker-compose -d --scale nodejs=5
$ docker container ls
```

## Conhecendo o "docker-compose run"
[Docker-Compose Run](https://docs.docker.com/compose/reference/run/)

- Padrão: docker-compose run [options] [-v VOLUME...] [-p PORT...] [-e KEY=VAL...] [-l KEY=VALUE...] SERVICE [COMMAND] [ARGS...]

Com o RUN podemos selecionar as portas, volume, entre outros, mas aqui vamos usar apenas para escolher as portas, lembrando que ainda estou com os serviços ativos

Neste ecemplo vou mudar as portas local e vou manter a última configuração das portas internas do docker


```bash
$ docker-compose run -d -p <port_local>:<port_docker> -p <port_local>:<port_docker> <service>
```

```bash
$ docker-compose run -d -p 2222:22 -p 3333:3030 nodejs
$ docker container ls
$ docker-compose down
```

## Trabalhando com Imagens e Docker-Compose
[File - Build](https://docs.docker.com/compose/compose-file/#build)
[CLI - Build](https://docs.docker.com/compose/reference/build/)
[CLI - Up](https://docs.docker.com/compose/reference/up/)

Aqui vamos trabalhar com a instrução *build*  onde vamos indicar o local do Dockerfile, para isso vamos criar nosso `Dockerfile`

```bash
FROM  ubuntu:latest
LABEL maintainer="paulorcvieira"

# Atualiza o SO
RUN apt-get update && apt-get install -y openssh-server vim curl git sudo
RUN apt-get install -y git
RUN mkdir /var/run/sshd

# Configura o SSH
RUN echo 'root:root' |chpasswd
RUN sed -ri 's/^PermitRootLogin\s+.*/PermitRootLogin yes/' /etc/ssh/sshd_config
RUN sed -ri 's/UsePAM yes/#UsePAM yes/g' /etc/ssh/sshd_config

# Boas-Vindas
RUN echo 'Banner /etc/banner' >> /etc/ssh/sshd_config
COPY etc/banner /etc/

# Adciona o usuário 'app'
RUN useradd -ms /bin/bash app
RUN adduser app sudo
RUN echo 'app:app' |chpasswd

# Altera para o usuário 'app'
USER app

# Instala o NVM
RUN /bin/bash -l -c "curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.35.3/install.sh | bash"
RUN /bin/bash -l -c ". ~/.nvm/nvm.sh && nvm install 12.16.3"

# Altera para o usuário 'root'
USER root

# Expõe as portas
EXPOSE 22
EXPOSE 3030

# Cria e configura o ponto de montagem do volume
RUN mkdir /workspace
RUN chmod 777 /workspace
VOLUME /workspace

CMD    ["/usr/sbin/sshd", "-D"]
```

Agora vamos criar o `docker-compose.yml` com as instrução `build`

```bash
version: '3'

services:
  nodejs:
    build: .
    image: paulorcvieira/ubuntu-sshd-nvm:latest
    volumes:
      - .:/workspace
    ports:
      - "22"
      - "3030"
```

E também vamos utilizar a tela de boas vindas que haviamos utilizado anteriormente `etc/banner`

```bash
              |
.  .    . .-. | .-..-. .--.--. .-.
 \  \  / (.-' |(  (   )|  |  |(.-'
  `' `'   `--'` `-'`-' '  '  ` `--'

This version add:

- VIM
- cURL
- NVM
- Node 12.16.3
- Git
- sudo
```

Vamos subir nosso compose, e para isso precisamos confirmar se estamos na mesma pasta de arquivos, e *caso não tenhamos essa imagem* criada anteriormente vamos rodar o comando

```bash
$ docker-compose up
```

Mas vamos *gerar uma nova image*, pois já tenho essa imagem, sendo assim, vamos rodar o comando

```bash
$ docker-compose build
```

Também temos a opção de quando gerarmos essa nova imagem, subir ele para já começar a utilizar o container em modo foreground, e para isso precisamos rodar o comando

```bash
$ docker-compose up -d --build
```

Caso tenhamos usado a segunda opção nosso container já foi montado, se não precisamos rodar o comando

```bash
$ docker images
$ docker container run -P -d <image id>
```

Agora podemos utilizar nosso container, vamos pegar a porta e entrar no ssh com o `usuário: app` e `senha: app`

```bash
$ docker container ls
$ docker container port <container id>
$ ssh app@localhost -p <port>
$ exit
```

Podemos apagar parar e apagar nossos containers

```bash
$ docker container stop <container id1> <container id2>
$ docker container prune
```

## Docker-Compose com múltiplos containers/imagens

Para isso vamos criar um arquivo `docker-compose.yml` na pasta raiz do nosso diretório

```bash
version: '3'

services:
  nodejs:
    build: .
    image: paulorcvieira/ubuntu-sshd-nvm:latest
    volumes:
      - .:/workspace
    ports:
      - "22"
      - "3030:3030"
  db:
    image: postgres
    restart: always
    environment:
      POSTGRES_PASSWORD: password
    ports:
      - "5454"
    volumes:
      - ./crud-node-postgres/database:/var/lib/postgresql/data
```

Agora vamos levantar a imagem

```bash
$ docker-compose up -d
$ docker container ls
$ docker container port <container id>
$ ssh app@localhost -p <port>
```

Já estamos dentro de nosso SSH, vamos acessa nosso workspace

```bash
$ cd /workspace/
$ ls
```

Precisamos conectar o pgAdmin

```bash
$ docker container ls
$ docker container port <container id>
```

Agora vamos criar um server no pgAdmin com nome `docker-crud` utilizando a porta do nosso container *postgres*, dentro desse server vamos criar um banco de dados `crud-node`

Feito isso vamos instalar os módulos do nosso projeto, como já estamos dentro do nosso `/workspace/` precisamos entrar na pasta do projeto antes

```bash
$ cd crud-node-postgres/
$ npm install -g bower
$ npm install
$ bower install
```

Agora precisamos configura o arquivo `/server/config/database.js` e colocar o nome do banco de dados criado no pgAdmin

Já podemos inserir os dados no postgres e iniciar nossa aplicação

```bash
$ node prepare.js
$ node server.js
```

Pronto! vamos acessar nosso endpoint `http://localhost:3030`

## Docker-Compose (depends_on)
[Doc - depends_on](https://docs.docker.com/compose/compose-file/#depends_on)

Com o `depends_on` podemos criar dependência entre serviços, onde um serviço depende de outro, vamos criar um `docker-compose-yml`

```bash
version: '3'

services:
  nodejs:
    build: .
    image: paulorcvieira/ubuntu-sshd-nvm:latest
    volumes:
      - .:/workspace
    ports:
      - "22"
      - "3030:3030"
    depends_on:
      - db
  db:
    image: postgres
    restart: always
    environment:
      POSTGRES_PASSWORD: password
    ports:
      - "5432"
    volumes:
      - ./crud-node-postgres/database:/var/lib/postgresql/data
```

É isso agora podemos levantar um serviço e antes vai carregar a dependeência, sem a qual não faria sentido iniciar o container

```bash
$ docker-compose up -d
```

## Docker-Compose (networks)

Vamos colocar nossos serviços em uma determinada rede, para isso precisamos colocar a instrução `networks` e o `nome da rede` a qual pertence em cada um dos seviços, junto já vamos usar um `aliases` para cada serviço, e no final vamos precisar declarar as redes que utilizamos

```bash
version: '3'

services:
  nodejs:
    build: .
    image: paulorcvieira/ubuntu-sshd-nvm:latest
    volumes:
      - .:/workspace
    ports:
      - "22"
      - "3030:3030"
    depends_on:
      - db
    networks:
      - myNetworks
  db:
    image: postgres
    restart: always
    environment:
      POSTGRES_PASSWORD: password
    ports:
      - "5432"
    volumes:
      - ./crud-node-postgres/database:/var/lib/postgresql/data
    networks:
      myNetworks:
        aliases:
          - postgresql

networks:
  myNetworks:
```

Agora vamos testar, e para isso precisamos levantar a imagem, acessar o ssh e instalar o `iputils-ping`

```bash
$ docker-compose -f docker-compose-networks.yml up -d
$ docker container ls
$ ssh app@localhost -p 32782
$ sudo apt-get install iputils-ping
$ ping db
$ ping postgresql
$ exit
$ docker container stop <container id>
$ docker container prune
$ docker images
$ docker image rmi <image id>
```

Pronto, conseguimos dar um `ping` tanto diretamente no `services db` quando no `aliases postgresql` que serve como uma apelido para o serviço.
Também já aproveitei pra finalizar os containes e apagar a imagem.