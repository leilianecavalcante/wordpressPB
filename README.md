# Projeto AWS-Docker-WordPress

Este projeto tem como objetivo realizar o deploy de uma aplicação WordPress em container Docker, de forma segura e com alta disponibilidade na AWS, utilizando boas práticas de infraestrutura.

---

## Etapa 1: Criação da VPC

**Objetivo:** Criar uma VPC com duas subnets públicas e duas privadas, distribuídas entre duas zonas de disponibilidade.

**Passo a passo:**

1. Acesse o serviço **VPC** no console da AWS.  
2. Clique em **Create VPC**.  
3. Preencha os campos conforme abaixo:

| Campo               | Valor                    |
|---------------------|--------------------------|
| Name tag            | aws-docker               |
| Resources to create  | VPC and more             |
| Availability Zones  | 2                        |
| Public Subnets      | 2                        |
| Private Subnets     | 2                        |
| NAT gateways        | 1 por AZ                 |
| VPC endpoints       | Nenhum                   |

4. Clique em **Create VPC** para confirmar.

---

## Etapa 2: Criando um Launch Template para instância EC2

**Objetivo:** Definir um modelo padronizado de instância EC2 para uso no Auto Scaling Group com Load Balancer.

**Passo a passo:**

1. Acesse o serviço **EC2** no console da AWS.  
2. No menu lateral, clique em **Launch Templates** > **Create launch template**.  
3. Configure os campos principais:

| Campo             | Valor                            |
|-------------------|---------------------------------|
| Launch template name | wordpress-template            |
| AMI               | AL2023                          |
| Instance type     | t3.micro (ou equivalente)        |
| Key pair          | Selecione um existente           |
| Network settings  | Nenhum (configurado via Auto Scaling) |
| Security Group    | Permita acesso HTTP               |
| User data         | Script de preparação do WordPress com Docker |

4. Clique em **Create launch template**.

---

## Etapa 3: Criação da Instância RDS

**Objetivo:** Criar banco de dados relacional gerenciado (Amazon RDS) para a aplicação.

**Passo a passo:**

1. Acesse o serviço **Amazon RDS**.  
2. Clique em **Create database** > **Standard Create**.  
3. Configure:

| Campo               | Valor                          |
|---------------------|--------------------------------|
| Engine              | MySQL                          |
| Version             | Versão estável mais recente    |
| DB instance identifier | wordpress-db                 |
| Master username     | admin                          |
| Master password     | auto-generate                  |
| DB instance size    | db.t3.micro                    |
| Storage             | 20 GB (General Purpose SSD)    |

4. Em **Connectivity**:

| Campo               | Valor                          |
|---------------------|--------------------------------|
| VPC                 | Selecione a VPC criada         |
| Subnet group        | Subnets privadas               |
| Public access       | No                            |
| VPC security group  | Permita acesso da aplicação    |

5. Em **Additional configuration**, defina o banco inicial como `wordpress`.  
6. Clique em **Create database**.

---

## Etapa 4: Auto Scaling Group com Load Balancer

**Objetivo:** Garantir alta disponibilidade e escalabilidade automática da aplicação.

**Passo a passo:**

1. Acesse **EC2** > **Auto Scaling Groups** > **Create Auto Scaling group**.  
2. Configure:

| Campo               | Valor                  |
|---------------------|------------------------|
| Auto Scaling group name | wordpress-asg        |
| Launch template     | wordpress-template       |

3. Defina rede e subnets: selecione a VPC e as duas subnets públicas.  
4. Em **Load Balancing**:

- Selecione **Attach to a new load balancer**  
- Tipo: Application Load Balancer (ALB)  
- Nome: wordpress-alb  
- Scheme: Internet-facing  
- Listeners: HTTP (porta 80)  
- Target group: novo, tipo instance, HTTP na porta 80  

5. Health checks: padrão HTTP na raiz (`/`).  
6. Configure o grupo de escalabilidade:

| Campo            | Valor  |
|------------------|---------|
| Desired capacity | 2       |
| Minimum capacity | 1       |
| M
### Passo a Passo para Criar Mount Targets do EFS em Subnets Privadas

1. **Abra o Console AWS e vá para o serviço EFS**.
2. Clique no seu sistema de arquivos EFS criado.
3. Vá na aba ou seção chamada **"Network"** ou **"Mount targets"**.
4. Clique em **"Manage"** para editar os mount targets.
5. Agora, você verá uma lista das Zonas de Disponibilidade (AZs) da sua região.
6. Para cada AZ, crie um mount target:
   - **Selecione a subnet privada correspondente** daquela AZ.  
     *Exemplo:* Se a AZ for `us-east-1a`, escolha a subnet privada que está em `us-east-1a`.
7. **Selecione o Security Group** criado para o EFS (ex: `wordpress-sg-efs`).
8. Repita o processo para todas as AZs onde sua VPC possui subnets privadas.
9. Clique em **"Save"** ou **"Create"** para confirmar os mount targets.

---

### Como identificar suas subnets privadas:

1. No Console AWS, acesse o serviço **VPC**.
2. Clique em **Subnets** no menu lateral.
3. Na lista, observe as colunas **Name** e **Availability Zone (AZ)**.
4. Veja a coluna **Auto-assign Public IP** — se estiver **No**, provavelmente é uma subnet privada.
5. Para confirmar, veja a rota da subnet:
   - Clique na subnet.
   - Acesse a aba **Route Table**.
   - Se a rota padrão (`0.0.0.0/0`) apontar para um **NAT Gateway** ou **NAT Instance**, essa é uma subnet privada.

## Passo a Passo: Configurando o Auto Scaling Group para WordPress

### 1. Criar o Auto Scaling Group
- No console da AWS, acesse **EC2** > **Auto Scaling Groups**.
- Clique em **Create Auto Scaling group**.
- Defina o nome do grupo, ex: `wordpress-scaling`.
- Selecione o **Launch Template** criado, ex: `wordpressAPI`.
- Clique em **Next**.

### 2. Configurar a Rede
- Selecione a **VPC** (ex: `wordpress-vpc`).
- Escolha as **subnets privadas** em diferentes zonas de disponibilidade para alta disponibilidade.
- Clique em **Next**.

### 3. Anexar o Load Balancer
- Marque a opção **attach to an existing load balancer**.
- Selecione o Load Balancer criado (ex: `wordpress`).
- Deixe as opções padrão.
- Clique em **Next**.

### 4. Definir Quantidade e Política
- Desired capacity: 2
- Minimum capacity: 2
- Maximum capacity: 4
- Mantenha políticas de escala automática padrão (ou nenhuma).
- Clique em **Next**.

### 5. Revisar e Criar
- Revise as configurações.
- Clique em **Create Auto Scaling group**.

### 6. Verificar e Monitorar
- Verifique se 2 instâncias foram lança



