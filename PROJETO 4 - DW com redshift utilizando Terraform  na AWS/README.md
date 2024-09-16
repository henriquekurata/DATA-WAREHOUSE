# 🚀 ***Criação de um Data Warehouse na AWS Redshift com Terraform***

## **Descrição do Projeto:**
Este projeto tem como objetivo a criação de um Data Warehouse utilizando o Amazon Redshift e automação da infraestrutura com Terraform. O projeto implementa uma solução escalável e segura para armazenar e consultar dados de vendas, estruturando um ambiente de Big Data em um cluster Redshift.

Utilizando práticas de infraestrutura como código (IaC) com Terraform, o ambiente é provisionado de forma automatizada, incluindo a criação de uma VPC, subnets, security groups, e o cluster Redshift. Além disso, há a configuração de uma política IAM para garantir que o Redshift possa acessar dados no Amazon S3.

Os dados de vendas são carregados no cluster Redshift e armazenados em tabelas dimensionais e factuais, permitindo a análise de grandes volumes de dados de forma rápida e eficiente. A modelagem de dados segue o padrão de Data Warehousing com tabelas fato e dimensões.


## 🛠️ **Tecnologias Utilizadas**:
- **Amazon Redshift**: Data Warehouse escalável para armazenar e consultar grandes volumes de dados.
- **Terraform**: Automação da infraestrutura em nuvem, provisionando e gerenciando recursos da AWS.
- **Docker**: Criação de um ambiente isolado para desenvolvimento e execução dos scripts.
- **AWS CLI**: Ferramenta de linha de comando para interagir com os serviços da AWS.
- **PostgreSQL**: Interface para executar queries SQL no cluster Redshift.

### Funcionalidades Implementadas:
1. **Provisionamento da Infraestrutura**: Criação de uma VPC, subnets e segurança de rede utilizando Terraform.
2. **Criação do Cluster Redshift**: Configuração do cluster de Redshift para armazenar os dados de vendas.
3. **Carregamento de Dados**: Modelagem e inserção de dados nas tabelas `dim_cliente`, `dim_produto`, `dim_localidade`, e `fato_vendas` via script SQL.
4. **Gerenciamento de Acesso IAM**: Criação de políticas IAM para permitir que o Redshift acesse buckets S3 para futura integração de dados.

### Objetivo
O projeto foi desenvolvido com o objetivo de criar uma solução eficiente de Data Warehouse, que possa ser replicada e escalada facilmente, garantindo flexibilidade e segurança na gestão e análise de grandes volumes de dados.


## 📋 **Descrição do Processo**
* Acessar conta AWS e criar as credenciais de segurança;
* Criar container docker para máquina cliente;
* Instalar AWS CLI e Terraform no container;
* Criar o arquivo Terraform no container;
* Executar o Terraform, aplicar infraestrutura e destruir (`init`, `apply`, `destroy`).



## **Comandos:**

A conexão entre o Terraform e o Redshift será feita pelo AWS CLI. Para isso funcionar, será necessário criar as credenciais de segurança para acesso remoto (criar diretamente no console da AWS).

---

### Preparação da Máquina Cliente 

#### 1. Criar um container Docker (na sua máquina local)

docker run -dti --name dsa_projeto2 --rm ubuntu


#### 2. Instalar utilitários 

Execute os comandos abaixo no container criado:

apt-get update

apt-get upgrade

apt-get install curl nano wget unzip


#### 3. Criar pasta de Downloads

mkdir Downloads

cd Downloads


#### 4. Download do AWS CLI

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"


#### 5. Unzip e install

unzip awscliv2.zip

./aws/install


#### 6. Verificar a versão

aws --version


#### 7. Configurar AWS CLI

aws configure

- Access key ID: coloque a sua chave
- Secret access key: coloque a sua chave
- Default region name: `us-east-2`
- Default output format: deixe em branco e pressione enter


