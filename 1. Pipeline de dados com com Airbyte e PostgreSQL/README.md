# 🚀 ***Pipeline de Dados com Airbyte e PostgreSQL***

## **Descrição do Projeto:**
Este projeto demonstra a implementação de um pipeline de dados utilizando **Airbyte** para integração de dados e **PostgreSQL** como banco de dados destino. O objetivo é desenhar e executar processos de ETL (Extração, Transformação e Carregamento) para um Data Warehouse moderno.


## 🛠️ **Tecnologias Utilizadas**: 
- **Airbyte**: Plataforma de integração de dados de código aberto que facilita a movimentação de dados entre fontes e destinos.
- **PostgreSQL**: Banco de dados relacional para armazenamento e manipulação de dados.
- **Docker**: Utilizado para containerização dos serviços, garantindo consistência e isolamento dos ambientes.


## **Funcionalidades**

1. **Criação e Configuração de Contêineres Docker**:
   - **Airbyte**: Configuração e execução do contêiner Airbyte para integração de dados.
   - **PostgreSQL**: Criação e execução do contêiner PostgreSQL para armazenamento dos dados.

2. **Configuração do Banco de Dados**:
   - Definição e criação do banco de dados e schema no PostgreSQL para armazenar os dados integrados.

3. **Integração de Dados com Airbyte**:
   - **Fonte de Dados**: Configuração do Airbyte para extrair dados de arquivos locais.
   - **Destino de Dados**: Configuração do Airbyte para carregar os dados extraídos no PostgreSQL.

4. **Gerenciamento de Arquivos Locais**:
   - Mapeamento de volumes Docker para permitir a leitura de arquivos CSV da máquina local e a transferência desses arquivos para o Airbyte.

5. **Interface de Configuração Airbyte**:
   - Acesso à interface do Airbyte para configuração de conexões de origem e destino.
   - Criação e gestão de pipelines de dados na interface do Airbyte.


## 📋 **Descrição do Processo**
* Criar imagem e container para o Airbyte;
* Criar imagem e container para o PostgreSQL;
* Configurar o SGBD;
* Acessar o Airbyte http://localhost:8000, fazer a extração dos dados da fonte (máquina local) e carga para o destino (SGBD).


## ⚙️ **Comandos**:
### Criar imagem e container do Airbyte no Docker
Acessar o terminal e instalar o git: https://git-scm.com/download/win para Windows
 
Executar os comandos abaixo:
git clone https://github.com/airbytehq/airbyte.git

cd airbyte

docker-compose up

---

### Criar imagem e container do banco de dados local
Executar o comando abaixo:

docker run --name dbdsa-lab4 -p 5432:5432 -e POSTGRES_USER=dsa -e POSTGRES_PASSWORD=dsa123 -e POSTGRES_DB=dsadb -d postgres

---

### Configurar o SGBD
Acessar o SGBD e criar:

Name SGBD Pgadmin: Lab4

Schema: dsadb

---

### Configuração de origem e destino no Airbyte
Para a leitura de arquivos local é necessário realizar o mapeamento de volumes direto no CMD da máquina local com o comando: 
docker cp C:\Arquivos\"Nome_Arquivo.csv" airbyte-server:\tmp\airbyte_local (Executar no CMD local)

Além disso, o arquivo deve estar na pasta raiz nomeada como "Arquivos"

---

### Criando a conexão 

Acessar o Airbyte:

User: airbyte

Password: password

Fazer as configurações da fonte, destino e criar a conexão entre ambos

---
## Contato

Se tiver dúvidas ou sugestões sobre o projeto, entre em contato comigo:

- 💼 [LinkedIn](https://www.linkedin.com/in/henrique-k-32967a2b5/)
- 🐱 [GitHub](https://github.com/henriquekurata?tab=overview&from=2024-09-01&to=2024-09-01)