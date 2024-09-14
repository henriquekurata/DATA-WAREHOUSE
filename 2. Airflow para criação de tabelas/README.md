# ***Automatização com Apache Airflow para criar e inserir dados no PostgreSQL com linguagem Python***

## **Descrição do Projeto:**
Este projeto tem como objetivo criar e automatizar um pipeline de dados usando o **Apache Airflow** e **Python** para inserir dados no **PostgreSQL**. 

## **Tecnologias Utilizadas**:
- Docker;
- PostgreSQL;
- pgAdmin;
- Apache Airflow;
- Anaconda.

## **Resumo**: 
* Criar imagem e container para o banco de dados do DW;
* Criar imagem e containers para o Apache Airflow;
* Configurar a comunicação entre as redes de containers (PostgreSQL e Airflow);
* Criar a connetion ID no Airlow;
* Criar a DAG;
* Inserir a DAG dentro da pasta raiz na máquina local do Airflow;
* Disparar a DAG.

## **Comandos**:

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

Com a rede de container em execução, agora é só acessar a porta localhost:8080 com as credenciais abaixo:

User: airflow e Password: airflow


### Configurando a rede de containers Docker para o SGBD e o Airflow se comunicarem:

Listar as redes Docker:

docker network ls

Inspecionar o container do banco de dados:

docker inspect dbdsa

Extrair detalhes sobre a rede do container:

docker inspect dbdsa -f "{{json .NetworkSettings.Networks }}"

Inspecionar a rede de todos os containers ao mesmo tempo:

docker ps --format '{{ .ID }} {{ .Names }} {{ json .Networks }}'

Inspecionar a rede do Airflow e a rede padrão bridge:

docker network inspect airflow_default

docker network inspect bridge

Instalar ferramentas de rede no container para testar a conexão:

apt-get update

apt-get install net-tools

apt-get install iputils-ping

Fazer o teste de conexão entre o container dbdsa e o webserver do Apache Airflow:

ifconfig

ping

Para deixar tudo no mesmo ambiente de rede é necessário seguir os passos abaixo:

Desconectar o container da rede atual:

docker network disconnect bridge dbdsa

Conectar o container na rede desejada

docker network connect airflow_default dbdsa

Inspecionar a rede de todos os containers ao mesmo tempo:

docker ps --format '{{ .ID }} {{ .Names }} {{ json .Networks }}'

Extrair detalhes sobre a rede do container:

docker inspect dbdsa -f "{{json .NetworkSettings.Networks }}"

Inspecionar a rede do Airflow:

docker network inspect airflow_default

Ao acessar o Apache Airflow é necessário criar a conexão (menu > connetcion): 

Name connetion id: Lab5DW

Observações para preenchimento da connection no Airflow:

Host = usar comando ifconfig e add o INET (IP máquina do SGBD)

Schema = Nome do banco de dados

PORT = Porta do container Docker


## Job ETL (nome do arquivo: job_etl_lab5)
```

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

Obs: Precisa levar o arquivo "job_etl_lab5" para a pasta "AIRFLOW" > "dag", ambas criadas na raiz da máquina local
Assim que o arquivo estiver na pasta raiz a DAG automaticamente irá aparecer na interface do Airflow (DAG), na porta 8080

Agora é só disparar a trigger da Dag no Airflow para que os dados sejam criados e inseridos no PostgreSQL