#### 8. Testar a configuração

aws s3 ls


#### 9.Instalar o Terraform

apt-get update && apt-get install -y gnupg software-properties-common

wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | tee /usr/share/keyrings/hashicorp-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/hashicorp.list

apt update

apt-get install terraform


#### 10. Verificar a versão do Terraform

terraform -version

----

### Criar pasta no container Docker com o nome do projeto que está o arquivo `main.tf`:

Nesse caso: Container dsa_projeto2 > mkdir terraform-aws-hcl na pasta raiz do container (~)

---

### Arquivo main.tf

Criar o arquivo inserindo os dados abaixo com o editor de texto:

```
provider "aws" {
  region = "us-east-2"
}

resource "aws_security_group" "allow_http_ssh" {
  name        = "allow_http_ssh"
  description = "Allow HTTP and SSH traffic"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "web_server" {
  ami           = "ami-0c55b159cbfafe1f0" # Amazon Linux 2 AMI ID
  instance_type = "dc2.large"

  vpc_security_group_ids = [aws_security_group.allow_http_ssh.id]

  tags = {
    Name = "Web Server"
  }
}

```
---

### Executar o terraform, aplicar infraestrutura e destruir 

terraform init (Inicialização do Teraform)

terraform apply (Validação para já executar o script)

terraform destroy (Limpa tudo - Grupo de segurança e instância EC2)

---

### Arquivo main.tf para o Cluster Redshift

#### Preparando cluster Redshift para o DW usando infraestrutura como códico com Terraform

```
#Configura o Provedor AWS

provider "aws" {
  region = "us-east-2"
}


#Configura a Redshift VPC (Organização lógica com range de endereços IP)
resource "aws_vpc" "redshift_vpc" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "Redshift VPC"
  }
}

# Configura a Redshift Subnet (Divisão da VPC para conctar os serviços a infraestrutura, exemplo: DW em uma subnet e aplicação ETL em outra subnet, para dar maior segurança)
resource "aws_subnet" "redshift_subnet" {
  cidr_block = "10.0.1.0/24"
  vpc_id     = aws_vpc.redshift_vpc.id

  tags = {
    Name = "Redshift Subnet"
  }
}

# Configura um Gateway da Internet e Anexa a VPC (Como endereços 10/172/192 são apenas internos, se faz necessário a criação da intenet gateway abrindo assim para a internet externa)
resource "aws_internet_gateway" "redshift_igw" {
  vpc_id = aws_vpc.redshift_vpc.id

  tags = {
    Name = "Redshift Internet Gateway"
  }
}

# Configura Uma Tabela de Roteamento (Configuração para a rota de saída com a internet)
resource "aws_route_table" "redshift_route_table" {
  vpc_id = aws_vpc.redshift_vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.redshift_igw.id
  }

  tags = {
    Name = "Redshift Route Table"
  }
}

# Associa a Tabela de Roteamento à Subnet (Configura a saída para as Subnets)
resource "aws_route_table_association" "redshift_route_table_association" {
  subnet_id      = aws_subnet.redshift_subnet.id
  route_table_id = aws_route_table.redshift_route_table.id
}

# Configura Um Grupo de Segurança de Acesso ao Data Warehouse com Redshift
resource "aws_security_group" "redshift_sg" {
  name        = "redshift_sg"
  description = "Allow Redshift traffic"
  vpc_id      = aws_vpc.redshift_vpc.id

  ingress {
    from_port   = 5439
    to_port     = 5439
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "Redshift Security Group"
  }
}

# Configura Um Grupo de Subnets Redshift
resource "aws_redshift_subnet_group" "redshift_subnet_group" {
  name       = "redshift-subnet-group"
  subnet_ids = [aws_subnet.redshift_subnet.id]

  tags = {
    Name = "Redshift Subnet Group"
  }
}

# Configura Um Cluster Redshift (Será feito com apenas uma máquina)
resource "aws_redshift_cluster" "redshift_cluster" {
  cluster_identifier = "redshift-cluster"
  database_name      = "dsadb"
  master_username    = "adminuser"
  master_password    = "dsaSecurePassw0rd!"
  node_type          = "dc2.large"
  number_of_nodes    = 1

  vpc_security_group_ids = [aws_security_group.redshift_sg.id]
  cluster_subnet_group_name = aws_redshift_subnet_group.redshift_subnet_group.name

  skip_final_snapshot = true
}

```

