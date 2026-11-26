# FrameFlow — Infraestrutura local

## 1. Objetivo

Disponibilizar um ambiente de infraestrutura local padronizado para o desenvolvimento e a integração dos serviços do FrameFlow, utilizando Docker Compose.

## 2. Pré-requisitos

- Docker Engine ou Docker Desktop.
- Docker Compose v2.
- AWS CLI v2 (opcional, para validação do armazenamento S3).

## 3. Componentes da infraestrutura

| Componente | Tecnologia | Responsabilidade |
|---|---|---|
| Banco de dados | PostgreSQL 17 | Persistência dos dados de autenticação e processamento de vídeos |
| Mensageria | RabbitMQ 4 | Comunicação assíncrona entre os serviços |
| Armazenamento | SeaweedFS 4.55 | Armazenamento de vídeos e arquivos processados, utilizando uma API compatível com Amazon S3 |

## 4. Inicialização do ambiente

Na raiz do repositório, execute os comandos abaixo no PowerShell.

Crie o arquivo de configuração local, caso ainda não exista:

```powershell
Copy-Item .env.example .env
```

Edite o arquivo `.env` com as credenciais desejadas antes de iniciar o ambiente.

Valide a configuração:

```powershell
docker compose config --quiet
```

Inicie os serviços:

```powershell
docker compose up -d --wait
```

Verifique a situação dos containers:

```powershell
docker compose ps
```

Todos os serviços devem apresentar o estado `healthy`.

## 5. Acessos locais

| Serviço | Endereço |
|---|---|
| PostgreSQL | localhost:5432 |
| RabbitMQ (AMQP) | localhost:5672 |
| RabbitMQ Management | http://localhost:15672 |
| SeaweedFS (S3 API) | http://localhost:8333 |

As portas dos serviços são publicadas exclusivamente na interface de loopback, restringindo o acesso à máquina local.

Para comunicação entre containers conectados à mesma rede Docker, utilize os nomes dos serviços:

- `postgres:5432`
- `rabbitmq:5672`
- `seaweedfs:8333`

As credenciais são definidas no arquivo `.env`, que não deve ser versionado.

## 6. PostgreSQL

O ambiente disponibiliza dois bancos de dados:

- `frameflow_auth`: dados relacionados à autenticação e aos usuários.
- `frameflow_video`: dados relacionados aos vídeos e ao acompanhamento do processamento.

Os bancos são inicializados automaticamente na primeira criação do volume PostgreSQL.

Para validar os bancos:

```powershell
docker compose exec postgres sh
```

Dentro do container:

```bash
psql -U "$POSTGRES_USER" -d postgres
```

Execute:

```sql
SELECT datname FROM pg_database;
```

Para sair:

```text
\q
exit
```

## 7. RabbitMQ

O RabbitMQ disponibiliza o virtual host `frameflow` para a comunicação assíncrona entre os serviços.

O painel administrativo pode ser acessado em:

http://localhost:15672

Para validar o virtual host e a disponibilidade do serviço:

```powershell
docker compose exec rabbitmq rabbitmqctl list_vhosts
docker compose exec rabbitmq rabbitmq-diagnostics ping
```

## 8. Armazenamento de objetos — SeaweedFS

O SeaweedFS disponibiliza uma API compatível com Amazon S3 para armazenamento de arquivos no ambiente local.

Os seguintes buckets são criados automaticamente:

| Bucket | Finalidade |
|---|---|
| `videos-original` | Armazenamento dos vídeos enviados pelos usuários |
| `videos-processed` | Armazenamento dos arquivos resultantes do processamento |

### 8.1. Configuração do cliente S3

| Propriedade | Valor local |
|---|---|
| Endpoint | http://localhost:8333 |
| Endpoint interno Docker | http://seaweedfs:8333 |
| Região | us-east-1 |
| Path Style | true |
| Access Key | Definida no `.env` |
| Secret Key | Definida no `.env` |

### 8.2. Validação dos buckets

Com o AWS CLI instalado, configure as credenciais locais em uma sessão dedicada do PowerShell, substituindo os valores pelas credenciais do `.env`:

```powershell
$env:AWS_ACCESS_KEY_ID="SUA_ACCESS_KEY"
$env:AWS_SECRET_ACCESS_KEY="SUA_SECRET_KEY"
$env:AWS_DEFAULT_REGION="us-east-1"
```

Liste os buckets:

```powershell
aws --endpoint-url http://localhost:8333 s3 ls
```

Os buckets `videos-original` e `videos-processed` devem ser apresentados.

### 8.3. Validação de upload e download

```powershell
"FrameFlow - Teste S3" | Set-Content teste.txt

aws --endpoint-url http://localhost:8333 s3 cp teste.txt s3://videos-original/teste.txt

aws --endpoint-url http://localhost:8333 s3 cp s3://videos-original/teste.txt teste-download.txt

(Get-FileHash teste.txt).Hash -eq (Get-FileHash teste-download.txt).Hash
```

O resultado esperado é `True`, confirmando a integridade do arquivo.

Remova os arquivos de teste:

```powershell
aws --endpoint-url http://localhost:8333 s3 rm s3://videos-original/teste.txt

Remove-Item teste.txt, teste-download.txt
```

Feche a sessão dedicada do PowerShell após concluir os testes, evitando utilizar as credenciais locais em operações destinadas à AWS.

## 9. Health checks

Todos os componentes possuem verificações de disponibilidade configuradas no Docker Compose.

| Serviço | Verificação |
|---|---|
| PostgreSQL | Disponibilidade do banco de dados |
| RabbitMQ | Disponibilidade do broker |
| SeaweedFS | Disponibilidade do endpoint HTTP S3 |

O health check do SeaweedFS considera os códigos HTTP 200 e 403 como respostas válidas de disponibilidade. A resposta 403 é esperada em requisições não autenticadas.

Para acompanhar os serviços:

```powershell
docker compose ps
```

Para consultar os logs:

```powershell
docker compose logs --tail 50
```

## 10. Persistência dos dados

Os serviços utilizam volumes Docker para preservar seus dados entre reinicializações.

- `postgres-data`
- `rabbitmq-data`
- `seaweedfs-data`

Para interromper o ambiente preservando os dados:

```powershell
docker compose down
```

Para remover o ambiente e todos os dados persistidos:

```powershell
docker compose down -v
```

**Atenção:** a remoção dos volumes é irreversível sem um backup.

## 11. Segurança e considerações

- O arquivo `.env` não deve ser versionado.
- O arquivo `.env.example` deve conter apenas valores de exemplo.
- Os serviços são expostos exclusivamente na interface de loopback.
- O armazenamento S3 exige autenticação para acesso aos buckets.
- A versão do SeaweedFS está fixada para evitar atualizações inesperadas.
- Os bancos compartilham uma conta administrativa no ambiente local.
- Alterações nas credenciais do PostgreSQL não modificam automaticamente os usuários já persistidos.
- Esta configuração é destinada exclusivamente ao desenvolvimento local e não deve ser utilizada diretamente em produção.

O ambiente de produção possuirá configurações específicas de segurança, disponibilidade e gerenciamento de credenciais.

## 12. Validação do ambiente

Durante a configuração da infraestrutura local, foram realizadas as seguintes validações:

- Inicialização dos containers.
- Criação dos bancos de dados PostgreSQL.
- Disponibilidade do RabbitMQ e do virtual host.
- Autenticação na API compatível com S3.
- Criação automática dos buckets.
- Upload e download de arquivos.
- Verificação da integridade dos arquivos.
- Disponibilidade dos health checks.