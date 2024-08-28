# ***Implementação de DW no PostgreSQL e processo ETL utilizando Airbyte/SQL***



## **Ferramentas:**

Docker, PostgreSQL, pgAdmin, Airbyte e SQL.



## **Passos:** 
* Criar e inserir fonte de dados no SGBD com SQL;
* criar a conexão e o destino com Airbyte da fonte de dados (schema 1 = servidor 1) para a SA (schema 2 = servidor 2);
* Linguegem SQL para criar a estrutura das tabelas do DW;
* Linguagem SQL para inserir os dados no DW;
* Utilizar o Group By para integridade dos resultados;
* Adicionar métrica na tabela fato;
* Criar View X Materialized View.



Obs: Para esse processo ETL vamos utilizar linguagem SQL para servidores iguais e o Airbyte para servidores distintos.
O servidor de origem dos dados deve receber a menor sobrecarga possível para nao comprometer os bancos transacionais, portanto a fonte estará no servidor 1 e a staging area e o destino no servidor 2.

#Servidor 1: Fonte
Name SGBD Pgadmin: dbdsafonte
Schema:schema 1
Container docker: dbdsafonte

#Servidor 2: Stating Area (Airbyte):
Name SGBD Pgadmin: dbdsadestino
Schema: schema 2
Container docker: airbyte


#Servidor 2: Destino
Name SGBD Pgadmin: dbdsadestino 
Schema: schema 3
Container docker: dbdsadestino



## **Comandos:**

### Criar o container para a fonte e o destino dos dados 

docker run --name dbdsafonte -p 5433:5432 -e POSTGRES_USER=dbadmin -e POSTGRES_PASSWORD=dbadmin123 -e POSTGRES_DB=postgresDB -d postgres

docker run --name dbdsadestino -p 5434:5432 -e POSTGRES_USER=dbadmin -e POSTGRES_PASSWORD=dbadmin123 -e POSTGRES_DB=postgresDB -d postgres

### Criar as tabelas da fonte diretamente no SGBD (schema 1) com SQL

