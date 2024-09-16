# 🚀 ***Pipeline de dados com Apache Airflow***

## **Descrição do Projeto:**
Este projeto tem como objetivo criar e automatizar um pipeline de dados usando o **Apache Airflow** e **Python** para criar e inserir dados no **PostgreSQL**. A solução inclui a configuração de contêineres Docker para isolar e gerenciar o ambiente do PostgreSQL e do Apache Airflow, além da implementação de uma DAG (Directed Acyclic Graph) para gerenciar o processo ETL (Extract, Transform, Load).

## 🛠️ **Tecnologias Utilizadas**:
* Docker: Plataforma para criar e gerenciar containers, isolando os ambientes de execução do PostgreSQL e Apache Airflow;
* PostgreSQL: Banco de dados relacional onde os dados são armazenados e manipulados;
* pgAdmin: Interface gráfica para gerenciar e administrar o PostgreSQL;
* Apache Airflow: Ferramenta de orquestração para automatizar e gerenciar pipelines de dados;
* Anaconda: Distribuição de Python que facilita a gestão de pacotes e ambientes de desenvolvimento.

## **Principais Funcionalidades**
### 1. **Criação e Configuração de Contêineres Docker**
   - **PostgreSQL**: Configuração de um contêiner Docker para o banco de dados relacional PostgreSQL, incluindo a criação do banco de dados e schema necessários.
   - **Apache Airflow**: Configuração de um contêiner Docker para o Apache Airflow, incluindo a criação de uma rede dedicada para comunicação entre contêineres e a configuração inicial do Airflow.

### 2. **Gerenciamento de Contêineres com Docker**
   - **Criação de Imagens e Contêineres**: Processos automatizados para criar e iniciar imagens e contêineres necessários para PostgreSQL e Airflow.
   - **Comunicação entre Contêineres**: Configuração de redes e conexões para permitir que o Airflow se comunique com o banco de dados PostgreSQL.

### 3. **Criação e Configuração de DAG no Apache Airflow**
   - **Definição de DAG (Directed Acyclic Graph)**: Criação de uma DAG no Apache Airflow para gerenciar o pipeline de dados, incluindo tarefas de criação e inserção de dados.
   - **Tarefas ETL (Extract, Transform, Load)**: Implementação de tarefas específicas para criar tabelas e inserir dados no PostgreSQL usando o operador `PostgresOperator` do Airflow.

### 4. **Automatização do Pipeline de Dados**
   - **Execução Programada**: Configuração da DAG para ser executada em um horário específico ou manualmente através da interface do Airflow.

### 5. **Configuração e Monitoramento**
   - **Interface Web do Airflow**: Acesso ao painel do Apache Airflow para monitorar a execução das DAGs, visualizar logs e gerenciar tarefas.
   - **Criação de Conexões no Airflow**: Configuração de conexões no Airflow para se comunicar com o PostgreSQL, incluindo a definição de credenciais e detalhes de conexão.




## 📋 **Descrição do Processo**
* Criar imagem e container para o banco de dados do DW;
* Criar imagem e containers para o Apache Airflow;
* Configurar a comunicação entre as redes de containers (PostgreSQL e Airflow);
* Criar a connetion ID no Airlow;
* Criar a DAG;
* Inserir a DAG dentro da pasta raiz na máquina local do Airflow;
* Disparar a DAG.


## ⚙️ **Comandos**:

### Preparando o Container Docker Para o Banco de Dados do DW

Execute os comandos abaixo no terminal ou prompt de comando para baixar a imagem e inicializar o Postgres:

docker pull postgres

docker run --name dbdsa -p 5433:5432 -e POSTGRES_USER=dsalabdw -e POSTGRES_PASSWORD=dsalabdw123 -e POSTGRES_DB=dwdb -d postgres



### Configurar o SGBD

Acesse o Postgres pelo **pgAdmin** e crie:

- Name SGBD Pgadmin: **Lab5**
- Schema: **dsalabdw**

---

### Preparando os Containers Docker para o Apache Airflow

