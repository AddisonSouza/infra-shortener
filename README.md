<p align="center">
  <img src="https://img.shields.io/badge/🔗-URL%20Shortener-blue?style=for-the-badge&logoColor=white" alt="URL Shortener Badge"/>
</p>

<h1 align="center">🚀 Encurtador de URLs de Alta Performance</h1>

<p align="center">
  <strong>Transforme links gigantes em URLs compactas e elegantes</strong>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white" alt="Docker"/>
  <img src="https://img.shields.io/badge/Node.js-339933?style=flat-square&logo=nodedotjs&logoColor=white" alt="Node.js"/>
  <img src="https://img.shields.io/badge/Cassandra-1287B1?style=flat-square&logo=apachecassandra&logoColor=white" alt="Cassandra"/>
  <img src="https://img.shields.io/badge/Redis-DC382D?style=flat-square&logo=redis&logoColor=white" alt="Redis"/>
  <img src="https://img.shields.io/badge/Nginx-009639?style=flat-square&logo=nginx&logoColor=white" alt="Nginx"/>
</p>

<p align="center">
  <a href="#-sobre-o-projeto">Sobre</a> •
  <a href="#-tecnologias">Tecnologias</a> •
  <a href="#-instalação">Instalação</a> •
  <a href="#-desafios">Desafios</a>
</p>

---

## 🎯 Sobre o Projeto

Um **encurtador de URLs profissional** construído para escalar! Esta aplicação permite que você transforme URLs longas e complexas em links curtos de apenas **6 caracteres alfanuméricos**.

### ✨ Features Principais

| Feature | Descrição |
|---------|-----------|
| 🔗 **Encurtamento Inteligente** | Gera códigos únicos de 6 caracteres usando Base62 |
| ⚡ **Redirecionamento Ultra-Rápido** | Cache Redis para acesso instantâneo |
| 🛡️ **Segurança** | Proteção via API Key e configurações CORS |
| 📊 **Escalabilidade** | Arquitetura distribuída com Cassandra |
| 🐳 **Containerizado** | Deploy simplificado com Docker Compose |

---

## 🛠️ Tecnologias

<table>
  <tr>
    <td align="center" width="96">
      <img src="https://skillicons.dev/icons?i=docker" width="48" height="48" alt="Docker" />
      <br>Docker
    </td>
    <td align="center" width="96">
      <img src="https://skillicons.dev/icons?i=nodejs" width="48" height="48" alt="Node.js" />
      <br>Node.js
    </td>
    <td align="center" width="96">
      <img src="https://skillicons.dev/icons?i=cassandra" width="48" height="48" alt="Cassandra" />
      <br>Cassandra
    </td>
    <td align="center" width="96">
      <img src="https://skillicons.dev/icons?i=redis" width="48" height="48" alt="Redis" />
      <br>Redis
    </td>
    <td align="center" width="96">
      <img src="https://skillicons.dev/icons?i=nginx" width="48" height="48" alt="Nginx" />
      <br>Nginx
    </td>
  </tr>
</table>

### Stack Completa

| Camada | Tecnologia | Propósito |
|--------|------------|-----------|
| **Frontend** | Web App | Interface amigável para encurtar URLs |
| **Backend** | Node.js | API REST para criação e redirecionamento |
| **Banco de Dados** | Apache Cassandra 4.1 | Armazenamento distribuído e escalável |
| **Cache** | Redis 7 Alpine | Cache de alta performance para redirecionamentos |
| **Reverse Proxy** | Nginx Alpine | Roteamento e load balancing |
| **Orquestração** | Docker Compose | Gerenciamento de todos os serviços |

---

## 🚀 Instalação

### Pré-requisitos

