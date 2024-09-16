# 🚀 ***Pipeline de dados com Apache Airflow***


## **Descrição do Projeto:**
Este projeto consiste na construção de um pipeline de dados utilizando **Apache Airflow** para orquestrar o processo de ETL (Extract, Transform, Load), que realiza a leitura de arquivos CSV armazenados na máquina local e carrega esses dados em um banco de dados **PostgreSQL**. A arquitetura do pipeline foi desenhada para simular a automação de processos comuns em ambientes de Data Warehousing, utilizando contêineres **Docker** para garantir a escalabilidade e reprodutibilidade do ambiente, facilitando o gerenciamento de dependências e serviços.

## 🛠️ **Tecnologias Utilizadas**:

- **Apache Airflow**: Orquestrador responsável pela automação do pipeline de dados, gerenciamento das DAGs e monitoramento das tarefas.
- **Docker**: Ferramenta de contêinerização usada para criar ambientes isolados para os serviços do projeto, como o PostgreSQL e o Airflow.
- **PostgreSQL**: Banco de dados relacional que armazena os dados processados, representando o destino final do pipeline ETL.
- **pgAdmin**: Interface de gerenciamento para o PostgreSQL, utilizada para a visualização e execução de queries no banco de dados.
- **Python**: Linguagem de programação utilizada para implementar as tarefas de transformação e carga no Airflow, incluindo a interação com o PostgreSQL.
- **Anaconda**: Distribuição de pacotes Python que facilita a criação e gerenciamento de ambientes, garantindo que todas as dependências necessárias para o projeto sejam facilmente configuradas.


## Principais Funcionalidades
1. **Automação de Tarefas**: Através de DAGs (Directed Acyclic Graphs), o Apache Airflow gerencia a execução das tarefas, incluindo leitura dos arquivos CSV, transformação dos dados e sua inserção em um banco de dados PostgreSQL.
2. **Execução Escalonável**: O ambiente é configurado utilizando **Docker** para garantir que todos os serviços rodem em contêineres independentes, facilitando o gerenciamento de dependências e garantindo a consistência entre diferentes execuções.
3. **Orquestração Programática**: O pipeline é totalmente controlado por código, o que permite a criação de tarefas dinâmicas, como a limpeza e carregamento de tabelas, por meio do Airflow. Além disso, o agendamento automático das execuções pode ser configurado com cron jobs.
4. **Modelagem de Dados**: O projeto inclui a criação de um esquema de Data Warehouse, com tabelas dimensionais e de fato, proporcionando um exemplo prático de como armazenar dados de forma organizada para futuras consultas analíticas.


## 📋 **Descrição do Processo** 

- Criar a estrutura das tabelas direto no DW com SQL.
- Criar a connection no Airflow.
- Criar a DAG.
- Inserir a DAG dentro da pasta raiz na máquina local do Airflow.
- Inserir os arquivos de dados CSV na máquina local dentro da pasta raiz (`AIRFLOW > dags > dados`).
- Disparar a DAG.

## **Comandos**:

### Configurar o Pgadmin:
Acesse o pgAdmin e crie um banco de dados e schema com os seguintes nomes:
- **Name SGBD pgAdmin**: DW-Lab 6
- **Schema**: lab6

---

### Criar a conncection no Airflow
- **Name**: Lab6DW

---

### Criar as estruturas das tabelas:
Dentro do schema, execute os scripts SQL a seguir:

```sql
CREATE TABLE lab6.DIM_CLIENTE
(
    id_cliente int NOT NULL,
    nome_cliente text,
    sobrenome_cliente text,
    PRIMARY KEY (id_cliente)
);


CREATE TABLE lab6.DIM_TRANSPORTADORA
(
    id_transportadora integer NOT NULL,
    nome_transportadora text,
    PRIMARY KEY (id_transportadora)
);


CREATE TABLE lab6.DIM_DEPOSITO
(
    id_deposito bigint NOT NULL,
    nome_deposito text,
    PRIMARY KEY (id_deposito)
);


CREATE TABLE lab6.DIM_ENTREGA
(
    id_entrega bigint NOT NULL,
    endereco_entrega text,
    pais_entrega text,
    PRIMARY KEY (id_entrega)
);


CREATE TABLE lab6.DIM_PAGAMENTO
(
    id_pagamento bigint NOT NULL,
    tipo_pagamento text,
    PRIMARY KEY (id_pagamento)
);


CREATE TABLE lab6.DIM_FRETE
(
    id_frete bigint NOT NULL,
    tipo_frete text,
    PRIMARY KEY (id_frete)
);


CREATE TABLE lab6.DIM_DATA
(
    id_data bigint NOT NULL,
    data_completa text,
    dia integer,
    mes integer,
    ano integer,
    PRIMARY KEY (id_data)
);


CREATE TABLE lab6.TB_FATO
(
    id_cliente integer,
    id_transportadora integer,
    id_deposito integer,
    id_entrega integer,
    id_pagamento integer,
    id_frete integer,
    id_data integer,
    valor_entrega double precision,
    PRIMARY KEY (id_cliente, id_transportadora, id_deposito, id_entrega, id_pagamento, id_frete, id_data)
);

```
--- 

