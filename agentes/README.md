# Voomp IA · Criar Agente de Checkout

Interface para criação de agentes de IA no checkout de produtos Voomp, integrada à Cogna IA via N8N.

---

## Estrutura do repositório

```
github-pages/
├── index.html                  ← Interface completa (GitHub Pages)
├── n8n-listar-produtos.json    ← Workflow N8N: listagem de produtos
├── n8n-cogna-criar-agente.json ← Workflow N8N: criação do agente
└── README.md
```

---

## 1. Deploy no GitHub Pages

### 1.1 Criar repositório

1. Acesse [github.com/new](https://github.com/new)
2. Nome sugerido: `voomp-ia-agentes`
3. Visibilidade: **Public** (GitHub Pages gratuito exige público)
4. Clique em **Create repository**

### 1.2 Subir os arquivos

```bash
cd /Users/lucashenrique/VoompKiro/github-pages

git init
git add .
git commit -m "feat: Voomp IA - criar agente de checkout"
git branch -M main
git remote add origin https://github.com/SEU-USUARIO/voomp-ia-agentes.git
git push -u origin main
```

### 1.3 Ativar GitHub Pages

1. No repositório → **Settings** → **Pages**
2. Source: **Deploy from a branch**
3. Branch: `main` / `/ (root)`
4. Clique em **Save**

Após ~1 minuto, o site estará em:
```
https://SEU-USUARIO.github.io/voomp-ia-agentes/
```

---

## 2. Importar workflows no N8N

### 2.1 Credencial MySQL

Antes de importar, crie a credencial do banco de produção:

1. No N8N → **Credentials** → **Add credential** → **MySQL**
2. Preencha:

| Campo    | Valor                          |
|----------|-------------------------------|
| Name     | `Voomp Production DB`         |
| Host     | `10.84.4.87`                  |
| Port     | `3306`                        |
| Database | `voompcreators_back_prd`      |
| User     | `lucas_duarte`                |
| Password | (ver `.env` → `DB_PASSWORD`)  |

### 2.2 Importar workflow de produtos

1. N8N → **Workflows** → `+` → **Import from file**
2. Selecione `n8n-listar-produtos.json`
3. Ative o workflow (toggle no canto superior direito)
4. Copie a URL do webhook: `https://SEU-N8N/webhook/produtos`

**Testar:**
```
GET https://SEU-N8N/webhook/produtos?seller_id=667209
```
Retorno esperado:
```json
{
  "products": [
    { "id": "16885", "name": "Mentoria Individual LinkeDin", ... }
  ],
  "total": 1
}
```

### 2.3 Importar workflow de criação de agente

1. N8N → **Workflows** → `+` → **Import from file**
2. Selecione `n8n-cogna-criar-agente.json`
3. Ative o workflow
4. Copie a URL do webhook: `https://SEU-N8N/webhook/criar-agente`

> Os 4 nós marcados com 🔲 são placeholders documentados.
> Cada um tem no campo **Notes** o endpoint, headers e body exatos para implementar a integração Cogna IA.

---

## 3. Configurar o index.html

Abra `index.html` e edite o bloco `CFG` no topo do `<script>`:

```js
var CFG = {
  sellerId:       '667209',
  sellerName:     'Lucas Henrique Duarte',
  n8nProdutos:    'https://SEU-N8N/webhook/produtos',     // ← URL real
  n8nCriarAgente: 'https://SEU-N8N/webhook/criar-agente', // ← URL real
  catalogFallback: [...]  // mantido como fallback offline
};
```

Faça commit e push:
```bash
git add index.html
git commit -m "config: URLs N8N produção"
git push
```

---

## 4. Fluxo completo

```
Usuário acessa GitHub Pages
       ↓
Seleciona produto (lista via N8N → MySQL → voompcreators_back_prd)
       ↓
Sobe documentos do produto (drag & drop)
       ↓
Configura identidade do widget (título, cor)
       ↓
Clica "Salvar e ativar"
       ↓
POST para N8N /webhook/criar-agente
  ├── Autenticar na Cogna IA  🔲
  ├── Upload dos documentos   🔲
  ├── Criar histórico (historyId) 🔲
  └── Aguardar vetorização    🔲
       ↓
Retorna { agentId, historyId, widgetConfig }
       ↓
Dashboard "Meus agentes" — produto aparece na lista
       ↓
Menu ⋯ → Copiar GTM ID
       ↓
Seller cola GTM ID na Edição Avançada do produto Voomp
```

---

## 5. Próximas etapas (roadmap)

| Etapa | O que fazer |
|-------|-------------|
| **Cogna IA** | Substituir os 4 nós 🔲 no workflow `n8n-cogna-criar-agente.json` pelas chamadas HTTP reais |
| **GTM API** | Adicionar nó para criar container GTM via API e retornar GTM ID real |
| **Token refresh** | Adicionar lógica de renovação automática do token Cogna IA |
| **Exit intent** | Implementar detecção de saída no widget GTM tag |
| **Captura de motivos** | Salvar motivos de abandono em Google Sheets via N8N |
| **Multi-seller** | Expandir para outros sellers além do 667209 |

---

## Credenciais relevantes

| Sistema | Onde encontrar |
|---------|---------------|
| MySQL produção | `.env` do projeto |
| Cogna IA token | `gtm-tag-cogna-widget-clean.html` (expira em 24h) |
| Cogna IA auth | `POST /cogna-ai/user/auth/token` com `Basic Base64(email:senha)` |
| GTM | Conta Google do seller |
