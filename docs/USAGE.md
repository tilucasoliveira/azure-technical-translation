# Guia de Uso - Azure Technical Translation

## Uso Básico

### Importar e Inicializar

\`\`\`python
from src.translator import TechnicalDocumentTranslator
import os
from dotenv import load_dotenv

load_dotenv()

translator = TechnicalDocumentTranslator(
    translator_key=os.getenv('TRANSLATOR_KEY'),
    translator_endpoint=os.getenv('TRANSLATOR_ENDPOINT'),
    translator_region=os.getenv('TRANSLATOR_REGION'),
    storage_connection_string=os.getenv('STORAGE_CONNECTION_STRING'),
    custom_model_id=os.getenv('CUSTOM_MODEL_ID')  # opcional
)
\`\`\`

## Traduções de Texto

### Tradução Simples

\`\`\`python
resultado = translator.translate_text(
    text="The neural network architecture enables deep learning",
    target_language='pt',
    source_language='en',
    use_custom_model=False  # usar modelo base
)

print(f"Original: {texto}")
print(f"Tradução: {resultado['translated_text']}")
print(f"Idioma detectado: {resultado['detected_language']}")
\`\`\`

### Tradução com Detecção Automática

\`\`\`python
resultado = translator.translate_text(
    text="Machine learning is a subset of artificial intelligence",
    target_language='pt',
    source_language='auto'  # detecção automática
)
\`\`\`

### Tradução com Modelo Personalizado

\`\`\`python
resultado = translator.translate_text(
    text="GPU acceleration optimizes model training",
    target_language='pt',
    source_language='en',
    use_custom_model=True  # usar modelo custom
)
\`\`\`

## Tradução em Lote de Documentos

### Preparar Documentos

1. Colocar documentos em container de origem do Azure Storage
2. Documentos suportados: PDF, DOCX, XLSX, PPTX, TXT, HTML

### Executar Tradução em Lote

\`\`\`python
operacao = translator.translate_document_batch(
    source_container='documents-source',
    target_container='documents-target',
    target_language='pt',
    source_language='en',
    glossary_url=None  # opcional
)

print(f"Operação iniciada: {operacao['operation_location']}")
\`\`\`

### Monitorar Progresso

\`\`\`python
import time

status = translator.check_translation_status(
    operacao['operation_location']
)

# Status pode ser: Running, Succeeded, Failed, ValidationFailed
print(f"Status: {status['status']}")
print(f"Resumo: {status['summary']}")

# Aguardar conclusão
while status['status'] == 'Running':
    time.sleep(5)
    status = translator.check_translation_status(
        operacao['operation_location']
    )
    print(f"Progresso: {status['summary']}")

if status['status'] == 'Succeeded':
    print("✓ Tradução concluída com sucesso!")
elif status['status'] == 'Failed':
    print("✗ Falha na tradução")
\`\`\`

## Gerenciamento de Glossários

### Upload de Glossário

\`\`\`python
glossary_url = translator.upload_glossary(
    glossary_file_path='glossaries/glossario_tecnico_ia.tsv',
    container_name='glossaries'
)

print(f"Glossário carregado: {glossary_url}")
\`\`\`

### Usar Glossário em Tradução

\`\`\`python
# Tradução de texto com glossário
resultado = translator.translate_text(
    text="Deep learning uses neural networks",
    target_language='pt',
    source_language='en',
    glossary_url=glossary_url  # não suportado em Text API
)

# Tradução de documento com glossário
operacao = translator.translate_document_batch(
    source_container='documents-source',
    target_container='documents-target',
    target_language='pt',
    source_language='en',
    glossary_url=glossary_url
)
\`\`\`

### Criar Glossário Customizado

Formato TSV (Source\tTarget):

\`\`\`
Machine Learning\tAprendizado de Máquina
Deep Learning\tAprendizado Profundo
Neural Network\tRede Neural
\`\`\`

Salve como `glossario_customizado.tsv` e faça upload.

## Avaliação de Qualidade

### Calcular BLEU Score

\`\`\`python
from src.quality_evaluator import QualityEvaluator

evaluator = QualityEvaluator(translator)

bleu_score = evaluator.calculate_bleu_score(
    reference_file='data/traducoes_referencia.txt',
    test_file='data/traducoes_teste.txt'
)

print(f"BLEU Score: {bleu_score:.2f}")

# Interpretação
if bleu_score >= 50:
    print("✓ Qualidade excelente")
elif bleu_score >= 40:
    print("✓ Qualidade boa")
elif bleu_score >= 30:
    print("⚠ Qualidade aceitável com revisão")
else:
    print("✗ Qualidade insuficiente")
\`\`\`

## Casos de Uso Avançados

### Tradução Segmentada de Documento Grande

\`\`\`python
# Dividir documento grande em seções
with open('documento_grande.txt', 'r', encoding='utf-8') as f:
    conteudo = f.read()

# Dividir por parágrafos
paragrafos = conteudo.split('\n\n')

# Traduzir cada parágrafo
traducoes = []
for paragrafo in paragrafos:
    resultado = translator.translate_text(
        text=paragrafo,
        target_language='pt',
        source_language='en'
    )
    traducoes.append(resultado['translated_text'])

# Salvar documento traduzido
with open('documento_grande_traduzido.txt', 'w', encoding='utf-8') as f:
    f.write('\n\n'.join(traducoes))
\`\`\`

### Tradução com Fallback para Modelo Base

\`\`\`python
try:
    # Tentar com modelo customizado
    resultado = translator.translate_text(
        text=texto,
        target_language='pt',
        use_custom_model=True
    )
except Exception as e:
    print(f"Erro com modelo custom: {e}")
    # Fallback para modelo base
    resultado = translator.translate_text(
        text=texto,
        target_language='pt',
        use_custom_model=False
    )

print(resultado['translated_text'])
\`\`\`

### Processamento em Paralelo

\`\`\`python
from concurrent.futures import ThreadPoolExecutor, as_completed

textos = [
    "Text 1",
    "Text 2",
    "Text 3"
]

def traduzir(texto):
    return translator.translate_text(
        text=texto,
        target_language='pt',
        source_language='en'
    )

with ThreadPoolExecutor(max_workers=4) as executor:
    futures = [executor.submit(traduzir, t) for t in textos]

    resultados = []
    for future in as_completed(futures):
        resultados.append(future.result())

for res in resultados:
    print(res['translated_text'])
\`\`\`

## Tratamento de Erros

\`\`\`python
import requests
from azure.core.exceptions import AzureError

try:
    resultado = translator.translate_text(
        text="Neural networks",
        target_language='pt'
    )
except requests.exceptions.Timeout:
    print("Timeout na requisição")
except requests.exceptions.ConnectionError:
    print("Erro de conexão com Azure")
except AzureError as e:
    print(f"Erro Azure: {e}")
except Exception as e:
    print(f"Erro inesperado: {e}")
\`\`\`

## Logging e Debug

\`\`\`python
import logging

# Habilitar debug logging
logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger('azure')

# Usar logger em seu código
logger.info(f"Traduzindo: {texto}")
\`\`\`

## Performance e Otimizações

### Caching de Traduções

\`\`\`python
cache_traducoes = {}

def traduzir_com_cache(texto, idioma):
    chave = f"{texto}_{idioma}"

    if chave in cache_traducoes:
        return cache_traducoes[chave]

    resultado = translator.translate_text(
        text=texto,
        target_language=idioma
    )

    cache_traducoes[chave] = resultado['translated_text']
    return resultado['translated_text']
\`\`\`

### Batch Processing

\`\`\`python
from tqdm import tqdm

textos = [...]

for i in range(0, len(textos), 10):
    lote = textos[i:i+10]

    for texto in tqdm(lote, desc=f"Lote {i//10}"):
        resultado = translator.translate_text(
            text=texto,
            target_language='pt'
        )
\`\`\`

Mais exemplos disponíveis em `examples/`