```
CREATE TABLE schema1.ft_categorias (
    id_categoria SERIAL PRIMARY KEY,
    nome_categoria VARCHAR(255) NOT NULL
);

INSERT INTO schema1.ft_categorias (nome_categoria) VALUES ('Computadores');
INSERT INTO schema1.ft_categorias (nome_categoria) VALUES ('Smartphones');
INSERT INTO schema1.ft_categorias (nome_categoria) VALUES ('Impressoras');

CREATE TABLE schema1.ft_subcategorias (
    id_subcategoria SERIAL PRIMARY KEY,
    nome_subcategoria VARCHAR(255) NOT NULL,
    id_categoria INTEGER REFERENCES schema1.ft_categorias(id_categoria)
);

INSERT INTO schema1.ft_subcategorias (nome_subcategoria, id_categoria) VALUES ('Notebook', 1);
INSERT INTO schema1.ft_subcategorias (nome_subcategoria, id_categoria) VALUES ('Desktop', 1);
INSERT INTO schema1.ft_subcategorias (nome_subcategoria, id_categoria) VALUES ('iPhone', 2);
INSERT INTO schema1.ft_subcategorias (nome_subcategoria, id_categoria) VALUES ('Samsung Galaxy', 2);
INSERT INTO schema1.ft_subcategorias (nome_subcategoria, id_categoria) VALUES ('Laser', 3);
INSERT INTO schema1.ft_subcategorias (nome_subcategoria, id_categoria) VALUES ('Matricial', 3);

CREATE TABLE schema1.ft_produtos (
    id_produto SERIAL PRIMARY KEY,
    nome_produto VARCHAR(255) NOT NULL,
    preco_produto NUMERIC(10,2) NOT NULL,
    id_subcategoria INTEGER REFERENCES schema1.ft_subcategorias(id_subcategoria)
);

INSERT INTO schema1.ft_produtos (nome_produto, preco_produto, id_subcategoria) VALUES ('Apple MacBook Pro M2', 6589.99, 1);
INSERT INTO schema1.ft_produtos (nome_produto, preco_produto, id_subcategoria) VALUES ('Desktop Dell 16 GB', 1500.50, 1);
INSERT INTO schema1.ft_produtos (nome_produto, preco_produto, id_subcategoria) VALUES ('iPhone 14', 4140.00, 2);
INSERT INTO schema1.ft_produtos (nome_produto, preco_produto, id_subcategoria) VALUES ('Samsung Galaxy Z', 3500.99, 2);
INSERT INTO schema1.ft_produtos (nome_produto, preco_produto, id_subcategoria) VALUES ('HP 126A Original LaserJet Imaging Drum', 300.90, 3);
INSERT INTO schema1.ft_produtos (nome_produto, preco_produto, id_subcategoria) VALUES ('Epson LX-300 II USB', 350.99, 3);

CREATE TABLE schema1.ft_cidades (
    id_cidade SERIAL PRIMARY KEY,
    nome_cidade VARCHAR(255) NOT NULL
);

INSERT INTO schema1.ft_cidades (nome_cidade) VALUES
    ('Natal'),
    ('Rio de Janeiro'),
    ('Belo Horizonte'),
    ('Salvador'),
    ('Blumenau'),
    ('Curitiba'),
    ('Fortaleza'),
    ('Recife'),
    ('Porto Alegre'),
    ('Manaus');

CREATE TABLE schema1.ft_localidades (
    id_localidade SERIAL PRIMARY KEY,
    pais VARCHAR(255) NOT NULL,
    regiao VARCHAR(255) NOT NULL,
    id_cidade INTEGER REFERENCES schema1.ft_cidades(id_cidade)
);

INSERT INTO schema1.ft_localidades (pais, regiao, id_cidade) VALUES
    ('Brasil', 'Nordeste', 1),
    ('Brasil', 'Sudeste', 2),
    ('Brasil', 'Sudeste', 3),
    ('Brasil', 'Nordeste', 4),
    ('Brasil', 'Sul', 5),
    ('Brasil', 'Sul', 6),
    ('Brasil', 'Nordeste', 7),
    ('Brasil', 'Nordeste', 8),
    ('Brasil', 'Sul', 9),
    ('Brasil', 'Norte', 10);

CREATE TABLE schema1.ft_tipo_cliente (
    id_tipo SERIAL PRIMARY KEY,
    nome_tipo VARCHAR(255) NOT NULL
);

INSERT INTO schema1.ft_tipo_cliente (nome_tipo) VALUES ('Corporativo');
INSERT INTO schema1.ft_tipo_cliente (nome_tipo) VALUES ('Consumidor');
INSERT INTO schema1.ft_tipo_cliente (nome_tipo) VALUES ('Desativado');

CREATE TABLE schema1.ft_clientes (
    id_cliente SERIAL PRIMARY KEY,
    nome_cliente VARCHAR(255) NULL,
    email_cliente VARCHAR(255) NULL,
    id_cidade INTEGER REFERENCES schema1.ft_cidades(id_cidade),
    id_tipo INTEGER REFERENCES schema1.ft_tipo_cliente(id_tipo)
);

INSERT INTO schema1.ft_clientes (nome_cliente, email_cliente, id_cidade, id_tipo) VALUES ('João Silva', 'joao.silva@exemplo.com', 1, 1);
INSERT INTO schema1.ft_clientes (nome_cliente, email_cliente, id_cidade, id_tipo) VALUES ('Maria Santos', 'maria.santos@exemplo.com', 2, 2);
INSERT INTO schema1.ft_clientes (nome_cliente, email_cliente, id_cidade, id_tipo) VALUES ('Pedro Lima', 'pedro.lima@exemplo.com', 3, 2);
INSERT INTO schema1.ft_clientes (nome_cliente, email_cliente, id_cidade, id_tipo) VALUES ('Ana Rodrigues', 'ana.rodrigues@exemplo.com', 4, 2);
INSERT INTO schema1.ft_clientes (nome_cliente, email_cliente, id_cidade, id_tipo) VALUES ('José Oliveira', 'jose.oliveira@exemplo.com', 1, 2);
INSERT INTO schema1.ft_clientes (nome_cliente, email_cliente, id_cidade, id_tipo) VALUES ('Carla Santos', 'carla.santos@exemplo.com', 4, 1);
INSERT INTO schema1.ft_clientes (nome_cliente, email_cliente, id_cidade, id_tipo) VALUES ('Marcos Souza', 'marcos.souza@exemplo.com', 5, 2);
INSERT INTO schema1.ft_clientes (nome_cliente, email_cliente, id_cidade, id_tipo) VALUES ('Julia Silva', 'julia.silva@exemplo.com', 1, 1);
INSERT INTO schema1.ft_clientes (nome_cliente, email_cliente, id_cidade, id_tipo) VALUES ('Lucas Martins', 'lucas.martins@exemplo.com', 3, 3);
INSERT INTO schema1.ft_clientes (nome_cliente, email_cliente, id_cidade, id_tipo) VALUES ('Fernanda Lima', 'fernanda.lima@exemplo.com', 4, 2);


CREATE TABLE schema1.ft_vendas (
  id_transacao VARCHAR(50) NOT NULL,
  id_produto INT NOT NULL,
  id_cliente INT NOT NULL,
  id_localizacao INT NOT NULL,
  data_transacao DATE NULL,
  quantidade INT NOT NULL,
  preco_venda DECIMAL(10,2) NOT NULL,
  custo_produto DECIMAL(10,2) NOT NULL
);

-- Gerar valores aleatórios para as colunas
WITH dados AS (
  SELECT 
    floor(random() * 1000000)::text AS id_transacao,
    floor(random() * 6 + 1) AS id_produto,
    floor(random() * 10 + 1) AS id_cliente,
    floor(random() * 4 + 1) AS id_localizacao,
    '2022-01-01'::date + floor(random() * 365)::integer AS data_transacao,
    floor(random() * 10 + 1) AS quantidade,
    round(CAST(random() * 100 + 1 AS numeric), 2) AS preco_venda,
    round(CAST(random() * 50 + 1 AS numeric), 2) AS custo_produto
  FROM generate_series(1,1000)
)
-- Inserir dados na tabela
INSERT INTO schema1.ft_vendas (id_transacao, id_produto, id_cliente, id_localizacao, data_transacao, quantidade, preco_venda, custo_produto)
SELECT 
  'TRAN-' || id_transacao AS id_transacao,
  id_produto,
  id_cliente,
  id_localizacao,
  data_transacao,
  quantidade,
  round(CAST(preco_venda AS numeric),2),
  round(CAST(custo_produto AS numeric),2)
FROM dados;

```


