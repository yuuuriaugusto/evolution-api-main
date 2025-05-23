## Pré-requisitos

Antes de começar, certifique-se de ter os seguintes itens instalados em sua máquina:

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)
- Uma conta no [Ngrok](https://ngrok.com/) (necessária para expor o n8n publicamente)

## Passo a Passo para Instalação

### 1. Clone o Repositório

Clone este repositório para sua máquina local:


### 2. Crie uma Conta no Ngrok

- Acesse [ngrok.com](https://ngrok.com/) e crie uma conta gratuita.
- Após o login, vá até o painel e copie seu **Authtoken** (exemplo: `1a2b3c4d5e6f7g8h9i0j_xxxxxxxxxxxxxxxxxxxxxx`).

### 3. Configure o Arquivo `.env`

Copie o arquivo de exemplo `.env.example` (se disponível) ou crie um arquivo `.env` na raiz do projeto com base no modelo abaixo. Substitua os valores conforme necessário:

```env
# Variáveis compartilhadas para o Postgres
POSTGRES_USER=postgres
POSTGRES_PASSWORD=senha_postgres
POSTGRES_DB=app_db

# Variáveis do n8n
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=usuario
N8N_BASIC_AUTH_PASSWORD=senha
DB_TYPE=postgresdb
DB_POSTGRESDB_HOST=postgres
DB_POSTGRESDB_PORT=5432
DB_POSTGRESDB_USER=${POSTGRES_USER}
DB_POSTGRESDB_PASSWORD=${POSTGRES_PASSWORD}
DB_POSTGRESDB_DATABASE=${POSTGRES_DB}

# Variáveis do evolution-api
AUTHENTICATION_API_KEY=SUA_KEY
DATABASE_ENABLED=true
DATABASE_PROVIDER=postgresql
DATABASE_CONNECTION_URI=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgres:5432/${POSTGRES_DB}?schema=public
DATABASE_CONNECTION_CLIENT_NAME=evolution_exchange
DATABASE_SAVE_DATA_INSTANCE=true
DATABASE_SAVE_DATA_NEW_MESSAGE=true
DATABASE_SAVE_MESSAGE_UPDATE=true
DATABASE_SAVE_DATA_CONTACTS=true
DATABASE_SAVE_DATA_CHATS=true
DATABASE_SAVE_DATA_LABELS=true
DATABASE_SAVE_DATA_HISTORIC=true

# Variáveis do Redis (evolution-api)
CACHE_REDIS_ENABLED=true
CACHE_REDIS_URI=redis://redis:6379/6
CACHE_REDIS_PREFIX_KEY=evolution
CACHE_REDIS_SAVE_INSTANCES=false

# Variáveis do cache local (evolution-api)
CACHE_LOCAL_ENABLED=false
```

- Substitua `SUA_KEY` em `AUTHENTICATION_API_KEY` por uma chave de API segura para o Evolution API.
- Substitua `SUA_KEY` em `NGROK_AUTHTOKEN` pelo token obtido no passo 2.
- Ajuste `POSTGRES_PASSWORD`, `N8N_BASIC_AUTH_USER` e `N8N_BASIC_AUTH_PASSWORD` para valores seguros de sua escolha.
- Substitua sua SUA_KEY do ngrok no arquivo ngrok.yml

### 4. Inicie os Serviços

Execute o comando abaixo para subir todos os serviços:

```bash
docker-compose up -d
```

Isso iniciará os seguintes serviços:

- **n8n**: Ferramenta de automação de fluxos de trabalho (porta 5678).
- **postgres**: Banco de dados PostgreSQL com suporte a vetores (porta 5432).
- **evolution-api**: API para integração de mensagens (porta 8080).
- **ngrok**: Túnel para expor o n8n publicamente (porta 4040).
- **redis**: Cache para o Evolution API (porta 6380).
- **adminer**: Interface web para gerenciar o PostgreSQL (porta 8081).

### 5. Verifique os Serviços

- Acesse o **n8n** em `http://localhost:5678` com as credenciais definidas em `N8N_BASIC_AUTH_USER` e `N8N_BASIC_AUTH_PASSWORD`.
- Acesse o **Evolution API** em `http://localhost:8080`.
- Verifique o túnel do **Ngrok** em `http://localhost:4040` para obter a URL pública do n8n.

### 6. Configure os Webhooks

Como este projeto utiliza o Ngrok, a URL pública gerada para o n8n muda a cada vez que os serviços são reiniciados. Você precisará:

- Acessar `http://localhost:4040` para obter a nova URL do Ngrok (exemplo: `https://abcd-1234.ngrok.io`).
- Atualizar a URL do webhook no **Evolution API** (geralmente em suas configurações ou endpoints de integração).
- Se você configurou webhooks em outros serviços ou fluxos do n8n que dependem dessa URL, atualize-os também.

**Lembrete**: Sempre redefina as URLs dos webhooks após reiniciar os serviços para garantir que tudo funcione corretamente.

### 7. Pare os Serviços (Opcional)

Para parar os serviços, use:

```bash
docker-compose down
```

Para parar e remover os volumes (limpar dados), use:

```bash
docker-compose down -v
```

## Estrutura dos Serviços

- **n8n**: Ferramenta de automação conectada ao PostgreSQL.
- **Evolution API**: API de mensagens com suporte a Redis e PostgreSQL.
- **PostgreSQL**: Banco de dados principal com suporte a vetores (via `pgvector`).
- **Redis**: Cache para o Evolution API.
- **Ngrok**: Expõe o n8n para acesso externo.

## Notas

- Certifique-se de que as portas (5678, 5432, 8080, 4040, 6380) estejam livres em sua máquina.
- O Ngrok fornece uma URL temporária. Para uma URL fixa, considere um plano pago ou outra solução de túnel.
- Mantenha o arquivo `.env` seguro e não o compartilhe publicamente.