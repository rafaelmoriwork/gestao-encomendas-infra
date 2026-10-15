# Gestão de Encomendas – Infraestrutura

Repositório responsável pela **configuração da infraestrutura** de um projeto de estudos de **TMS (Transportation Management System)** construído com **arquitetura de microserviços**.

Aqui ficam o `docker-compose.yml`, os Dockerfiles de cada serviço/banco e as variáveis de ambiente necessárias para subir o ambiente. O código das aplicações **não** fica neste repositório: cada microserviço possui o seu próprio repositório.

## Microserviços

Cada microserviço é **independente** e pode ter **um ou mais bancos de dados** próprios.

| Serviço | Repositório | Função |
|---|---|---|
| `encomenda-service` | [rafaelmoriwork/encomenda-service](https://github.com/rafaelmoriwork/encomenda-service) | Responsável pelo **gerenciamento das encomendas** do TMS. |

## Estrutura do repositório

```
.
├── docker-compose.yml                       # Orquestra os containers
├── .env.example                             # Modelo das variáveis de ambiente
├── .env                                     # Valores reais (não versionado)
└── encomenda-service/
    ├── app/Dockerfile                       # Imagem da API (build multi-stage, Node 24 Alpine)
    └── databases/main-database/Dockerfile   # Imagem do banco (MySQL 8.4)
```

## Configuração

Crie o arquivo `.env` a partir do modelo:

```bash
cp .env.example .env
```

Todas as variáveis abaixo são **obrigatórias** (o compose falha com “Variavel obrigatoria” se alguma estiver vazia), exceto `ENCOMENDA_NODE_ENV`, que assume `production` por padrão.

### `encomenda-service`: banco de dados (MySQL)

| Variável | Descrição | Exemplo |
|---|---|---|
| `ENCOMENDA_DB_NAME` | Nome do banco criado na inicialização | `encomendas` |
| `ENCOMENDA_DB_USER` | Usuário da aplicação | `encomenda_user` |
| `ENCOMENDA_DB_PASSWORD` | Senha do usuário da aplicação | *(defina uma senha forte)* |
| `ENCOMENDA_DB_ROOT_PASSWORD` | Senha do usuário `root` do MySQL | *(defina uma senha forte)* |
| `ENCOMENDA_DB_PORT` | Porta **no host** para acessar o banco (mapeada para a 3306 do container) | `3307` |

### `encomenda-service`: aplicação

| Variável | Descrição | Exemplo |
|---|---|---|
| `ENCOMENDA_NODE_ENV` | Ambiente do Node (`NODE_ENV`) | `production` |
| `ENCOMENDA_API_PORT` | Porta **no host** para acessar a API (mapeada para a 3000 do container) | `3000` |
| `ENCOMENDA_API_SRC` | Caminho do código-fonte do `encomenda-service` (projeto da API), usado como *build context* | `/caminho/para/encomenda-service` |

#### Sobre o `ENCOMENDA_API_SRC`

O código do `encomenda-service` pode estar **em qualquer pasta** da sua máquina: basta clonar o [repositório](https://github.com/rafaelmoriwork/encomenda-service) onde preferir e apontar `ENCOMENDA_API_SRC` para ela (caminho absoluto, ou relativo à pasta deste repositório).

### O que o compose configura automaticamente

Para a API, o compose já injeta: `PORT=3000`, `DB_HOST=encomenda-service-main-db`, `DB_PORT=3306`, além de `DB_USERNAME`, `DB_PASSWORD` e `DB_DATABASE` a partir das variáveis `ENCOMENDA_DB_*`. A aplicação precisa ler essas variáveis para se conectar ao banco.

As portas são publicadas apenas em `127.0.0.1` (acessíveis somente pela própria máquina).

## Subindo o ambiente

### Todos os serviços

```bash
docker compose up -d --build
```

### Cada serviço separadamente

Como cada microserviço é independente, é possível subir um por vez. Preencha no `.env` apenas as variáveis do serviço desejado e rode:

```bash
# Somente o banco do encomenda-service
docker compose up -d encomenda-service-main-db

# A API (sobe o banco junto, pois depende dele e aguarda o healthcheck)
docker compose up -d --build encomenda-service
```