### ETL Fase 1 (Fonte para a Staging Area - Schema 1 para Schema 2 - Será feito com Airbyte)

#Fazer a sincronização no Airbyte (Sync)

Configurar a fonte > destino > conexão > Fazer o Sync

Obs: Para a criação da conexão no Airbyte é importante utilizar as portas externas, pois o container docker do Airbyte está em uma rede diferente do container Destino.



### ETL Fase 2 (Staging Area para o DW - Schema  2 para Schema 3 - Será feito com linguagem SQL)

#Prepara os dados para a dimensão cliente

#Campos necessários: id_cliente, nome, tipo

#Query:

```
SELECT id_cliente, 
       nome_cliente, 
       nome_tipo
FROM schema2.st_ft_clientes tb1, schema2.st_ft_tipo_cliente tb2
WHERE tb2.id_tipo = tb1.id_tipo;

```


#Prepara os dados para a dimensão produto

#Campos necessários: id_produto, nome_produto, categoria, subcategoria

#Query:

```

SELECT id_produto, 
       nome_produto, 
       nome_categoria, 
       nome_subcategoria
FROM schema2.st_ft_produtos tb1, schema2.st_ft_subcategorias tb2, schema2.st_ft_categorias tb3
WHERE tb3.id_categoria = tb2.id_categoria
AND tb2.id_subcategoria = tb1.id_subcategoria;

```


#Prepara os dados para a dimensão localidade
#Campos necessários: id_localizacao, pais, regiao, estado, cidade 
#Query:

```

SELECT id_localidade, 
       pais, 
       regiao, 
       CASE
        WHEN nome_cidade = 'Natal' THEN 'Rio Grande do Norte'
        WHEN nome_cidade = 'Rio de Janeiro' THEN 'Rio de Janeiro'
        WHEN nome_cidade = 'Belo Horizonte' THEN 'Minas Gerais'
        WHEN nome_cidade = 'Salvador' THEN 'Bahia'
        WHEN nome_cidade = 'Blumenau' THEN 'Santa Catarina'
        WHEN nome_cidade = 'Curitiba' THEN 'Paraná'
        WHEN nome_cidade = 'Fortaleza' THEN 'Ceará'
        WHEN nome_cidade = 'Recife' THEN 'Pernambuco'
        WHEN nome_cidade = 'Porto Alegre' THEN 'Rio Grande do Sul'
        WHEN nome_cidade = 'Manaus' THEN 'Amazonas'
       END estado, 
       nome_cidade
FROM schema2.st_ft_localidades tb1, schema2.st_ft_cidades tb2
WHERE tb2.id_cidade = tb1.id_cidade;

```


