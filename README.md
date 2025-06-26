# 🚀 Guia Completo de Configuração - MCPs Integrados

## 📋 Sumário
- [Visão Geral](#visão-geral)
- [Pré-requisitos](#pré-requisitos)
- [Configuração Azure DevOps](#configuração-azure-devops)
- [Configuração Gmail](#configuração-gmail)
- [Configuração Browser](#configuração-browser)
- [Configuração Gemini AI](#configuração-gemini-ai)
- [Servidor Integrado](#servidor-integrado)
- [Configuração Claude Code](#configuração-claude-code)
- [Teste e Validação](#teste-e-validação)
- [Solução de Problemas](#solução-de-problemas)
- [Comandos Disponíveis](#comandos-disponíveis)

---

## 🎯 Visão Geral

Este guia ensina como configurar um **servidor MCP integrado** que combina 4 funcionalidades:
- **🏢 Azure DevOps** - Gestão de projetos, work items, repositórios
- **📧 Gmail** - Acesso a emails, labels, busca
- **🌐 Browser** - Navegação web, busca, screenshots
- **🤖 Gemini AI** - Geração de texto, análise de código, tradução

**Resultado:** Todos os MCPs funcionando nativamente no Claude Code através de um único servidor Node.js.

---

## ⚙️ Pré-requisitos

### 🛠️ Software Necessário
```bash
# Node.js (versão 18 ou superior)
node --version  # v18.0.0+

# Python (versão 3.11 ou 3.12)
python3 --version  # 3.11.x ou 3.12.x

# Azure CLI
az --version

# Git
git --version

# Claude Code
# Instalar via: https://claude.ai/code
```

### 🔑 Credenciais Necessárias
- **Azure DevOps:** Acesso à organização Azure DevOps
- **Gmail:** Conta Google com API habilitada
- **Gemini AI:** Chave de API do Google AI Studio

---

## 🏢 Configuração Azure DevOps

### 1. Autenticação Azure CLI
```bash
# Login no Azure
az login --tenant SEU_TENANT.onmicrosoft.com

# Configurar organização padrão
az devops configure --defaults organization=https://dev.azure.com/SUA_ORGANIZACAO/

# Instalar extensão Azure DevOps
az extension add --name azure-devops
```

### 2. Verificar Acesso
```bash
# Testar listagem de projetos
az devops project list --organization https://dev.azure.com/SUA_ORGANIZACAO/

# Verificar token de acesso
az account get-access-token --resource 499b84ac-1321-427f-aa17-267ca6975798
```

---

## 📧 Configuração Gmail

### 1. Preparar Diretório Gmail
```bash
# Criar diretório para MCP Gmail
mkdir -p ~/workspace/mcp-servers/gmail
cd ~/workspace/mcp-servers/gmail
```

### 2. Configurar Google Cloud Project
1. Acesse [Google Cloud Console](https://console.cloud.google.com)
2. Crie um novo projeto ou selecione existente
3. Habilite a **Gmail API**:
   - APIs & Services → Library
   - Busque "Gmail API" → Enable

### 3. Criar Credenciais OAuth2
1. APIs & Services → Credentials
2. Create Credentials → OAuth 2.0 Client ID
3. Application Type: **Desktop Application**
4. Nome: "MCP Gmail Client"
5. Download o arquivo JSON → salvar como `credentials.json`

### 4. Configurar Ambiente Python
```bash
# Criar virtual environment
python3 -m venv venv
source venv/bin/activate  # Linux/Mac
# ou venv\Scripts\activate  # Windows

# Instalar dependências
pip install google-auth google-auth-oauthlib google-api-python-client
```

### 5. Autenticar Gmail
```bash
# Criar script de autenticação
cat > test_auth.py << 'EOF'
import os
import json
from google.auth.transport.requests import Request
from google.oauth2.credentials import Credentials
from google_auth_oauthlib.flow import InstalledAppFlow
from googleapiclient.discovery import build

SCOPES = [
    'https://www.googleapis.com/auth/gmail.readonly',
    'https://www.googleapis.com/auth/gmail.labels'
]

def authenticate_gmail():
    creds = None
    
    if os.path.exists('token.json'):
        creds = Credentials.from_authorized_user_file('token.json', SCOPES)
    
    if not creds or not creds.valid:
        if creds and creds.expired and creds.refresh_token:
            creds.refresh(Request())
        else:
            flow = InstalledAppFlow.from_client_secrets_file('credentials.json', SCOPES)
            creds = flow.run_local_server(port=0)
        
        with open('token.json', 'w') as token:
            token.write(creds.to_json())
    
    service = build('gmail', 'v1', credentials=creds)
    
    # Testar conexão
    results = service.users().labels().list(userId='me').execute()
    labels = results.get('labels', [])
    
    print(f"✅ Gmail autenticado com sucesso!")
    print(f"📧 Conta: {service.users().getProfile(userId='me').execute()['emailAddress']}")
    print(f"🏷️ Labels encontrados: {len(labels)}")
    
    return service

if __name__ == '__main__':
    authenticate_gmail()
EOF

# Executar autenticação
python test_auth.py
```

---

## 🌐 Configuração Browser

### 1. Preparar Diretório Browser
```bash
mkdir -p ~/workspace/mcp-servers/browser
cd ~/workspace/mcp-servers/browser
```

### 2. Instalar Dependências
```bash
# Criar virtual environment
python3 -m venv venv
source venv/bin/activate

# Instalar dependências
pip install playwright beautifulsoup4 requests lxml

# Instalar browsers do Playwright
playwright install chromium
```

### 3. Teste do Browser
```bash
# Criar script de teste
cat > test_browser.py << 'EOF'
from playwright.sync_api import sync_playwright
import requests

def test_browser():
    print("🌐 Testando Browser MCP...")
    
    # Teste 1: Playwright
    try:
        with sync_playwright() as p:
            browser = p.chromium.launch()
            page = browser.new_page()
            page.goto("https://example.com")
            title = page.title()
            browser.close()
            print(f"✅ Playwright funcionando - Título: {title}")
    except Exception as e:
        print(f"❌ Erro Playwright: {e}")
    
    # Teste 2: Requests
    try:
        response = requests.get("https://httpbin.org/get", timeout=10)
        if response.status_code == 200:
            print("✅ Requests funcionando")
        else:
            print(f"❌ Requests erro: {response.status_code}")
    except Exception as e:
        print(f"❌ Erro Requests: {e}")

if __name__ == '__main__':
    test_browser()
EOF

# Executar teste
python test_browser.py
```

---

## 🤖 Configuração Gemini AI

### 1. Obter API Key
1. Acesse [Google AI Studio](https://makersuite.google.com/app/apikey)
2. Clique em "Create API Key"
3. Copie a chave gerada

### 2. Preparar Diretório Gemini
```bash
mkdir -p ~/workspace/mcp-servers/gemini
cd ~/workspace/mcp-servers/gemini
```

### 3. Configurar Gemini
```bash
# Criar virtual environment
python3 -m venv venv
source venv/bin/activate

# Instalar dependências
pip install google-generativeai

# Configurar API Key
echo "GEMINI_API_KEY=SUA_API_KEY_AQUI" > .env

# Criar script de teste
cat > test_gemini.py << 'EOF'
import os
import google.generativeai as genai
from dotenv import load_dotenv

load_dotenv()

def test_gemini():
    print("🤖 Testando Gemini AI...")
    
    api_key = os.getenv('GEMINI_API_KEY')
    if not api_key:
        print("❌ API Key não encontrada!")
        return
    
    try:
        genai.configure(api_key=api_key)
        
        model = genai.GenerativeModel('gemini-1.5-flash')
        response = model.generate_content("Diga olá em português")
        
        print(f"✅ Gemini funcionando!")
        print(f"🤖 Resposta: {response.text}")
        
    except Exception as e:
        print(f"❌ Erro Gemini: {e}")

if __name__ == '__main__':
    test_gemini()
EOF

# Instalar python-dotenv
pip install python-dotenv

# Executar teste
python test_gemini.py
```

---

## 🔧 Servidor Integrado

### 1. Clonar Repositório
```bash
# Navegar para diretório de trabalho
cd ~/workspace/projects/javascript

# Clonar o repositório MCP integrado
git clone https://github.com/SEU_USUARIO/azure-devops-mcp.git
cd azure-devops-mcp
```

### 2. Instalar Dependências Node.js
```bash
# Instalar dependências
npm install

# Verificar estrutura
ls -la src/tools/
# Deve mostrar: core.ts, workitems.ts, gmail.ts, browser.ts, gemini.ts, etc.
```

### 3. Configurar Caminhos
Edite os arquivos de proxy para apontar para os diretórios corretos:

```typescript
// src/tools/gmail.ts
const gmailPath = "/Users/SEU_USUARIO/workspace/mcp-servers/gmail";

// src/tools/browser.ts  
const browserPath = "/Users/SEU_USUARIO/workspace/mcp-servers/browser";

// src/tools/gemini.ts
const geminiPath = "/Users/SEU_USUARIO/workspace/mcp-servers/gemini";
```

### 4. Build do Servidor
```bash
# Compilar TypeScript
npm run build

# Verificar se compilou
ls -la dist/
```

---

## ⚙️ Configuração Claude Code

### 1. Localizar Arquivo de Configuração
```bash
# Localizar arquivo de configuração do Claude
# Mac:
open ~/Library/Application\ Support/Claude/

# Linux:
~/.config/claude/

# Windows:
%APPDATA%/Claude/
```

### 2. Editar claude_desktop_config.json
```json
{
  "mcpServers": {
    "azure-devops-unified": {
      "command": "node",
      "args": [
        "/Users/SEU_USUARIO/workspace/projects/javascript/azure-devops-mcp/dist/index.js",
        "SUA_ORGANIZACAO_AZURE_DEVOPS"
      ],
      "env": {
        "NODE_ENV": "production"
      }
    }
  }
}
```

### 3. Reiniciar Claude Code
```bash
# Fechar completamente o Claude Code
# Reabrir Claude Code
# Os MCPs devem aparecer automaticamente
```

---

## ✅ Teste e Validação

### 1. Verificar Conexão MCPs
No Claude Code, digite:
```
ajuda mcp
```

Deve mostrar o banner com todos os 4 MCPs disponíveis.

### 2. Testar Cada MCP

**Azure DevOps:**
```
liste meus projetos azure devops
```

**Gmail:**
```
liste meus labels do gmail
```

**Browser:**
```
busque na web: Claude Code tips
```

**Gemini:**
```
generate com gemini: diga olá em português
```

### 3. Teste Integrado
```
mostre dashboard do projeto SEU_PROJETO
```

---

## 🚨 Solução de Problemas

### Problema: MCPs não aparecem no Claude Code
**Solução:**
```bash
# 1. Verificar se o servidor compila
cd azure-devops-mcp
npm run build

# 2. Verificar configuração
cat ~/Library/Application\ Support/Claude/claude_desktop_config.json

# 3. Testar manualmente
node dist/index.js SUA_ORGANIZACAO
```

### Problema: Erro de autenticação Azure
**Solução:**
```bash
# Reautenticar
az logout
az login --tenant SEU_TENANT.onmicrosoft.com
```

### Problema: Gmail não funciona
**Solução:**
```bash
cd ~/workspace/mcp-servers/gmail
source venv/bin/activate
python test_auth.py
```

### Problema: Python não encontrado
**Solução:**
```bash
# Verificar versão Python
python3 --version

# Se necessário, instalar pyenv
curl https://pyenv.run | bash
pyenv install 3.12.8
pyenv global 3.12.8
```

---

## 📚 Comandos Disponíveis

### 🏢 Azure DevOps
```bash
# Projetos
liste meus projetos azure devops
liste desenvolvedores do projeto [NOME]

# Work Items  
liste tarefas em progresso do projeto [NOME]
liste tarefas fechadas do projeto [NOME]
mostre dashboard do projeto [NOME]

# Repositórios
liste repositórios do projeto [NOME]
mostre pull requests do projeto [NOME]
mostre builds do projeto [NOME]
```

### 📧 Gmail
```bash
liste meus labels do gmail
mostre meus emails recentes
busque emails de [REMETENTE]
leia o email [ID]
```

### 🌐 Browser
```bash
busque na web: [TERMO]
busque o conteúdo desta URL: [URL]
extraia conteúdo de: [URL]
capture screenshot de: [URL]
```

### 🤖 Gemini AI
```bash
generate com gemini: [PROMPT]
analise este código com gemini: [CÓDIGO]
traduza com gemini: [TEXTO]
resuma com gemini: [TEXTO LONGO]
gemini chat: [MENSAGEM]
```

---

## 🎯 Conclusão

Após seguir este guia, você terá:

✅ **Servidor MCP integrado** funcionando  
✅ **4 MCPs** ativos (Azure DevOps, Gmail, Browser, Gemini)  
✅ **Claude Code** conectado e funcional  
✅ **Comandos em português** disponíveis  
✅ **Gestão completa** de projetos e desenvolvimento  

**🚀 Agora você pode gerenciar seus projetos, emails, pesquisas web e IA diretamente no Claude Code!**

---

## 📞 Suporte

Para dúvidas ou problemas:
1. Verificar seção [Solução de Problemas](#solução-de-problemas)
2. Consultar logs: `node dist/index.js SUA_ORG`
3. Verificar configurações de cada MCP individualmente

**Última atualização:** Dezembro 2024  
**Versão:** 1.0.0
