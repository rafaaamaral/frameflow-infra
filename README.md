# FrameFlow — Infrastructure

Repositório responsável pelo provisionamento, configuração e gerenciamento da infraestrutura do FrameFlow, projeto desenvolvido como parte do Hackathon da pós-graduação em Software Architecture da FIAP.

## Sobre o projeto

O FrameFlow é uma solução distribuída para processamento assíncrono de vídeos, permitindo o envio de arquivos, a extração de frames e a disponibilização dos resultados para download.

A infraestrutura fornece os recursos necessários para a comunicação, persistência, armazenamento e execução dos serviços da aplicação.

## Tecnologias

| Tecnologia | Finalidade |
|---|---|
| Docker | Containerização |
| Docker Compose | Orquestração do ambiente local |
| PostgreSQL | Persistência de dados |
| RabbitMQ | Mensageria e processamento assíncrono |
| SeaweedFS | Armazenamento de objetos no ambiente local |
| Amazon S3 | Armazenamento de objetos em nuvem |
| Terraform | Infraestrutura como código |
| Kubernetes | Orquestração de containers |
| AWS | Infraestrutura em nuvem |
| GitHub Actions | Automação de CI/CD |

## Arquitetura

A solução é organizada em serviços independentes, com responsabilidades específicas e comunicação assíncrona para o processamento de vídeos.

A infraestrutura contempla:

- Persistência de dados de autenticação e processamento.
- Comunicação assíncrona por mensageria.
- Armazenamento de vídeos originais e arquivos processados.
- Isolamento dos componentes.
- Escalabilidade dos serviços.
- Automação do provisionamento e deployment.

## Repositórios

| Repositório | Responsabilidade |
|---|---|
| [frameflow-auth](https://github.com/rafaaamaral/frameflow-auth) | Cadastro e autenticação de usuários |
| [frameflow-video-api](https://github.com/rafaaamaral/frameflow-video-api) | Gerenciamento de vídeos e acompanhamento do processamento |
| [frameflow-video-worker](https://github.com/rafaaamaral/frameflow-video-worker) | Processamento assíncrono e extração de frames |
| [frameflow-infra](https://github.com/rafaaamaral/frameflow-infra) | Infraestrutura e provisionamento de recursos |

## Ambiente local

O ambiente local disponibiliza os seguintes componentes:

- PostgreSQL 17.
- RabbitMQ 4.
- SeaweedFS 4.55.

Todos os componentes possuem persistência de dados e verificações de disponibilidade.

Para configurar e executar o ambiente, consulte:

[Documentação do ambiente local](./README-local.md)

## Infraestrutura em nuvem

A infraestrutura em nuvem será provisionada utilizando Terraform, com execução dos serviços em Kubernetes e armazenamento de objetos no Amazon S3.

## Segurança

- Gerenciamento de credenciais por variáveis de ambiente.
- Separação entre configurações locais e de produção.
- Restrição de acesso aos serviços de infraestrutura.
- Versionamento de