#Prepara os dados para a dimensão tempo

#Campos necessários: id_tempo, ano, mes, dia 

#Query:

```
SELECT EXTRACT(YEAR FROM d)::INT, 
       EXTRACT(MONTH FROM d)::INT, 
       EXTRACT(DAY FROM d)::INT, d::DATE
FROM generate_series('2020-01-01'::DATE, '2024-12-31'::DATE, '1 day'::INTERVAL) d;
```


#Modelo físico (Criação de estruturas das tabelas com Surrogate Keys)

```
-- Tabela Dimensão Cliente
CREATE TABLE schema3.dim_cliente (
  sk_cliente SERIAL PRIMARY KEY,
  id_cliente INT NOT NULL,
  nome VARCHAR(50) NOT NULL,
  tipo VARCHAR(50) NOT NULL
);

-- Tabela Dimensão Produto
CREATE TABLE schema3.dim_produto (
  sk_produto SERIAL PRIMARY KEY,
  id_produto INT NOT NULL,
  nome_produto VARCHAR(50) NOT NULL,
  categoria VARCHAR(50) NOT NULL,
  subcategoria VARCHAR(50) NOT NULL
);

-- Tabela Dimensão Localidade
CREATE TABLE schema3.dim_localidade (
  sk_localidade SERIAL PRIMARY KEY,
  id_localidade INT NOT NULL,
  pais VARCHAR(50) NOT NULL,
  regiao VARCHAR(50) NOT NULL,
  estado VARCHAR(50) NOT NULL,
  cidade VARCHAR(50) NOT NULL
);

-- Tabela Dimensão Tempo
CREATE TABLE schema3.dim_tempo (
  sk_tempo SERIAL PRIMARY KEY,
  data_completa date,
  ano INT NOT NULL,
  mes INT NOT NULL,
  dia INT NOT NULL
);

-- Tabela Fato de Vendas
CREATE TABLE schema3.fato_vendas (
  sk_produto INT NOT NULL,
  sk_cliente INT NOT NULL,
  sk_localidade INT NOT NULL,
  sk_tempo INT NOT NULL,
  quantidade INT NOT NULL,
  preco_venda DECIMAL(10,2) NOT NULL,
  custo_produto DECIMAL(10,2) NOT NULL,
  receita_vendas DECIMAL(10,2) NOT NULL,
  PRIMARY KEY (sk_produto, sk_cliente, sk_localidade, sk_tempo),
  FOREIGN KEY (sk_produto) REFERENCES schema3.dim_produto (sk_produto),
  FOREIGN KEY (sk_cliente) REFERENCES schema3.dim_cliente (sk_cliente),
  FOREIGN KEY (sk_localidade) REFERENCES schema3.dim_localidade (sk_localidade),
  FOREIGN KEY (sk_tempo) REFERENCES schema3.dim_tempo (sk_tempo)
);

```

### Carregando os dados no DW

#Carrega a tabela dim_tempo

``` 

INSERT INTO schema3.dim_tempo (ano, mes, dia, data_completa)
SELECT EXTRACT(YEAR FROM d)::INT, 
       EXTRACT(MONTH FROM d)::INT, 
       EXTRACT(DAY FROM d)::INT, d::DATE
FROM generate_series('2020-01-01'::DATE, '2024-12-31'::DATE, '1 day'::INTERVAL) d;
```

#Carrega a tabela dim_cliente no DW a partir da Staging Area

```
INSERT INTO schema3.dim_cliente (id_cliente, nome, tipo)
SELECT id_cliente, 
       nome_cliente, 
       nome_tipo
FROM schema2.st_ft_clientes tb1, schema2.st_ft_tipo_cliente tb2
WHERE tb2.id_tipo = tb1.id_tipo;
```

#Carrega a tabela dim_produto no DW a partir da Staging Area

```
INSERT INTO schema3.dim_produto (id_produto, nome_produto, categoria, subcategoria)
SELECT id_produto, 
       nome_produto, 
       nome_categoria, 
       nome_subcategoria
FROM schema2.st_ft_produtos tb1, schema2.st_ft_subcategorias tb2, schema2.st_ft_categorias tb3
WHERE tb3.id_categoria = tb2.id_categoria
AND tb2.id_subcategoria = tb1.id_subcategoria;
```