---

Acessando o Redshift: Redshift-cluster > Query data > Conectar ao banco de dados com nome e senha do arquivo main.tf


---








# ***Deploy do DW na AWS com Terraform - Parte 2***



## **Comandos:**

### 1- Acesse sua conta AWS e crie um bucket na região de Ohio.

### 2- Dentro do bucket crie uma pasta chamada dados.

### 3- Faça o upload dos 5 arquivos CSV para essa pasta criada.

### 4-Criar o container Docker local.

Preparação da Máquina Cliente Para o Projeto 2

#### Cria um container Docker (na sua máquina local) com PostgreSQL e bibliotecas de conexão cliente:

docker run --name cliente_dsa -p 5438:5432 -e POSTGRES_USER=dsadmin -e POSTGRES_PASSWORD=dsadmin123 -e POSTGRES_DB=dsdb -d postgres



### Instala utilitários

apt-get update

apt-get upgrade

apt-get install curl nano wget unzip vim sudo



### Cria pasta de Downloads


mkdir Downloads

cd Downloads

### Download do AWS CLI


curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"



### Unzip e install

unzip awscliv2.zip

./aws/install



### Versão

aws --version



### Configura AWS CLI

aws configure

Access key ID: coloque a sua chave

Secret access key: coloque a sua chave

Default region name: us-east-2

Default output format: deixe em branco e pressione enter



### Teste

aws s3 ls



### Instala o Terraform
apt-get update && apt-get install -y gnupg software-properties-common

wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | tee /usr/share/keyrings/hashicorp-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/hashicorp.list

apt update

apt-get install terraform



### Versão do Terraform
terraform -version



### 5- Acesse o terminal do container e crie as pastas abaixo:

cd ~

mkdir projeto2

cd projeto2

mkdir etapa1

cd etapa1



### 6- Na pasta etapa1 crie os arquivos abaixo:

touch provider.tf

touch redshift.tf

touch redshift_role.tf



### 7- Edite cada um dos arquivos:

nano provider.tf

nano redshift.tf

nano redshift_role.tf


### provider.tf:

```
provider "aws" {
  region = "us-east-2"
}
```


### redshift.tf:


#Configura a Redshift VPC
```
resource "aws_vpc" "redshift_vpc" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "Redshift VPC"
  }
}


```


#Configura a Redshift Subnet
```
resource "aws_subnet" "redshift_subnet" {
  cidr_block = "10.0.1.0/24"
  vpc_id     = aws_vpc.redshift_vpc.id

  tags = {
    Name = "Redshift Subnet"
  }
}
```

#Configura um Gateway da Internet e Anexa a VPC
```
resource "aws_internet_gateway" "redshift_igw" {
  vpc_id = aws_vpc.redshift_vpc.id

  tags = {
    Name = "Redshift Internet Gateway"
  }
}
```


#Configura Uma Tabela de Roteamento
```
resource "aws_route_table" "redshift_route_table" {
  vpc_id = aws_vpc.redshift_vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.redshift_igw.id
  }

  tags = {
    Name = "Redshift Route Table"
  }
}
```


#Associa a Tabela de Roteamento à Subnet
```
resource "aws_route_table_association" "redshift_route_table_association" {
  subnet_id      = aws_subnet.redshift_subnet.id
  route_table_id = aws_route_table.redshift_route_table.id
}
```


