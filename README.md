# n8n Chatwoot MAIA Bot

Assistente virtual multilíngue inteligente para Chatwoot, com detecção automática de idioma, classificação de intenções e integração com dashboards.

## 📋 Descrição

MAIA (Multidrop AI Assistant) é um bot avançado para Chatwoot que oferece suporte automatizado em 3 idiomas (Português, Inglês e Alemão), com capacidade de detectar automaticamente o idioma do usuário, classificar suas intenções e fornecer respostas contextualizadas usando ferramentas específicas.

## ✨ Funcionalidades

### 🌍 Detecção Automática de Idioma
- Suporte para **Português (PT)**, **Inglês (EN)** e **Alemão (DE)**
- Algoritmo avançado de detecção baseado em padrões linguísticos
- Análise de palavras-chave, estruturas gramaticais e caracteres especiais
- Cache de idioma por conversação para consistência

### 🎯 Classificação Inteligente de Intenções
- **Saudações**: Respostas rápidas e personalizadas
- **Produtor**: Consultas sobre vendas, faturamento e produtos
- **Afiliado**: Informações sobre comissões e performance
- **Saque**: Valores disponíveis para retirada
- **Suporte**: Tutoriais e documentação via RAG (Gemini)

### 🛡️ Sistema Anti-Loop
- **Cache de mensagens**: Previne processamento duplicado
- **Filtros críticos**: Bloqueia mensagens do próprio bot
- **Validação de tipo**: Processa apenas mensagens incoming
- **Limpeza automática**: Cache com TTL de 10 minutos

### 🔧 Integrações
- **Chatwoot Webhook**: Recebe eventos em tempo real
- **OpenAI GPT-4.1-mini**: Geração de respostas contextualizadas
- **Gemini RAG Search**: Base de conhecimento para suporte
- **Dashboard APIs**: Dados de performance do usuário
- **Simple Memory**: Contexto de conversação

## 🚀 Requisitos

- n8n instalado (self-hosted ou cloud)
- Conta Chatwoot com acesso a webhooks
- API Key da OpenAI (GPT-4)
- API Key do Google Gemini (para RAG)
- Backend da Multidrop (para dashboards)

## 📦 Instalação

1. **Importar o Workflow**
   ```bash
   # No n8n, vá em Workflows > Import from File
   # Selecione o arquivo workflow.json
   ```

2. **Configurar Credenciais**
   
   Configure as seguintes credenciais no n8n:
   
   - **OpenAI API**: Para o modelo de chat
   - **Chatwoot API** (HTTP Header Auth): Token de API do Chatwoot
   - Nenhuma credencial necessária para Gemini (usa API key na URL)

3. **Configurar Webhook no Chatwoot**
   
   - Acesse Chatwoot > Settings > Integrations > Webhooks
   - Adicione novo webhook apontando para a URL do n8n
   - Selecione evento: `message_created`
   - Copie a URL do webhook do n8n (nó "Webhook Chatwoot")

4. **Atualizar Endpoints**
   
   No workflow, atualize:
   - URL do backend Multidrop (nós de Dashboard)
   - API Key do Gemini (nó "Gemini RAG Search")
   - Account ID e Inbox ID do Chatwoot

## 🎯 Como Usar

1. **Ative o Workflow** no n8n
2. **Configure o webhook** no Chatwoot
3. Quando um usuário enviar mensagem:
   - Sistema detecta idioma automaticamente
   - Classifica a intenção da mensagem
   - Chama ferramentas apropriadas (Dashboard, RAG, etc.)
   - Gera resposta contextualizada no idioma correto
   - Envia resposta de volta ao Chatwoot

## 📊 Arquitetura do Bot

```
Webhook Chatwoot
    ↓
Cache Anti-Duplicação
    ↓
Filtro Crítico (valida tipo de mensagem)
    ↓
Extrair Dados (idioma + intenção)
    ↓
Validar Dados Extraídos
    ↓
HTTP Request (autenticação backend)
    ↓
AI Agent (com tools: Dashboard Produtor, Dashboard Afiliado, Gemini RAG)
    ↓
Formatar Resposta
    ↓
HTTP Request (envia para Chatwoot)
    ↓
Responder ao Webhook
```

