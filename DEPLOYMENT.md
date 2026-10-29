# 🚀 Deployment - ProduPlan

Este documento estabelece o processo de implantação do sistema ProduPlan na infraestrutura On-Premise (com VM) gerenciada pelo Núcleo de Tecnologia da Informação (NTI).

Todo o ecossistema — banco de dados, backend e frontend — opera por meio de contêineres Docker isolados, com armazenamento persistente mapeado diretamente para o disco físico da Máquina Virtual.

## 1. Pré-requisitos e Acessos

Para provisionar e atualizar o ambiente, são necessários:

- Acesso à rede da Ufal, presencialmente ou via VPN institucional.
- Credenciais de acesso SSH à Máquina Virtual fornecida pelo NTI.
- Docker e Docker Compose (V2) instalados no servidor Linux (Ubuntu recomendado).
- DNS interno configurado pelo NTI (exemplo: produplan.intranet.ufal.br).
- Certificado SSL interno emitido pelo NTI, obrigatório para que o aplicativo mobile realize requisições HTTPS.

## 2. Estrutura de Diretórios e Volumes (Persistência)

Como o banco de dados armazenará apenas os caminhos das fotos (URLs), é necessário garantir que os arquivos físicos permaneçam disponíveis mesmo após a reinicialização dos contêineres.

Crie a estrutura de diretórios no disco físico da Máquina Virtual antes de iniciar a aplicação:

```bash
sudo mkdir -p /var/www/produplan/pgdata
sudo mkdir -p /var/www/produplan/uploads
sudo chown -R "$USER":"$USER" /var/www/produplan
```

### Descrição dos diretórios

| Diretório                    | Finalidade                                               |
| ---------------------------- | -------------------------------------------------------- |
| `/var/www/produplan/pgdata`  | Armazenamento persistente dos dados do PostgreSQL.       |
| `/var/www/produplan/uploads` | Armazenamento persistente das fotos enviadas ao sistema. |

Esses diretórios são utilizados como bind mounts pelo Docker, mantendo os arquivos no disco da VM mesmo quando os contêineres são recriados.

## 3. Variáveis de Ambiente (.env.prod)

Na raiz do projeto no servidor, crie o arquivo .env.prod para centralizar as configurações do banco de dados, da segurança, do armazenamento e do frontend.

```sh

# Banco de Dados

DB_HOST=db
DB_PORT=5432
DB_NAME=produplan_prod
DB_USER=produplan_admin
DB_PASSWORD=<senha-forte-gerada>

# Backend: Segurança / JWT

JWT_SECRET=<chave-secreta-longa-em-base64>
JWT_EXPIRATION=86400000
SPRING_PROFILES_ACTIVE=prod

# Backend: Diretório de uploads dentro do contêiner

UPLOAD_DIR=/app/uploads

# Frontend: URL da API

PUBLIC_API_URL=https://produplan.intranet.ufal.br:8080
NEXT_PUBLIC_API_URL=https://produplan.intranet.ufal.br:8080
```

### Recomendações de segurança

- Substitua os valores de exemplo por credenciais e chaves secretas fortes.
- Não versione o arquivo .env.prod no Git.
- Restrinja as permissões de leitura do arquivo às contas autorizadas.
- Gere o segredo JWT de acordo com os requisitos de segurança da aplicação.

## 4. Orquestração com Docker Compose

O arquivo docker-compose.prod.yml centraliza a inicialização dos serviços PostgreSQL, Spring Boot e Next.js.

O mapeamento de volumes é essencial para garantir a persistência dos dados no ambiente do NTI.

Crie o arquivo docker-compose.prod.yml na raiz do projeto:

```sh
services:
db:
image: postgres:17
restart: always
environment:
POSTGRES_USER: ${DB_USER}
POSTGRES_PASSWORD: ${DB_PASSWORD}
POSTGRES_DB: ${DB_NAME}
volumes: # Dados do banco armazenados no disco da VM - /var/www/produplan/pgdata:/var/lib/postgresql/data
healthcheck:
test: ["CMD-SHELL", "pg_isready -U ${DB_USER} -d ${DB_NAME}"]
interval: 10s
retries: 5

api:
build:
context: ./agro-backend
restart: always
ports: - "8080:8080"
environment: - SPRING_PROFILES_ACTIVE=${SPRING_PROFILES_ACTIVE}
      - UPLOAD_DIR=${UPLOAD_DIR} - SPRING_DATASOURCE_URL=jdbc:postgresql://${DB_HOST}:${DB_PORT}/${DB_NAME}
      - SPRING_DATASOURCE_USERNAME=${DB_USER} - SPRING_DATASOURCE_PASSWORD=${DB_PASSWORD}
volumes: # Fotos armazenadas no disco da VM - /var/www/produplan/uploads:/app/uploads
depends_on:
db:
condition: service_healthy

frontend:
build:
context: ./agro-frontend
restart: always
ports: - "3000:3000"
environment: - PUBLIC_API_URL=${PUBLIC_API_URL}
      - NEXT_PUBLIC_API_URL=${NEXT_PUBLIC_API_URL}
depends_on: - api
```

### 4.1. Inicialização do ambiente

Para construir as imagens e iniciar todos os serviços em segundo plano, execute:

```bash
docker compose \
 --env-file .env.prod \
 -f docker-compose.prod.yml \
 up -d --build
```

### 4.2. Verificação dos contêineres

Para verificar o estado dos serviços:

```bash
docker compose \
 --env-file .env.prod \
 -f docker-compose.prod.yml \
 ps
```

## 5. Rede e Proxy Reverso (Nginx/SSL)

O aplicativo mobile para Android e iOS requer conexões seguras via HTTPS. Como o ambiente opera em uma rede restrita, é necessário configurar o acesso seguro dentro da infraestrutura institucional.

### 5.1. Configuração inicial

1. Instale o Nginx na própria VM do NTI.
2. Solicite ao NTI os arquivos do certificado SSL (`.crt` e `.key`) válidos para a intranet.
3. Configure o Nginx para escutar na porta `443` (HTTPS).
4. Configure o redirecionamento das requisições:
   - `/api` para o backend, na porta `8080`.
   - `/` para o frontend, na porta `3000`.
5. Confirme que o DNS interno aponta para o endereço correto da VM.
6. Verifique se os dispositivos clientes confiam na autoridade certificadora que emitiu o certificado.

### 5.2. Fluxo de comunicação

| Componente            | Porta interna | Finalidade                            |
| --------------------- | ------------: | ------------------------------------- |
| Nginx                 |         `443` | Recepção das conexões HTTPS.          |
| Backend (Spring Boot) |        `8080` | Processamento das requisições da API. |
| Frontend (Next.js)    |        `3000` | Interface web do sistema.             |
| PostgreSQL            |        `5432` | Persistência dos dados.               |

O Nginx atua como proxy reverso, encaminhando as requisições para o serviço correspondente.

---

## 6. Rotinas de Deploy Manual e Manutenção

Como ferramentas de integração e implantação contínuas (CI/CD), como o GitHub Actions, podem não ter acesso SSH através do firewall da Ufal, as atualizações são realizadas manualmente ou por meio de tarefas agendadas dentro da rede institucional.

### 6.1. Atualização do sistema (Deploy)

Conecte-se à VM via SSH:

```bash
ssh usuario@ip-da-vm-nti
```

Navegue até o diretório do projeto:

```bash
cd /caminho/do/projeto/produplan
```

Atualize o código-fonte a partir da branch main:

```bash
git pull origin main
```

Reconstrua e reinicie os contêineres do backend e do frontend:

```bash
docker compose \
  --env-file .env.prod \
  -f docker-compose.prod.yml \
  up -d --build api frontend
```