- [Docker](https://www.docker.com/) instalado
- [Docker Compose](https://docs.docker.com/compose/) v2+
- ~2GB de RAM disponível

### Passo a Passo

```bash
# 1️⃣ Clone o repositório
git clone https://github.com/seu-usuario/infra-shortener.git
cd infra-shortener

# 2️⃣ Configure as variáveis de ambiente
cp local-env.example .env

# 3️⃣ Edite o arquivo .env com suas configurações
nano .env  # ou use seu editor favorito

# 4️⃣ Suba todos os containers
docker compose up -d

# 5️⃣ Acesse a aplicação
# 🌐 Frontend: http://localhost
# 🔌 API: http://localhost/api
```

### 🔧 Configuração do `.env`

```env
# Gere valores seguros para produção!
SALT_BASE_62=sua_string_aleatoria_longa_aqui
API_KEY=sua_chave_api_forte_aqui

# Configure o CORS para seu domínio
CORS_ALLOWED_ORIGINS=http://seu-dominio.com
```

### 📋 Verificar se está funcionando

```bash
# Ver status dos containers
docker compose ps

# Ver logs em tempo real
docker compose logs -f

# Testar a API
curl -X POST http://localhost/api/shorten \
  -H "Content-Type: application/json" \
  -H "x-api-key: SUA_API_KEY" \
  -d '{"url": "https://www.exemplo.com/url-muito-longa"}'
```

---

## 💪 Desafios Enfrentados

### 🔴 Desafio 1: Sincronização do Cassandra na Inicialização

**Problema:** O backend tentava conectar antes do Cassandra estar pronto, causando falhas.

**Solução:** 
- Implementamos `healthcheck` robusto no Cassandra
- Criamos um container `cassandra-init` que só executa após o banco estar saudável
- Utilizamos `depends_on` com condições específicas (`service_healthy`, `service_completed_successfully`)

```yaml
depends_on:
  cassandra:
    condition: service_healthy
```

---

### 🔴 Desafio 2: Configuração de Memória no Docker

**Problema:** Cassandra consumia toda a memória disponível, derrubando outros serviços.

**Solução:**
- Definimos `mem_limit` para cada container
- Configuramos `MAX_HEAP_SIZE` e `HEAP_NEWSIZE` específicos para o Cassandra
- Balanceamos recursos: Cassandra (768MB), Backend (512MB), Frontend (256MB), Redis (256MB)

---

### 🔴 Desafio 3: Roteamento Nginx para URLs Curtas

**Problema:** Diferenciar entre rotas do frontend, API e URLs encurtadas.

**Solução:**
- Regex pattern `"^/[a-zA-Z0-9]{6}$"` para capturar apenas códigos de 6 caracteres
- Rotas `/api/` direcionadas ao backend
- Todo o resto vai para o frontend

```nginx
location ~ "^/[a-zA-Z0-9]{6}$" {
    proxy_pass http://backend:3000;
}
```

---

### 🔴 Desafio 4: Latência no Redirecionamento

**Problema:** Buscar a URL no Cassandra a cada redirecionamento era lento.

**Solução:**
- Implementamos Redis como camada de cache
- URLs mais acessadas ficam em memória
- Tempo de resposta reduzido de ~50ms para ~2ms

---

## 📊 Arquitetura

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Cliente   │────▶│    Nginx    │────▶│  Frontend   │
└─────────────┘     │  (Proxy)    │     └─────────────┘
                    │             │
                    │   /api/*    │     ┌─────────────┐
                    │   /AbCd12   │────▶│   Backend   │
                    └─────────────┘     │  (Node.js)  │
                                        └──────┬──────┘
                                               │
                              ┌────────────────┼────────────────┐
                              │                │                │
                              ▼                ▼                ▼
                        ┌──────────┐    ┌──────────┐    ┌──────────┐
                        │  Redis   │    │Cassandra │    │  Cache   │
                        │  Cache   │    │    DB    │    │  Layer   │
                        └──────────┘    └──────────┘    └──────────┘
```

<p align="center">
  <strong>⭐ Se este projeto te ajudou, deixe uma estrela!</strong>
</p>

<p align="center">
  Feito com ❤️ e muito ☕
</p>