### Observação:

Como estamos usando Docker, é necessário apontar o mapeamento de volumes para os arquivos das tabelas dimensões e fato no módulo Python, por exemplo: `op_kwargs = {'params': {'csv_file_path': '/opt/airflow/dags/dados/DIM_CLIENTE.csv'}}`.

O caminho `/opt/airflow/dags/dados` é a pasta criada no diretório raiz da máquina local.

---

### Job ETL - Apache Airflow (etl_dw_v5)

```python
#Imports
import csv
import airflow
import time
import pandas as pd
from datetime import datetime
from datetime import timedelta
from airflow import DAG
from airflow.operators.python_operator import PythonOperator
from airflow.operators.postgres_operator import PostgresOperator
from airflow.utils.dates import days_ago

# Argumentos
default_args = {
    'owner': 'airflow',
    'start_date': datetime(2023, 1, 1),
    'depends_on_past': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}

# Cria a DAG
# https://crontab.guru/
dag_lab6_dsa = DAG(dag_id = "lab6_final",
                   default_args = default_args,
                   schedule_interval = '0 0 * * *',
                   dagrun_timeout = timedelta(minutes = 60),
                   description = 'Job ETL de Carga no DW com Airflow',
                   start_date = airflow.utils.dates.days_ago(1)
)

##### Tabela de Clientes #####

def func_carrega_dados_clientes(**kwargs):
    
    # Get the csv file path
    csv_file_path = kwargs['params']['csv_file_path']

    # Inicializa o contador
    i = 0

    # Open the csv file
    with open(csv_file_path, 'r') as f:

        reader = csv.DictReader(f)

        for item in reader:

            # Icrementa o contador
            i += 1
            
            # Extrai uma linha como dicionário
            dados_cli = dict(item)

            # Insert data into the PostgreSQL table
            sql_query_cli = "INSERT INTO lab6.DIM_CLIENTE (%s) VALUES (%s)" % (','.join(dados_cli.keys()), ','.join([item for item in dados_cli.values()]))
    
            # Operador do Postgres com incremento no id da tarefa (para cada linha inserida)
            postgres_operator = PostgresOperator(task_id = 'carrega_dados_clientes_' + str(i), 
                                                 sql = sql_query_cli, 
                                                 params = (dados_cli), 
                                                 postgres_conn_id = 'Lab6DW', 
                                                 dag = dag_lab6_dsa)
    
            # Executa o operador
            postgres_operator.execute(context = kwargs)


tarefa_carrega_dados_clientes = PythonOperator(
        task_id = 'tarefa_carrega_dados_clientes',
        python_callable = func_carrega_dados_clientes,
        provide_context = True,
        op_kwargs = {'params': {'csv_file_path': '/opt/airflow/dags/dados/DIM_CLIENTE.csv'}},
        dag = dag_lab6_dsa
    )


##### Tabela de Transportadoras #####

def func_carrega_dados_transp(**kwargs):
    
    # Get the csv file path
    csv_file_path = kwargs['params']['csv_file_path']

    # Inicializa o contador
    i = 0

    # Open the csv file
    with open(csv_file_path, 'r') as f:

        reader = csv.DictReader(f)

        for item in reader:

            # Icrementa o contador
            i += 1
            
            # Extrai uma linha como dicionário
            dados_transp = dict(item)

            # Insert data into the PostgreSQL table
            sql_query_transp = "INSERT INTO lab6.DIM_TRANSPORTADORA (%s) VALUES (%s)" % (','.join(dados_transp.keys()), ','.join([item for item in dados_transp.values()]))
    
            # Operador do Postgres com incremento no id da tarefa (para cada linha inserida)
            postgres_operator = PostgresOperator(task_id = 'carrega_dados_transp_' + str(i), 
                                                 sql = sql_query_transp, 
                                                 params = (dados_transp), 
                                                 postgres_conn_id = 'Lab6DW', 
                                                 dag = dag_lab6_dsa)
    
            # Executa o operador
            postgres_operator.execute(context = kwargs)


tarefa_carrega_dados_transportadora = PythonOperator(
        task_id = 'tarefa_carrega_dados_transportadora',
        python_callable = func_carrega_dados_transp,
        provide_context = True,
        op_kwargs = {'params': {'csv_file_path': '/opt/airflow/dags/dados/DIM_TRANSPORTADORA.csv'}},
        dag = dag_lab6_dsa
    )

##### Tabela de Depósitos #####

def func_carrega_dados_dep(**kwargs):
    
    # Get the csv file path
    csv_file_path = kwargs['params']['csv_file_path']

    # Inicializa o contador
    i = 0

    # Open the csv file
    with open(csv_file_path, 'r') as f:

        reader = csv.DictReader(f)

        for item in reader:

            # Icrementa o contador
            i += 1
            
            # Extrai uma linha como dicionário
            dados_dep = dict(item)

            # Insert data into the PostgreSQL table
            sql_query_dep = "INSERT INTO lab6.DIM_DEPOSITO (%s) VALUES (%s)" % (','.join(dados_dep.keys()), ','.join([item for item in dados_dep.values()]))
    
            # Operador do Postgres com incremento no id da tarefa (para cada linha inserida)
            postgres_operator = PostgresOperator(task_id = 'carrega_dados_dep_' + str(i), 
                                                 sql = sql_query_dep, 
                                                 params = (dados_dep), 
                                                 postgres_conn_id = 'Lab6DW', 
                                                 dag = dag_lab6_dsa)
    
            # Executa o operador
            postgres_operator.execute(context = kwargs)


tarefa_carrega_dados_deposito = PythonOperator(
        task_id = 'tarefa_carrega_dados_deposito',
        python_callable = func_carrega_dados_dep,
        provide_context = True,
        op_kwargs = {'params': {'csv_file_path': '/opt/airflow/dags/dados/DIM_DEPOSITO.csv'}},
        dag = dag_lab6_dsa
    )

##### Tabela de Entregas #####

def func_carrega_dados_ent(**kwargs):
    
    # Get the csv file path
    csv_file_path = kwargs['params']['csv_file_path']

    # Inicializa o contador
    i = 0

    # Open the csv file
    with open(csv_file_path, 'r') as f:

        reader = csv.DictReader(f)

        for item in reader:

            # Icrementa o contador
            i += 1
            
            # Extrai uma linha como dicionário
            dados_ent = dict(item)

            # Insert data into the PostgreSQL table
            sql_query_ent = "INSERT INTO lab6.DIM_ENTREGA (%s) VALUES (%s)" % (','.join(dados_ent.keys()), ','.join([item for item in dados_ent.values()]))
    
            # Operador do Postgres com incremento no id da tarefa (para cada linha inserida)
            postgres_operator = PostgresOperator(task_id = 'carrega_dados_ent_' + str(i), 
                                                 sql = sql_query_ent, 
                                                 params = (dados_ent), 
                                                 postgres_conn_id = 'Lab6DW', 
                                                 dag = dag_lab6_dsa)
    
            # Executa o operador
            postgres_operator.execute(context = kwargs)


tarefa_carrega_dados_entrega = PythonOperator(
        task_id = 'tarefa_carrega_dados_entrega',
        python_callable = func_carrega_dados_ent,
        provide_context = True,
        op_kwargs = {'params': {'csv_file_path': '/opt/airflow/dags/dados/DIM_ENTREGA.csv'}},
        dag = dag_lab6_dsa
    )

##### Tabela de Frete #####

def func_carrega_dados_frete(**kwargs):
    
    # Get the csv file path
    csv_file_path = kwargs['params']['csv_file_path']

    # Inicializa o contador
    i = 0

    # Open the csv file
    with open(csv_file_path, 'r') as f:

        reader = csv.DictReader(f)

        for item in reader:

            # Icrementa o contador
            i += 1
            
            # Extrai uma linha como dicionário
            dados_frete = dict(item)

            # Insert data into the PostgreSQL table
            sql_query_frete = "INSERT INTO lab6.DIM_FRETE (%s) VALUES (%s)" % (','.join(dados_frete.keys()), ','.join([item for item in dados_frete.values()]))
    
            # Operador do Postgres com incremento no id da tarefa (para cada linha inserida)
            postgres_operator = PostgresOperator(task_id = 'carrega_dados_frete_' + str(i), 
                                                 sql = sql_query_frete, 
                                                 params = (dados_frete), 
                                                 postgres_conn_id = 'Lab6DW', 
                                                 dag = dag_lab6_dsa)
    
            # Executa o operador
            postgres_operator.execute(context = kwargs)


tarefa_carrega_dados_frete = PythonOperator(
        task_id = 'tarefa_carrega_dados_frete',
        python_callable = func_carrega_dados_frete,
        provide_context = True,
        op_kwargs = {'params': {'csv_file_path': '/opt/airflow/dags/dados/DIM_FRETE.csv'}},
        dag = dag_lab6_dsa
    )

##### Tabela de Tipos de Pagamentos #####

def func_carrega_dados_pagamento(**kwargs):
    
    # Get the csv file path
    csv_file_path = kwargs['params']['csv_file_path']

    # Inicializa o contador
    i = 0

    # Open the csv file
    with open(csv_file_path, 'r') as f:

        reader = csv.DictReader(f)

        for item in reader:

            # Icrementa o contador
            i += 1
            
            # Extrai uma linha como dicionário
            dados_pag = dict(item)

            # Insert data into the PostgreSQL table
            sql_query_pag = "INSERT INTO lab6.DIM_PAGAMENTO (%s) VALUES (%s)" % (','.join(dados_pag.keys()), ','.join([item for item in dados_pag.values()]))
    
            # Operador do Postgres com incremento no id da tarefa (para cada linha inserida)
            postgres_operator = PostgresOperator(task_id = 'carrega_dados_pag_' + str(i), 
                                                 sql = sql_query_pag, 
                                                 params = (dados_pag), 
                                                 postgres_conn_id = 'Lab6DW', 
                                                 dag = dag_lab6_dsa)
    
            # Executa o operador
            postgres_operator.execute(context = kwargs)


tarefa_carrega_dados_pagamento = PythonOperator(
        task_id = 'tarefa_carrega_dados_pagamento',
        python_callable = func_carrega_dados_pagamento,
        provide_context = True,
        op_kwargs = {'params': {'csv_file_path': '/opt/airflow/dags/dados/DIM_PAGAMENTO.csv'}},
        dag = dag_lab6_dsa
    )

##### Tabela de Data #####

def func_carrega_dados_data(**kwargs):
    
    # Get the csv file path
    csv_file_path = kwargs['params']['csv_file_path']

    # Inicializa o contador
    i = 0

    # Open the csv file
    with open(csv_file_path, 'r') as f:

        reader = csv.DictReader(f)

        for item in reader:

            # Icrementa o contador
            i += 1
            
            # Extrai uma linha como dicionário
            dados_data = dict(item)

            # Insert data into the PostgreSQL table
            sql_query_data = "INSERT INTO lab6.DIM_DATA (%s) VALUES (%s)" % (','.join(dados_data.keys()), ','.join([item for item in dados_data.values()]))
    
            # Operador do Postgres com incremento no id da tarefa (para cada linha inserida)
            postgres_operator = PostgresOperator(task_id = 'carrega_dados_data_' + str(i), 
                                                 sql = sql_query_data, 
                                                 params = (dados_data), 
                                                 postgres_conn_id = 'Lab6DW', 
                                                 dag = dag_lab6_dsa)
    
            # Executa o operador
            postgres_operator.execute(context = kwargs)


tarefa_carrega_dados_data = PythonOperator(
        task_id = 'tarefa_carrega_dados_data',
        python_callable = func_carrega_dados_data,
        provide_context = True,
        op_kwargs = {'params': {'csv_file_path': '/opt/airflow/dags/dados/DIM_DATA.csv'}},
        dag = dag_lab6_dsa
    )

##### Tabela de Fatos #####

def func_carrega_dados_fatos(**kwargs):
    
    # Get the csv file path
    csv_file_path = kwargs['params']['csv_file_path']

    # Inicializa o contador
    i = 0

    # Open the csv file
    with open(csv_file_path, 'r') as f:

        reader = csv.DictReader(f)

        for item in reader:

            # Icrementa o contador
            i += 1
            
            # Extrai uma linha como dicionário
            dados_fatos = dict(item)

            # Insert data into the PostgreSQL table
            sql_query_fatos = "INSERT INTO lab6.TB_FATO (%s) VALUES (%s)" % (','.join(dados_fatos.keys()), ','.join([item for item in dados_fatos.values()]))
    
            # Operador do Postgres com incremento no id da tarefa (para cada linha inserida)
            postgres_operator = PostgresOperator(task_id = 'carrega_dados_fatos_' + str(i), 
                                                 sql = sql_query_fatos, 
                                                 params = (dados_fatos), 
                                                 postgres_conn_id = 'Lab6DW', 
                                                 dag = dag_lab6_dsa)
    
            # Executa o operador
            postgres_operator.execute(context = kwargs)


tarefa_carrega_dados_fatos = PythonOperator(
        task_id = 'tarefa_carrega_dados_fatos',
        python_callable = func_carrega_dados_fatos,
        provide_context = True,
        op_kwargs = {'params': {'csv_file_path': '/opt/airflow/dags/dados/TB_FATO.csv'}},
        dag = dag_lab6_dsa
    )


# Tarefas para limpar as tabelas
tarefa_trunca_tb_fato = PostgresOperator(task_id = 'tarefa_trunca_tb_fato', postgres_conn_id = 'Lab6DW', sql = "TRUNCATE TABLE lab6.TB_FATO CASCADE", dag = dag_lab6_dsa)
tarefa_trunca_dim_cliente = PostgresOperator(task_id = 'tarefa_trunca_dim_cliente', postgres_conn_id = 'Lab6DW', sql = "TRUNCATE TABLE lab6.DIM_CLIENTE CASCADE", dag = dag_lab6_dsa)
tarefa_trunca_dim_pagamento = PostgresOperator(task_id = 'tarefa_trunca_dim_pagamento', postgres_conn_id = 'Lab6DW', sql = "TRUNCATE TABLE lab6.DIM_PAGAMENTO CASCADE", dag = dag_lab6_dsa)
tarefa_trunca_dim_frete = PostgresOperator(task_id = 'tarefa_trunca_dim_frete', postgres_conn_id = 'Lab6DW', sql = "TRUNCATE TABLE lab6.DIM_FRETE CASCADE", dag = dag_lab6_dsa)
tarefa_trunca_dim_data = PostgresOperator(task_id = 'tarefa_trunca_dim_data', postgres_conn_id = 'Lab6DW', sql = "TRUNCATE TABLE lab6.DIM_DATA CASCADE", dag = dag_lab6_dsa)
tarefa_trunca_dim_transportadora = PostgresOperator(task_id = 'tarefa_trunca_dim_transportadora', postgres_conn_id = 'Lab6DW', sql = "TRUNCATE TABLE lab6.DIM_TRANSPORTADORA CASCADE", dag = dag_lab6_dsa)
tarefa_trunca_dim_entrega = PostgresOperator(task_id = 'tarefa_trunca_dim_entrega', postgres_conn_id = 'Lab6DW', sql = "TRUNCATE TABLE lab6.DIM_ENTREGA CASCADE", dag = dag_lab6_dsa)
tarefa_trunca_dim_deposito = PostgresOperator(task_id = 'tarefa_trunca_dim_deposito', postgres_conn_id = 'Lab6DW', sql = "TRUNCATE TABLE lab6.DIM_DEPOSITO CASCADE", dag = dag_lab6_dsa)


# Upstream
tarefa_trunca_tb_fato >> tarefa_trunca_dim_cliente >> tarefa_trunca_dim_pagamento >> tarefa_trunca_dim_frete >> tarefa_trunca_dim_data >> tarefa_trunca_dim_transportadora >> tarefa_trunca_dim_entrega >> tarefa_trunca_dim_deposito >> tarefa_carrega_dados_clientes >> tarefa_carrega_dados_transportadora >> tarefa_carrega_dados_deposito >> tarefa_carrega_dados_entrega >> tarefa_carrega_dados_frete >> tarefa_carrega_dados_pagamento >> tarefa_carrega_dados_data >> tarefa_carrega_dados_fatos


# Bloco main
if __name__ == "__main__":
    dag_lab6_dsa.cli()

```


---
## Contato

Se tiver dúvidas ou sugestões sobre o projeto, entre em contato comigo:

- [LinkedIn](https://www.linkedin.com/in/henrique-k-32967a2b5/)
- [GitHub](https://github.com/henriquekurata?tab=overview&from=2024-09-01&to=2024-09-01)