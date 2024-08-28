# ***Construção de um pipeline de dados com fonte de arquivos na máquina local e movimentação com o Airbyte para levar ao SGBD (PostgreSQL)***


## **Ferramentas usadas nesse lab**: 
Docker, Airbyte, PostgreSQL e PgAdmin.


## **Resumo**: 
* Criar imagem e container para o Airbyte;
* Criar imagem e container para o PostgreSQL;
* Configurar o SGBD;
* Acessar o Airbyte http://localhost:8000, fazer a extração dos dados da fonte (máquina local) e carga para o destino (SGBD).


## **Comandos**:
### Criar imagem e container do Airbyte no Docker
Acessar o terminal e instalar o git: https://git-scm.com/download/win para Windows

Executar os comandos abaixo:
git clone https://github.com/airbytehq/airbyte.git
cd airbyte
docker-compose up


### Criar imagem e container do banco de dados local
Executar o comando abaixo:
docker run --name dbdsa-lab4 -p 5432:5432 -e POSTGRES_USER=dsa -e POSTGRES_PASSWORD=dsa123 -e POSTGRES_DB=dsadb -d postgres


### Configurar o SGBD
Acessar o SGBD e criar:
Name SGBD Pgadmin: Lab4
Schema: dsadb


### Configuração de origem e destino no Airbyte
Para a leitura de arquivos local é necessário realizar o mapeamento de volumes direto no CMD da máquina local com o comando: 
docker cp C:\Arquivos\"Nome_Arquivo.csv" airbyte-server:\tmp\airbyte_local
Além disso, o arquivo deve estar na pasta raiz nomeada como "Arquivos"

#Criando a conexão
Acessar o Airbyte:
User: airbyte
Password: password

Fazer as configurações da fonte, destino e criar a conexão entre ambos