#Configura Um Grupo de Segurança de Acesso ao Data Warehouse com Redshift
```
resource "aws_security_group" "redshift_sg" {
  name        = "redshift_sg"
  description = "Allow Redshift traffic"
  vpc_id      = aws_vpc.redshift_vpc.id

  ingress {
    from_port   = 5439
    to_port     = 5439
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "Redshift Security Group"
  }
}
```


#Configura Um Grupo de Subnets Redshift
```
resource "aws_redshift_subnet_group" "redshift_subnet_group" {
  name       = "redshift-subnet-group"
  subnet_ids = [aws_subnet.redshift_subnet.id]

  tags = {
    Name = "Redshift Subnet Group"
  }
}

```

#Configura Um Cluster Redshift 
```
resource "aws_redshift_cluster" "redshift_cluster" {
  cluster_identifier = "redshift-cluster"
  database_name      = "dsadb"
  master_username    = "adminuser"
  master_password    = "dsaS9curePassw2rd"
  node_type          = "dc2.large"
  number_of_nodes    = 1

  vpc_security_group_ids = [aws_security_group.redshift_sg.id]
  cluster_subnet_group_name = aws_redshift_subnet_group.redshift_subnet_group.name
  iam_roles = [aws_iam_role.redshift_role.arn]

  skip_final_snapshot = true
}

```

### Redshift_role:

redshift_role: (IAM é o privilégio de acesso entre serviços distintos da AWS)

```
resource "aws_iam_role" "redshift_role" {
  name = "RedshiftS3AccessRole"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "redshift.amazonaws.com"
        }
      }
    ]
  })
}

resource "aws_iam_role_policy_attachment" "redshift_s3_read" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess"
  role       = aws_iam_role.redshift_role.name
}

```

### 8- Pelo terminal, na pasta etapa1, execute os comandos abaixo:

terraform init

terraform validate

terraform plan

terraform apply

### 9- Acesse o painel do Redshift na AWS e confirme que o cluster do Redshift foi criado para o DW.



### 10- Acesse o painel do IAM na AWS e verifique se a role RedshiftS3AccessRole foi criada. Copie o endereço ARN da role e coloque no arquivo load_data.sql.



### 11- Crie a pasta etapa2 no container:

cd ~

cd projeto2

mkdir etapa2

cd etapa2



### 12- Dentro da pasta etapa2 coloque o arquivo load_data.sql:

touch load_data.sql

nano load_data.sql

