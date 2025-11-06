# Azure Technical Translation 

Solução completa e escalável para tradução automática de artigos técnicos utilizando **Azure AI Translator**, com suporte a modelos personalizados, glossários técnicos, avaliação de qualidade, e integração com workflows automatizados.

## 🎯 Características Principais

- ✅ **Tradução Neural Avançada**: Baseada em transformers com suporte a 100+ idiomas
- ✅ **Modelos Personalizados**: Custom Translator para terminologia específica de domínio
- ✅ **Glossários Técnicos**: Suporte a glossários TSV/CSV para consistência terminológica
- ✅ **Tradução em Lote**: Processamento assíncrono de múltiplos documentos preservando formatação
- ✅ **Avaliação de Qualidade**: Cálculo automático de BLEU scores e métricas de qualidade
- ✅ **Integração Azure**: Blob Storage, Key Vault, e Managed Identity
- ✅ **Automação Completa**: Scripts para treinamento de modelos e processamento em lote
- ✅ **Conformidade**: LGPD, GDPR, e residência de dados em Brasil

## 📋 Requisitos

- Python 3.9 ou superior
- Conta Microsoft Azure ativa
- Recursos Azure configurados:
  - Azure AI Translator
  - Azure Blob Storage
  - Azure Key Vault (recomendado)
- Dependências Python (ver `config/requirements.txt`)

## 🚀 Início Rápido

### 1. Clonar Repositório

\`\`\`bash
git clone https://github.com/tilucasoliveira/azure-technical-translation/
\`\`\`

### 2. Configurar Ambiente

\`\`\`bash
# Copiar arquivo de configuração
cp config/.env.example .env

# Editar com suas credenciais Azure
nano .env

# Instalar dependências
pip install -r config/requirements.txt
\`\`\`

### 3. Executar Exemplo Simples

\`\`\`python
from src.translator import TechnicalDocumentTranslator

translator = TechnicalDocumentTranslator(
    translator_key='SUA_CHAVE',
    translator_endpoint='https://api.cognitive.microsofttranslator.com',
    translator_region='brazilsouth'
)

resultado = translator.translate_text(
    text='Neural networks enable deep learning',
    target_language='pt'
)

print(resultado['translated_text'])
\`\`\`

## 📚 Documentação

- **[SETUP.md](docs/SETUP.md)**: Guia detalhado de configuração
- **[USAGE.md](docs/USAGE.md)**: Como usar a solução
- **[BEST_PRACTICES.md](docs/BEST_PRACTICES.md)**: Boas práticas e otimizações
- **[ARCHITECTURE.md](docs/ARCHITECTURE.md)**: Arquitetura técnica e componentes
- **[API_REFERENCE.md](docs/API_REFERENCE.md)**: Referência completa de API

## 💡 Exemplos de Uso

### Tradução de Texto Simples

\`\`\`bash
python examples/simple_text_translation.py
\`\`\`

### Tradução em Lote de Documentos

\`\`\`bash
python examples/batch_document_translation.py
\`\`\`

### Treinamento de Modelo Personalizado

\`\`\`bash
python examples/custom_model_training.py
\`\`\`

### Integração com Glossários

\`\`\`bash
python examples/glossary_integration.py
\`\`\`

## 🔧 Scripts de Automação

### Setup de Recursos Azure

\`\`\`bash
./scripts/setup_azure_resources.sh
\`\`\`

### Treinar Modelo Customizado

\`\`\`bash
python scripts/train_custom_model.py --training-data data/training_data.tsv
\`\`\`

### Avaliar Qualidade de Tradução

\`\`\`bash
python scripts/evaluate_translation_quality.py --model-id seu-modelo-id
\`\`\`

### Traduzir Lote de Documentos

\`\`\`bash
python scripts/batch_translate_documents.py --source-container docs-pt --target-container docs-en
\`\`\`

## 📊 Estrutura do Projeto

\`\`\`
azure-technical-translation/
├── README.md                 # Este arquivo
├── LICENSE                   # Licença MIT
├── CONTRIBUTING.md           # Guia de contribuição
├── docs/                     # Documentação completa
├── src/                      # Código-fonte principal
├── examples/                 # Exemplos práticos
├── glossaries/               # Glossários técnicos
├── tests/                    # Testes unitários
├── config/                   # Arquivos de configuração
├── scripts/                  # Scripts de automação
└── data/                     # Dados de treinamento/teste
\`\`\`

## 🎓 Casos de Uso

- 📖 **Documentação Técnica de Software**: APIs, SDKs, manuais de produto
- 🔬 **Artigos Científicos**: Publicações acadêmicas, whitepapers
- 📋 **Manuais Técnicos**: Documentação regulatória, guias de operação
- 🏥 **Documentação Médica**: Com conformidade HIPAA/LGPD
- 🌐 **Conteúdo Multilíngue**: Integração com CMS e plataformas de publicação

## 💰 Custos e Otimização

| Serviço | Preço (USD) | Uso |
|---------|------------|-----|
| Tradução de Texto Padrão | $10/1M chars | Tradução em tempo real |
| Tradução Customizada | $40/1M chars | Modelos personalizados |
| Tradução de Documentos | $40/1M chars | Lotes em paralelo |
| Treinamento de Modelo | $10/1M chars | Treinamento inicial |
| Hospedagem de Modelo | $10/mês | Por modelo, por região |

Veja [BEST_PRACTICES.md](docs/BEST_PRACTICES.md) para estratégias de otimização de custos.

## ⚙️ Configuração Avançada

### Usar Modelo Personalizado

\`\`\`python
translator = TechnicalDocumentTranslator(
    ...,
    custom_model_id='seu-modelo-id'
)
\`\`\`

### Aplicar Glossário

\`\`\`python
translator.translate_document_batch(
    source_container='docs-origin',
    target_container='docs-translated',
    target_language='pt',
    glossary_url='https://storage.../glossario.tsv?sas'
)
\`\`\`

### Monitorar Qualidade

\`\`\`python
from src.quality_evaluator import QualityEvaluator

evaluator = QualityEvaluator(translator)
bleu_score = evaluator.calculate_bleu_score(
    reference_translations='referencias.txt',
    test_translations='teste.txt'
)
print(f"BLEU Score: {bleu_score:.2f}")
\`\`\`

## 🧪 Testes

\`\`\`bash
# Executar todos os testes
pytest tests/

# Testes com cobertura
pytest --cov=src tests/

# Teste específico
pytest tests/test_translator.py -v
\`\`\`

## 🤝 Contribuição

Contribuições são bem-vindas! Por favor:

1. Faça fork do projeto
2. Crie uma branch para sua feature (`git checkout -b feature/AmazingFeature`)
3. Commit suas mudanças (`git commit -m 'Add some AmazingFeature'`)
4. Push para a branch (`git push origin feature/AmazingFeature`)
5. Abra um Pull Request

Veja [CONTRIBUTING.md](CONTRIBUTING.md) para mais detalhes.

## 📝 Licença

Este projeto está licenciado sob a Licença MIT - veja o arquivo [LICENSE](LICENSE) para detalhes.


## 🔗 Recursos Adicionais

- [Documentação Azure AI Translator](https://learn.microsoft.com/en-us/azure/ai-services/translator/)
- [Azure SDK Python](https://github.com/Azure/azure-sdk-for-python)
- [Custom Translator Portal](https://portal.customtranslator.azure.ai/)
- [BLEU Score Explicado](https://en.wikipedia.org/wiki/BLEU)



---