#Carrega a tabela dim_localidade no DW a partir da Staging Area

```
INSERT INTO schema3.dim_localidade (id_localidade, pais, regiao, estado, cidade)
SELECT id_localidade, 
	   pais, 
	   regiao, 
	   CASE
	   	WHEN nome_cidade = 'Natal' THEN 'Rio Grande do Norte'
		WHEN nome_cidade = 'Rio de Janeiro' THEN 'Rio de Janeiro'
		WHEN nome_cidade = 'Belo Horizonte' THEN 'Minas Gerais'
		WHEN nome_cidade = 'Salvador' THEN 'Bahia'
		WHEN nome_cidade = 'Blumenau' THEN 'Santa Catarina'
		WHEN nome_cidade = 'Curitiba' THEN 'Paraná'
		WHEN nome_cidade = 'Fortaleza' THEN 'Ceará'
		WHEN nome_cidade = 'Recife' THEN 'Pernambuco'
		WHEN nome_cidade = 'Porto Alegre' THEN 'Rio Grande do Sul'
		WHEN nome_cidade = 'Manaus' THEN 'Amazonas'
	   END estado, 
	   nome_cidade
FROM schema2.st_ft_localidades tb1, schema2.st_ft_cidades tb2
WHERE tb2.id_cidade = tb1.id_cidade;

```

#Carga de dados na tabela Fato:
Como a origem não tem as surrogate Keys, devemos fazer joins pelo Id de cada tabela para o cálculo das métricas

```
INSERT INTO schema3.fato_vendas (sk_produto, 
	                              sk_cliente, 
	                              sk_localidade, 
	                              sk_tempo, 
	                              quantidade, 
	                              preco_venda, 
	                              custo_produto, 
	                              receita_vendas)
SELECT sk_produto,
	     sk_cliente,
	     sk_localidade,
       sk_tempo, 
       quantidade, 
       preco_venda, 
	     custo_produto, 
	     ROUND((CAST(quantidade AS numeric) * CAST(preco_venda AS numeric)), 2) AS receita_vendas
FROM schema2.st_ft_vendas tb1, 
     schema2.st_ft_clientes tb2, 
	   schema2.st_ft_localidades tb3, 
	   schema2.st_ft_produtos tb4,
	   schema3.dim_tempo tb5,
	   schema3.dim_produto tb6,
	   schema3.dim_localidade tb7,
	   schema3.dim_cliente tb8
WHERE tb2.id_cliente = tb1.id_cliente
AND tb3.id_localidade = tb1.id_localizacao
AND tb4.id_produto = tb1.id_produto
AND tb1.data_transacao = tb5.data_completa
AND tb2.id_cliente = tb8.id_cliente
AND tb3.id_localidade = tb7.id_localidade
AND tb4.id_produto = tb6.id_produto;

```

Para não haver duplicidade entre as Surogate Keys, vamos utilizar o GROUP BY na tabela Fato Vendas:
#Carrega a tabela fato_vendas

```
INSERT INTO schema3.fato_vendas (sk_produto, 
	                            sk_cliente, 
	                            sk_localidade, 
	                            sk_tempo, 
	                            quantidade, 
	                            preco_venda, 
	                            custo_produto, 
	                            receita_vendas)
SELECT sk_produto,
	  sk_cliente,
	  sk_localidade,
       sk_tempo, 
       SUM(quantidade) AS quantidade, 
       SUM(preco_venda) AS preco_venda, 
	  SUM(custo_produto) AS custo_produto, 
	  SUM(ROUND((CAST(quantidade AS numeric) * CAST(preco_venda AS numeric)), 2)) AS receita_vendas
FROM schema2.st_ft_vendas tb1, 
     schema2.st_ft_clientes tb2, 
	schema2.st_ft_localidades tb3, 
	schema2.st_ft_produtos tb4,
	schema3.dim_tempo tb5,
	schema3.dim_produto tb6,
	schema3.dim_localidade tb7,
	schema3.dim_cliente tb8
WHERE tb2.id_cliente = tb1.id_cliente
AND tb3.id_localidade = tb1.id_localizacao
AND tb4.id_produto = tb1.id_produto
AND tb1.data_transacao = tb5.data_completa
AND tb2.id_cliente = tb8.id_cliente
AND tb3.id_localidade = tb7.id_localidade
AND tb4.id_produto = tb6.id_produto
GROUP BY sk_produto, sk_cliente, sk_localidade, sk_tempo;
```

