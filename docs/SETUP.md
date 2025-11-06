# Guia de Setup - Azure Technical Translation

## 1. Pré-requisitos

Antes de começar, você precisará de:

- **Python 3.9+**: Verifique com `python --version`
- **Pip**: Gerenciador de pacotes Python
- **Git**: Para clonar o repositório
- **Conta Microsoft Azure**: Necessária para usar os serviços
- **Azure CLI** (opcional): Para gerenciar recursos via linha de comando

## 2. Criar Recursos Azure

### Opção A: Via Azure Portal

1. **Criar Recurso Azure AI Translator**
   - Acesse: https://portal.azure.com
   - Clique em "Criar um recurso"
   - Busque "Translator"
   - Selecione "Translator (Text Translation)"
   - Preencha: Grupo de recursos, Nome, Região (Brazil South recomendado), Plano de preço
   - Clique "Revisar e criar"

2. **Criar Conta de Armazenamento (Blob Storage)**
   - Clique em "Criar um recurso"
   - Busque "Storage Account"
   - Preencha: Grupo de recursos, Nome da conta, Região, Desempenho, Redundância
   - Abra a conta criada
   - Vá em "Containers" e crie containers: `documents-source`, `documents-target`, `glossaries`

3. **Obter Credenciais**
   - Para Translator: Vá em "Chaves e Ponto de Extremidade"
   - Copie: Key 1 e Endpoint
   - Para Storage: Vá em "Chaves de acesso"
   - Copie: Connection string

### Opção B: Via Azure CLI

\`\`\`bash
# Fazer login
az login

# Definir variáveis
RESOURCE_GROUP="azure-translation-rg"
LOCATION="brazilsouth"
TRANSLATOR_NAME="translator-$(date +%s)"
STORAGE_NAME="storage$(date +%s%N | cut -c1-8)"

# Criar grupo de recursos
az group create --name $RESOURCE_GROUP --location $LOCATION

# Criar Translator
az cognitiveservices account create \
  --name $TRANSLATOR_NAME \
  --resource-group $RESOURCE_GROUP \
  --kind TextTranslation \
  --sku S1 \
  --location $LOCATION

# Criar Storage Account
az storage account create \
  --name $STORAGE_NAME \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION

# Criar containers
az storage container create \
  --account-name $STORAGE_NAME \
  --name documents-source

az storage container create \
  --account-name $STORAGE_NAME \
  --name documents-target

az storage container create \
  --account-name $STORAGE_NAME \
  --name glossaries
\`\`\`

## 3. Configurar Ambiente Local

### Passo 1: Clonar Repositório

\`\`\`bash
git clone https://github.com/seu-usuario/azure-technical-translation.git
cd azure-technical-translation
\`\`\`

### Passo 2: Criar Ambiente Virtual

\`\`\`bash
# Windows
python -m venv venv
venv\Scripts\activate

# macOS/Linux
python3 -m venv venv
source venv/bin/activate
\`\`\`

### Passo 3: Instalar Dependências

\`\`\`bash
pip install -r config/requirements.txt
\`\`\`

### Passo 4: Configurar Variáveis de Ambiente

\`\`\`bash
# Copiar arquivo de exemplo
cp config/.env.example .env

# Editar com suas credenciais
# Windows: notepad .env
# macOS/Linux: nano .env
\`\`\`

**Preencher valores:**

\`\`\`
TRANSLATOR_KEY=sua-chave-translator
TRANSLATOR_ENDPOINT=https://api.cognitive.microsofttranslator.com
TRANSLATOR_REGION=brazilsouth
STORAGE_CONNECTION_STRING=sua-connection-string
STORAGE_ACCOUNT_NAME=seu-nome-storage
\`\`\`

## 4. Verificar Configuração

\`\`\`bash
python -c "
import os
from dotenv import load_dotenv

load_dotenv()
print('✓ TRANSLATOR_KEY:', 'Configurado' if os.getenv('TRANSLATOR_KEY') else '❌ Falta')
print('✓ TRANSLATOR_ENDPOINT:', 'Configurado' if os.getenv('TRANSLATOR_ENDPOINT') else '❌ Falta')
print('✓ STORAGE_CONNECTION_STRING:', 'Configurado' if os.getenv('STORAGE_CONNECTION_STRING') else '❌ Falta')
"
\`\`\`

## 5. Testar Conexão com Azure

\`\`\`bash
python examples/simple_text_translation.py
\`\`\`

Se funcionar, você deve ver uma tradução de exemplo do inglês para português.

## 6. Próximos Passos

Após verificar a configuração:

1. Revisar [USAGE.md](USAGE.md) para aprender como usar a API
2. Consultar exemplos em `examples/`
3. Ler [BEST_PRACTICES.md](BEST_PRACTICES.md) para otimizar traduções
4. Executar testes: `pytest tests/`

## Troubleshooting

### Erro: "AuthenticationError: Invalid credentials"
- Verifique se TRANSLATOR_KEY está correto
- Confirme se está usando a região correta
- Verifique se a chave não expirou

### Erro: "ContainerNotFound"
- Confirme se containers existem em Storage Account
- Verifique permissões de Storage Blob Data Contributor

### Erro: "Timeout"
- Aumentar TIMEOUT_SECONDS em .env
- Dividir documentos grandes em arquivos menores
- Verificar latência de rede

### Erro: ImportError para azure-ai-translation-document
- Reinstalar dependências: `pip install --upgrade -r config/requirements.txt`
- Verificar versão do Python (precisa ser 3.9+)

## Suporte e Documentação

- Azure Translator: https://learn.microsoft.com/en-us/azure/ai-services/translator/
- Azure Storage: https://learn.microsoft.com/en-us/azure/storage/
- Python SDK: https://github.com/Azure/azure-sdk-for-python