1. Crie uma pasta vazia na raiz com o nome `Airflow` na máquina local.
2. Navegue até a pasta `Airflow` usando o terminal ou CMD.
3. Siga a documentação oficial do Airflow no link:  
   [Documentação do Airflow com Docker](https://airflow.apache.org/docs/apache-airflow/stable/howto/docker-compose/index.html)
   
Execute os seguintes comandos dentro da pasta `Airflow`:



curl -LfO 'https://airflow.apache.org/docs/apache-airflow/2.4.3/docker-compose.yaml'

mkdir -p ./dags ./logs ./plugins ./config

echo -e "AIRFLOW_UID=$(id -u)" > .env

docker compose up airflow-init

docker compose up


Agora, com a rede de containers em execução, acesse o painel do Airflow via `localhost:8080` usando as credenciais padrão:

- **User**: airflow  
- **Password**: airflow

---

### Configurando a Comunicação entre Containers (Docker PostgreSQL e Airflow)

#### Passo 1: Listar as Redes Docker

docker network ls


#### Passo 2: Inspecionar o Container do Banco de Dados


docker inspect dbdsa


#### Passo 3: Extrair Detalhes sobre a Rede do Container


docker inspect dbdsa -f "{{json .NetworkSettings.Networks }}"


#### Passo 4: Inspecionar a Rede de Todos os Containers Simultaneamente


docker ps --format '{{ .ID }} {{ .Names }} {{ json .Networks }}'


#### Passo 5: Inspecionar a Rede do Airflow e da Rede `bridge`


docker network inspect airflow_default

docker network inspect bridge


#### Passo 6: Instalar Ferramentas de Rede no Container para Testar Conexão

Dentro do container do PostgreSQL, execute os comandos:

apt-get update

apt-get install net-tools

apt-get install iputils-ping


#### Passo 7: Fazer Teste de Conexão entre o Container `dbdsa` e o Webserver do Apache Airflow


ifconfig

ping


#### Passo 8: Colocar o Banco de Dados na Mesma Rede do Airflow

1. Desconectar o container `dbdsa` da rede `bridge`:

docker network disconnect bridge dbdsa


2. Conectar o container `dbdsa` na rede do Airflow:

docker network connect airflow_default dbdsa

---

### Criar a Conexão no Airflow

Ao acessar o Apache Airflow é necessário criar a conexão (menu > connetcion): 

- **Name connetion id**: Lab5DW  
- Preencher os campos com os seguintes valores:
  - **Host**: Utilize o comando `ifconfig` para adicionar o INET (IP da máquina do SGBD)
  - **Schema**: Nome do banco de dados
  - **Port**: Porta do container Docker

---



## Job ETL (Arquivo: `job_etl_lab5`)
```python

# Imports
import airflow
from datetime import timedelta
from airflow import DAG
from airflow.operators.postgres_operator import PostgresOperator
from airflow.utils.dates import days_ago

# Argumentos
args = {'owner': 'airflow'}

# Argumentos default
default_args = {
    'owner': 'airflow',    
    #'start_date': airflow.utils.dates.days_ago(2),
    #'end_date': datetime(),
    #'depends_on_past': False,
    #'email': ['airflow@example.com'],
    #'email_on_failure': False,
    #'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes = 5),
}

# Cria a DAG
dag_lab5_dsa = DAG(dag_id = "Lab5",
                   default_args = args,
                   # schedule_interval='0 0 * * *',
                   schedule_interval = '@once',  
                   dagrun_timeout = timedelta(minutes = 60),
                   description = 'Job ETL de Carga no DW com Airflow',
                   start_date = airflow.utils.dates.days_ago(1)
)

# Instrução SQL de criação de tabela
sql_cria_tabela = """CREATE TABLE IF NOT EXISTS tb_funcionarios (id INT NOT NULL, nome VARCHAR(250) NOT NULL, departamento VARCHAR(250) NOT NULL);"""

# Tarefa de criação da tabela
cria_tabela = PostgresOperator(sql = sql_cria_tabela,
                               task_id = "tarefa_cria_tabela",
                               postgres_conn_id = "Lab5DW",
                               dag = dag_lab5_dsa
)

# Instrução SQL de insert na tabela
sql_insere_dados = """
insert into tb_funcionarios (id, nome, departamento) values (1000, 'Bob', 'Marketing'), (1001, 'Maria', 'Contabilidade'),(1002, 'Jeremias', 'Engenharia de Dados'), (1003, 'Messi', 'Marketing') ;"""

# Tarefa de insert na tabela
insere_dados = PostgresOperator(sql = sql_insere_dados,
                                task_id = "tarefa_insere_dados",
                                postgres_conn_id = "Lab5DW",
                                dag = dag_lab5_dsa
)

# Fluxo da DAG
cria_tabela >> insere_dados

# Bloco main
if __name__ == "__main__":
    dag_lab5_dsa.cli()

```

### Observações Finais:

* Coloque o arquivo job_etl_lab5 dentro da pasta AIRFLOW/dag criada na raiz da máquina local.
  
* Assim que o arquivo estiver na pasta correta, a DAG automaticamente irá aparecer na interface do Airflow (porta 8080).
  
* Dispare a trigger da DAG no Airflow para que os dados sejam criados e inseridos no PostgreSQL.


---
## Contato

Se tiver dúvidas ou sugestões sobre o projeto, entre em contato comigo:

- [LinkedIn](https://www.linkedin.com/in/henrique-k-32967a2b5/)
- [GitHub](https://github.com/henriquekurata?tab=overview&from=2024-09-01&to=2024-09-01)