### Adicionando uma nova métrica na tabela Fato:
#Limpa a tabela

```
TRUNCATE TABLE schema3.fato_vendas;
```

#Adiciona a nova coluna

```
ALTER TABLE IF EXISTS schema3.fato_vendas
    ADD COLUMN resultado numeric(10, 2) NOT NULL;
```

#Carrega a tabela fato

```
INSERT INTO schema3.fato_vendas (sk_produto, 
                                sk_cliente, 
                                sk_localidade, 
                                sk_tempo, 
                                quantidade, 
                                preco_venda, 
                                custo_produto, 
                                receita_vendas,
                                resultado)
SELECT sk_produto,
       sk_cliente,
       sk_localidade,
       sk_tempo, 
       SUM(quantidade) AS quantidade, 
       SUM(preco_venda) AS preco_venda, 
       SUM(custo_produto) AS custo_produto, 
       SUM(ROUND((CAST(quantidade AS numeric) * CAST(preco_venda AS numeric)), 2)) AS receita_vendas,
       SUM(ROUND((CAST(quantidade AS numeric) * CAST(preco_venda AS numeric)), 2) - custo_produto) AS resultado 
FROM schema2.st_ft_vendas tb1, 
     schema2.st_ft_clientes tb2, 
     schema2.st_ft_localidades tb3, 
     schema2.st_ft_produtos tb4,
     schema3.dim_tempo tb5,
     schema3.dim_produto tb6,
     schema3.dim_localidade tb7,
     schema3.dim_cliente tb8
WHERE tb2.id_cliente = tb1.id_cliente
AND tb3.id_localidade = tb1.id_localizacao
AND tb4.id_produto = tb1.id_produto
AND to_char(tb1.data_transacao, 'YYYY-MM-DD') = to_char(tb5.data_completa, 'YYYY-MM-DD')
AND to_char(tb1.data_transacao, 'HH') = tb5.hora
AND tb2.id_cliente = tb8.id_cliente
AND tb3.id_localidade = tb7.id_localidade
AND tb4.id_produto = tb6.id_produto
GROUP BY sk_produto, sk_cliente, sk_localidade, sk_tempo;
```

### Usando Materialized View para melhorar a performance das consultas:
#View e View Materializada

#Cria uma view (Grava a Query, porém o plano de execução ainda é realizado)

```
CREATE VIEW schema3.vw_relatorio
AS
SELECT estado, 
       categoria, 
       tipo AS tipo_cliente, 
       hora, 
       SUM(resultado)
FROM schema3.dim_produto AS tb1, 
     schema3.dim_cliente AS tb2, 
     schema3.dim_localidade AS tb3, 
     schema3.dim_tempo AS tb4, 
     schema3.fato_vendas AS tb5
WHERE tb5.sk_produto = tb1.sk_produto
AND tb5.sk_cliente = tb2.sk_cliente
AND tb5.sk_localidade = tb3.sk_localidade
AND tb5.sk_tempo = tb4.sk_tempo 
GROUP BY estado, categoria, tipo, hora
ORDER BY estado, categoria, tipo, hora;


SELECT * FROM schema3.vw_relatorio
```


### Cria uma view materializada (Com a MV não há execução de query, portanto a performance melhora)

```
CREATE MATERIALIZED VIEW schema3.mv_relatorio AS
SELECT estado, 
       categoria, 
       tipo AS tipo_cliente, 
       hora, 
       SUM(resultado)
FROM schema3.dim_produto AS tb1, 
     schema3.dim_cliente AS tb2, 
     schema3.dim_localidade AS tb3, 
     schema3.dim_tempo AS tb4, 
     schema3.fato_vendas AS tb5
WHERE tb5.sk_produto = tb1.sk_produto
AND tb5.sk_cliente = tb2.sk_cliente
AND tb5.sk_localidade = tb3.sk_localidade
AND tb5.sk_tempo = tb4.sk_tempo 
GROUP BY estado, categoria, tipo, hora
ORDER BY estado, categoria, tipo, hora;

SELECT * FROM schema3.mv_relatorio;
``` 