Esse comando atualiza os serviços especificados sem solicitar a recriação explícita do serviço de banco de dados.

⚠️ Atenção: a execução desse comando não garante ausência de indisponibilidade. Alterações incompatíveis na API, no frontend ou no esquema do banco podem exigir procedimentos adicionais de migração e validação.

### 6.2. Monitoramento de logs

Para acompanhar os logs do backend em tempo real:

```bash
docker compose \
  --env-file .env.prod \
  -f docker-compose.prod.yml \
  logs -f api
```

Para acompanhar os logs de todos os serviços:

```bash
docker compose \
  --env-file .env.prod \
  -f docker-compose.prod.yml \
  logs -f
```

### 6.3. Backup do banco de dados

Crie previamente o diretório de backups:

```bash
sudo mkdir -p /var/www/produplan/backups
sudo chown -R "$USER":"$USER" /var/www/produplan/backups
```

Para gerar um backup lógico do PostgreSQL em formato customizado:

```bash
docker compose \
  --env-file .env.prod \
  -f docker-compose.prod.yml \
  exec -T db \
  pg_dump -U produplan_admin -d produplan_prod -F c \
  > /var/www/produplan/backups/db_backup_$(date +%F).dump
```

Esse comando grava o backup diretamente no sistema de arquivos da VM, no diretório de backups.

### 6.4. Backup dos arquivos enviados

Para compactar o diretório de fotos:

```bash
tar -czvf \
 /var/www/produplan/backups/uploads*backup*$(date +%F).tar.gz \
 -C /var/www/produplan uploads
```

O arquivo resultante conterá os uploads armazenados no diretório persistente da aplicação.

### 6.5. Agendamento de backups

Recomenda-se configurar uma tarefa agendada (cron job) para executar os backups periodicamente.

O procedimento de backup deve contemplar:

- Banco de dados PostgreSQL.
- Arquivos de upload.
- Retenção de cópias antigas.
- Armazenamento de cópias em local separado da VM.
- Monitoramento de falhas.
- Testes periódicos de restauração.

  ⚠️ Importante: os backups devem ser protegidos contra acesso não autorizado. Sempre que possível, mantenha uma cópia externa à VM para reduzir o risco de perda simultânea do sistema e dos backups.

## 7. Checklist de Implantação

Antes de considerar a implantação concluída, verifique os seguintes itens:

- Acesso SSH à VM funcionando.
- Docker e Docker Compose instalados e operacionais.
- Diretórios persistentes criados e com permissões adequadas.
- Arquivo .env.prod configurado e protegido.
- Arquivo docker-compose.prod.yml validado.
- Banco de dados inicializado e saudável.
- Backend iniciado e conectado ao PostgreSQL.
- Frontend iniciado e acessível.
- DNS interno configurado pelo NTI.
- Certificado SSL válido e confiável pelos dispositivos clientes.
- Proxy reverso Nginx configurado.
- Comunicação entre frontend, backend e banco de dados validada.
- Upload e recuperação de fotos testados.
- Backup do banco de dados executado e validado.
- Backup dos arquivos enviados executado e validado.
- Procedimento de restauração documentado e testado.

## 8. Considerações Finais

A implantação do ProduPlan no ambiente do NTI utiliza uma arquitetura conteinerizada, com persistência dos dados e arquivos no disco físico da Máquina Virtual.

A operação segura e confiável do sistema depende de:

- Configuração adequada da rede institucional.
- Proteção das credenciais e dos segredos da aplicação.
- Uso de HTTPS com certificado confiável pelos dispositivos clientes.
- Persistência dos dados fora do ciclo de vida dos contêineres.
- Monitoramento contínuo dos serviços.
- Execução regular de backups e testes de restauração.

A manutenção deve seguir os procedimentos institucionais do NTI, respeitando as políticas de segurança, acesso e disponibilidade da infraestrutura.