## 🔍 Detecção de Idioma

### Padrões Analisados

**Português:**
- Palavras: o, a, os, as, que, como, quando, etc.
- Estruturas: -ção, -mento, gerúndio (-ndo)
- Acentos: á, é, ê, ã, õ, ç

**Inglês:**
- Palavras: the, be, to, of, and, have, etc.
- Estruturas: -ing, -tion, contrações (I'm, you're)

**Alemão:**
- Palavras: der, die, das, und, oder, etc.
- Estruturas: -ung, -heit, -keit, ge-
- Caracteres: ä, ö, ü, ß

### Sistema de Pontuação

Cada padrão tem um peso específico:
- Palavras comuns: 2 pontos
- Estruturas gramaticais: 3 pontos
- Acentos/caracteres especiais: 1-4 pontos

## 🎯 Classificação de Intenções

### Categorias

| Intenção | Palavras-chave | Ação |
|----------|----------------|------|
| **greeting** | oi, olá, hello, hallo | Resposta rápida |
| **producer** | vendas, faturamento, produtos | Dashboard Produtor |
| **affiliate** | comissão, cliques, leads | Dashboard Afiliado |
| **withdrawal** | saque, disponível, liberado | Dashboard Produtor |
| **support** | como fazer, tutorial, ajuda | Gemini RAG |
| **casual** | Outras mensagens | AI Agent genérico |

## ⚙️ Configuração Avançada

### Personalizar Respostas

Edite o System Message do nó "AI Agent" para ajustar:
- Tom de voz da MAIA
- Formato das respostas
- Regras de idioma
- Comportamento por intenção

### Adicionar Novos Idiomas

1. No nó "Extrair Dados", adicione padrões no `languageDetector`
2. Atualize `intentClassifier` com palavras-chave do novo idioma
3. Adicione traduções no System Message do AI Agent

### Adicionar Novas Ferramentas

1. Crie novo nó HTTP Request Tool
2. Configure descrição clara de quando chamar
3. Conecte ao AI Agent
4. Atualize System Message com instruções

## 🔒 Segurança

⚠️ **IMPORTANTE**:

- **Nunca** exponha API keys no código
- Use credenciais do n8n para tokens
- Configure `content_attributes.skip_webhook: true` nas respostas
- Implemente rate limiting no Chatwoot
- Monitore logs para detectar loops

### Flags de Segurança no Workflow

```javascript
// Ao enviar mensagem para Chatwoot
{
  content_attributes: {
    source: "n8n_bot",      // Identifica origem
    skip_webhook: true       // Previne loop
  }
}
```

## 🐛 Troubleshooting

### Bot está respondendo a si mesmo
- Verifique se `skip_webhook: true` está configurado
- Confirme que filtros estão ativos
- Limpe o cache global se necessário

### Idioma detectado incorretamente
- Mensagens muito curtas podem ter detecção imprecisa
- Verifique se padrões do idioma estão atualizados
- Use cache de idioma para consistência

### Tools não são chamadas
- Verifique descrição das tools (deve ser clara)
- Confirme que palavras-chave estão corretas
- Teste intenção manualmente no console

## 📝 Logs e Debug

O workflow inclui logs detalhados em cada etapa:

```javascript
console.log('🔍 DETECÇÃO DE IDIOMA');
console.log('🎯 CLASSIFICAÇÃO DE INTENÇÃO');
console.log('🛡️ FILTRO CRÍTICO');
console.log('📤 EXTRAÇÃO DE DADOS');
```

Monitore execuções no n8n para debug.

## 🤝 Contribuindo

Contribuições são bem-vindas! Áreas para melhoria:
- Adicionar mais idiomas
- Melhorar detecção de intenções
- Criar mais ferramentas/integrações
- Otimizar performance

## 📄 Licença

MIT License - veja o arquivo LICENSE para detalhes.

## 🆘 Suporte

- [Documentação n8n](https://docs.n8n.io/)
- [Documentação Chatwoot](https://www.chatwoot.com/docs/)
- [OpenAI API Docs](https://platform.openai.com/docs/)

---

**Desenvolvido com ❤️ para Multidrop**
