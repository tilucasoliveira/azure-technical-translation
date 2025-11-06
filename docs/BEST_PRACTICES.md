# Melhores Práticas - Azure Technical Translation

## 1. Otimização de Custos

### Use Translation Memory
- Evite retradução de conteúdo já traduzido
- Mantenha histórico de traduções em banco de dados
- Reduza custos em 40-60% para conteúdo repetitivo

### Escolha o Tipo de Tradução Apropriado
| Tipo | Custo | Quando usar |
|------|-------|-----------|
| Text API | $ | Tempo real, textos curtos |
| Document Batch | $$ | Documentos não urgentes |
| Custom Model | $$$$ | Terminologia especializada crítica |

### Implemente Caching
\`\`\`python
import redis

cache = redis.Redis()

def traduzir(texto, idioma):
    chave = f"traducao:{texto}:{idioma}"
    resultado = cache.get(chave)

    if resultado:
        return resultado.decode()

    resultado = translator.translate_text(texto, idioma)
    cache.setex(chave, 86400, resultado['translated_text'])

    return resultado['translated_text']
\`\`\`

## 2. Garantia de Qualidade

### Estabeleça BLEU Score Mínimo
- Produções: BLEU ≥ 45
- Staging: BLEU ≥ 35
- Desenvolvimento: BLEU ≥ 25

### Implemente Validação Humana

\`\`\`python
def validar_traducao(original, traducao, especialista_id):
    # Armazenar para revisão humana
    db.inserir_validacao({
        'original': original,
        'traducao': traducao,
        'especialista_id': especialista_id,
        'status': 'pendente',
        'timestamp': datetime.now()
    })

    # Notificar especialista
    enviar_email_validacao(especialista_id)
\`\`\`

### Teste Antes de Deploy

\`\`\`bash
pytest tests/ --cov=src

# Testar com dados reais
python scripts/evaluate_translation_quality.py \
    --model-id seu-modelo \
    --test-data data/test_real.tsv
\`\`\```

## 3. Segurança e Conformidade

### Proteção de Dados Sensíveis
- Use Azure Key Vault para credenciais
- Ative logging de auditoria
- Implemente acesso baseado em roles (RBAC)

\`\`\`python
from azure.keyvault.secrets import SecretClient
from azure.identity import DefaultAzureCredential

credential = DefaultAzureCredential()
client = SecretClient(
    vault_url="https://seu-keyvault.vault.azure.net",
    credential=credential
)

translator_key = client.get_secret("translator-key").value
\`\`\`

### Conformidade LGPD/GDPR
- Dados processados em Brazil South para LGPD
- Implementar direito ao esquecimento
- Documentar processamento de dados

\`\`\`python
def deletar_dados_usuario(usuario_id):
    # Remover do storage
    container.delete_blobs_if_exists(f"user_{usuario_id}/*")

    # Remover de cache
    cache.delete(f"user_translations:{usuario_id}")

    # Log para auditoria
    log_auditoria(f"Deletado dados de {usuario_id}")
\`\`\`

## 4. Escalabilidade

### Use Managed Identity em Produção
Evite armazenar credenciais no código

\`\`\`python
from azure.identity import DefaultAzureCredential

credential = DefaultAzureCredential()
translator = TechnicalDocumentTranslator(
    credential=credential,
    # ... outros parâmetros
)
\`\`\`

### Implemente Circuit Breaker
Evite cascata de falhas

\`\`\`python
from pybreaker import CircuitBreaker

cb = CircuitBreaker(
    fail_max=5,
    reset_timeout=60,
    listeners=[
        lambda cb, e: logger.error(f"Circuit aberto: {e}")
    ]
)

@cb
def traduzir_com_circuit_breaker(texto):
    return translator.translate_text(text=texto, target_language='pt')
\`\`\`

### Monitoramento e Alertas
\`\`\`python
import prometheus_client

metrica_traducoes = prometheus_client.Counter(
    'traducoes_total',
    'Total de traduções',
    ['idioma_destino', 'status']
)

metrica_latencia = prometheus_client.Histogram(
    'latencia_traducao_segundos',
    'Latência de tradução em segundos'
)

@metrica_latencia.time()
def traduzir(texto):
    resultado = translator.translate_text(
        text=texto,
        target_language='pt'
    )
    metrica_traducoes.labels(
        idioma_destino='pt',
        status='sucesso'
    ).inc()
    return resultado
\`\`\`

## 5. Desenvolvimento de Modelos Personalizados

### Preparação de Dados
- ✓ Mínimo 10.000 pares paralelos
- ✓ 80% treinamento, 10% tuning, 10% teste
- ✓ Representativo do conteúdo real
- ✓ Sem duplicação entre conjuntos

\`\`\`python
def validar_dados_treinamento(arquivo_treino, arquivo_tuning, arquivo_teste):
    dados_treino = set(open(arquivo_treino).readlines())
    dados_tuning = set(open(arquivo_tuning).readlines())
    dados_teste = set(open(arquivo_teste).readlines())

    # Verificar sobreposição
    if dados_treino & dados_tuning:
        raise ValueError("Sobreposição entre treino e tuning!")

    if dados_treino & dados_teste:
        raise ValueError("Sobreposição entre treino e teste!")

    print(f"✓ Validação OK: {len(dados_treino)} treino, "
          f"{len(dados_tuning)} tuning, {len(dados_teste)} teste")
\`\`\`

### Iteração e Melhoria
1. Treinar modelo com dados iniciais
2. Avaliar BLEU score
3. Analisar erros comuns
4. Coletar mais dados nessas áreas
5. Retreinar com dados adicionados
6. Repetir até qualidade aceitável

\`\`\`bash
#!/bin/bash

for i in {1..5}; do
    echo "Iteração $i"
    python scripts/train_custom_model.py \
        --iteration $i \
        --training-data data/train_v${i}.tsv

    python scripts/evaluate_translation_quality.py \
        --model-id modelo-v${i}

    # Se BLEU >= 45, parar
done
\`\`\`

## 6. Integração Contínua/Contínua

### GitHub Actions Workflow

\`\`\`.github/workflows/translate-ci.yml
name: Translation CI

on: [push, pull_request]

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Setup Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.9'

      - name: Install Dependencies
        run: pip install -r config/requirements.txt

      - name: Run Tests
        run: pytest tests/ --cov=src

      - name: Code Quality
        run: |
          flake8 src/ tests/
          mypy src/

      - name: Evaluate Translation Quality
        env:
          TRANSLATOR_KEY: \${{ secrets.TRANSLATOR_KEY }}
        run: python scripts/evaluate_translation_quality.py
\`\`\`

## 7. Troubleshooting Comum

### Baixo BLEU Score
1. ✓ Aumentar volume de dados de treinamento
2. ✓ Melhorar qualidade/consistência dos dados
3. ✓ Revisar alinhamento de pares de sentenças
4. ✓ Considerar subdomínios separados

### Glossário Não Sendo Aplicado
1. ✓ Verificar case sensitivity
2. ✓ Validar formato TSV
3. ✓ Confirmar que termos aparecem no texto
4. ✓ Usar Neural Dictionary em vez de Phrase

### Timeout em Documentos Grandes
1. ✓ Dividir em arquivos menores (< 50MB)
2. ✓ Usar batch processing assíncrono
3. ✓ Aumentar timeout do cliente
4. ✓ Implementar processamento paralelo

## 8. Recursos Adicionais

- [Microsoft Learn - Azure Translator](https://learn.microsoft.com/en-us/azure/ai-services/translator/)
- [Documentação Python SDK](https://github.com/Azure/azure-sdk-for-python)
- [BLEU Score Explained](https://en.wikipedia.org/wiki/BLEU)
- [Boas Práticas de Machine Translation](https://machinetranslate.org/)

