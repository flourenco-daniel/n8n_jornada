# Documentação do docker-compose.yml

## Visão Geral
Este `docker-compose.yml` define um ambiente com dois serviços principais:

1. **Traefik**: Um proxy reverso que gerencia o roteamento de tráfego e a emissão de certificados SSL.
2. **n8n**: Uma plataforma de automação de fluxos de trabalho auto-hospedada.

O Traefik gerencia as conexões HTTP e HTTPS, garantindo comunicação segura e facilitando a descoberta de serviços no Docker.

---
## Serviços

### 1. Traefik

**Imagem:** `traefik`

**Função:**
- Atua como um proxy reverso para gerenciar as conexões HTTP/HTTPS.
- Implementa a renovação automática de certificados SSL usando Let's Encrypt.
- Expõe uma interface de administração (insegura por padrão).

**Configurações principais:**
- Porta `80` (HTTP) redirecionada para `443` (HTTPS).
- Certificados gerenciados automaticamente via Let's Encrypt.
- Detecta automaticamente serviços Docker com rótulos adequados.
- Usa `/var/run/docker.sock` para monitorar containers.

**Volumes:**
- `traefik_data`: Armazena os certificados SSL gerados.
- `/var/run/docker.sock`: Permite que Traefik leia os serviços do Docker.

**Variáveis de Ambiente:**
- `${SSL_EMAIL}`: Endereço de e-mail para registro no Let's Encrypt.

### 2. n8n

**Imagem:** `docker.n8n.io/n8nio/n8n`

**Função:**
- Executa a plataforma de automação n8n.

**Configurações principais:**
- Porta local `5678` mapeada apenas para `127.0.0.1`, restringindo acesso externo.
- Utiliza rótulos Traefik para expor o serviço via domínio especificado.
- Define variáveis de ambiente essenciais para o funcionamento.

**Volumes:**
- `n8n_data`: Armazena dados e configurações do n8n.
- `./local-files:/files`: Diretório local montado para armazenar arquivos.

**Variáveis de Ambiente:**
- `N8N_HOST=localhost`
- `N8N_PORT=5678`
- `N8N_PROTOCOL=http`
- `NODE_ENV=production`
- `WEBHOOK_URL=http://localhost`
- `GENERIC_TIMEZONE=Europe/Berlin`

---
## Volumes
- `traefik_data`: Armazena os certificados SSL gerenciados pelo Traefik.
- `n8n_data`: Armazena os dados e configurações persistentes do n8n.

---
## Uso
1. Certifique-se de definir as variáveis `${SSL_EMAIL}`, `${SUBDOMAIN}` e `${DOMAIN_NAME}`.
2. Execute o ambiente com:
   ```sh
   docker-compose up -d
   ```
3. Acesse a interface do Traefik (insegura) em `http://localhost:8080` (caso a porta tenha sido configurada).
4. Acesse o n8n via `https://{SUBDOMAIN}.{DOMAIN_NAME}`.

---
## Notas
- A interface do Traefik está configurada como insegura (`--api.insecure=true`). Para produção, recomenda-se desativá-la ou protegê-la.
- O `N8N_HOST` está definido como `localhost`, o que pode exigir ajustes se for acessado externamente.
- O armazenamento de certificados SSL é persistente no volume `traefik_data`.

Essa configuração fornece um ambiente seguro e escalável para automação com n8n, utilizando Traefik como proxy reverso.