Arquivo load_data.sql:
```SQL
CREATE SCHEMA IF NOT EXISTS dsaschema;

CREATE TABLE IF NOT EXISTS dsaschema.dim_cliente 
(
    sk_cliente integer NOT NULL,
    id_cliente integer NOT NULL,
    nome character varying(50) NOT NULL,
    tipo character varying(50),
    CONSTRAINT dim_cliente_pkey PRIMARY KEY (sk_cliente)
);

CREATE TABLE IF NOT EXISTS dsaschema.dim_localidade
(
    sk_localidade integer NOT NULL,
    id_localidade integer NOT NULL,
    pais character varying(50) NOT NULL,
    regiao character varying(50) NOT NULL,
    estado character varying(50) NOT NULL,
    cidade character varying(50) NOT NULL,
    CONSTRAINT dim_localidade_pkey PRIMARY KEY (sk_localidade)
);

CREATE TABLE IF NOT EXISTS dsaschema.dim_produto
(
    sk_produto integer NOT NULL,
    id_produto integer NOT NULL,
    nome_produto character varying(50) NOT NULL,
    categoria character varying(50) NOT NULL,
    subcategoria character varying(50) NOT NULL,
    CONSTRAINT dim_produto_pkey PRIMARY KEY (sk_produto)
);

CREATE TABLE IF NOT EXISTS dsaschema.dim_tempo
(
    sk_tempo integer NOT NULL,
    data_completa date,
    ano integer NOT NULL,
    mes integer NOT NULL,
    dia integer NOT NULL,
    CONSTRAINT dim_tempo_pkey PRIMARY KEY (sk_tempo)
);

CREATE TABLE IF NOT EXISTS dsaschema.fato_vendas
(
    sk_produto integer NOT NULL,
    sk_cliente integer NOT NULL,
    sk_localidade integer NOT NULL,
    sk_tempo integer NOT NULL,
    quantidade integer NOT NULL,
    preco_venda numeric(10,2) NOT NULL,
    custo_produto numeric(10,2) NOT NULL,
    receita_vendas numeric(10,2) NOT NULL,
    CONSTRAINT fato_vendas_pkey PRIMARY KEY (sk_produto, sk_cliente, sk_localidade, sk_tempo),
    CONSTRAINT fato_vendas_sk_cliente_fkey FOREIGN KEY (sk_cliente) REFERENCES dsaschema.dim_cliente (sk_cliente),
    CONSTRAINT fato_vendas_sk_localidade_fkey FOREIGN KEY (sk_localidade) REFERENCES dsaschema.dim_localidade (sk_localidade),
    CONSTRAINT fato_vendas_sk_produto_fkey FOREIGN KEY (sk_produto) REFERENCES dsaschema.dim_produto (sk_produto),
    CONSTRAINT fato_vendas_sk_tempo_fkey FOREIGN KEY (sk_tempo) REFERENCES dsaschema.dim_tempo (sk_tempo)
);

COPY dsaschema.dim_cliente
FROM 's3://dsa-projeto2/dados/dim_cliente.csv'
IAM_ROLE 'arn:aws:iam::890582101704:role/RedshiftS3AccessRole'
CSV;

COPY dsaschema.dim_localidade
FROM 's3://dsa-projeto2/dados/dim_localidade.csv'
IAM_ROLE 'arn:aws:iam::890582101704:role/RedshiftS3AccessRole'
CSV;

COPY dsaschema.dim_produto
FROM 's3://dsa-projeto2/dados/dim_produto.csv'
IAM_ROLE 'arn:aws:iam::890582101704:role/RedshiftS3AccessRole'
CSV;

COPY dsaschema.dim_tempo
FROM 's3://dsa-projeto2/dados/dim_tempo.csv'
IAM_ROLE 'arn:aws:iam::890582101704:role/RedshiftS3AccessRole'
CSV;

COPY dsaschema.fato_vendas
FROM 's3://dsa-projeto2/dados/fato_vendas.csv'
IAM_ROLE 'arn:aws:iam::890582101704:role/RedshiftS3AccessRole'
CSV;

```

### 13- Copie o endpoint do seu cluster Redshift e ajuste o comando abaixo e então execute no terminal do container dentro da pasta etapa2. 

Digite a senha (dsaS9curePassw2rd) quando solicitado.

psql -h redshift-cluster.cbwssuxzxipm.us-east-2.redshift.amazonaws.com -U adminuser -d dsadb -p 5439 -f load_data.sql

O comando acima irá criar e inserir o schema e os dados no banco Redshift



### 14- Edite o arquivo redshift.tf e acrescente a linha abaixo para associar a role do S3 ao cluster Redshift.

iam_roles = [aws_iam_role.redshift_role.arn]



### 15- Execute novamente o terraform apply para modificar o cluster em tempo real. Repita o passo 13. Seu DW está pronto para uso.



### 16- Acesse o editor de consultas do Redshift e confira se os dados foram carregados.



### 17- Quando terminar o trabalho, destrua a infra com o comando: terraform destroy.


---
## Contato

Se tiver dúvidas ou sugestões sobre o projeto, entre em contato comigo:

- 💼 [LinkedIn](https://www.linkedin.com/in/henrique-k-32967a2b5/)
- 🐱 [GitHub](https://github.com/henriquekurata?tab=overview&from=2024-09-01&to=2024-09-01)