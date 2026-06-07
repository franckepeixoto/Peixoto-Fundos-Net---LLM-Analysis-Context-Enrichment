# Peixoto Fundos Net - LLM Analysis Context Enrichment Platform

<div align="center">

**Uma plataforma inteligente de enriquecimento de contexto para análise de fundos imobiliários por LLMs**

[Overview](#visão-geral) • [Arquitetura](#arquitetura) • [Como Usar](#como-usar) • [Autores](#autores)

</div>

---

## 📋 Visão Geral

**Peixoto Fundos Net** é uma plataforma de **Context-Aware Prompting** que agrega dados estruturados de fundos imobiliários (FIIs) de múltiplas fontes e os apresenta em formato otimizado para análise por Large Language Models (LLMs).

### O Problema

Analistas precisam de dados consolidados de fundos imobiliários para análise fundamentalista, mas essas informações estão espalhadas em diferentes fontes:
- **FNET (BMF Bovespa)**: Proventos, informes mensais
- **ADVFN**: Histórico de cotações
- **BCB (Banco Central)**: Indicadores macroeconômicos (SELIC, IPCA)

### A Solução

Peixoto Fundos Net coleta, valida, formata e estrutura essas informações em um **prompt contextualizado** que um LLM pode usar para:
- ✅ Análise de rentabilidade
- ✅ Avaliação de risco
- ✅ Comparação de desempenho
- ✅ Previsão de tendências
- ✅ Relatórios automatizados

### Tipo de Solução: Context Enrichment via Prompting

Este projeto implementa **LLM Context Enrichment** (Enriquecimento de Contexto para LLM), combinando:

- **Structured Data Aggregation**: Coleta dados de múltiplas APIs
- **Data Normalization**: Formata em padrões consistentes
- **Context Composition**: Estrutura em formato de prompt
- **Semantic Formatting**: Mantém estrutura semanticamente clara para LLMs

Isso difere de:
- **RAG (Retrieval Augmented Generation)**: Busca documentos similares em uma base de conhecimento
- **MCP (Model Context Protocol)**: Protocolo de comunicação estruturada
- **Fine-tuning**: Treino de modelos específicos

Aqui, o contexto é **estruturado dinamicamente** para cada fundo, otimizado para prompt engineering.

---

## 🏗️ Arquitetura

### Visão em Camadas

```
┌─────────────────────────────────────────────────────────┐
│  Azure Functions / HTTP API                             │
│  POST /api/Obter_Dados_Por_Ticket_e_CNPJ               │
└─────────────────────┬───────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────┐
│  Handlers (Validação & Orquestração)                    │
│  └─ Recebe CNPJ e ticker                               │
│  └─ Valida entrada                                      │
│  └─ Trata erros                                         │
└─────────────────────┬───────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────┐
│  Services (Lógica de Negócio)                           │
│  ├─ FundDataService (Orquestrador)                      │
│  │  ├─ EconomicIndicatorsService (SELIC, IPCA)         │
│  │  ├─ FnetService (Proventos, Informes)               │
│  │  └─ AdvfnService (Cotações)                         │
│  └─ Todos reutilizam HttpClient                        │
└─────────────────────┬───────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────┐
│  Utilities (Suporte)                                    │
│  ├─ Validators: CNPJ, ticker                           │
│  ├─ Formatters: Moeda, percentual                      │
│  ├─ HttpClient: Retry automático                       │
│  └─ XmlParser: Parsing seguro                          │
└─────────────────────┬───────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────┐
│  External Data Sources                                  │
│  ├─ FNET: Proventos e Informes                         │
│  ├─ ADVFN: Histórico de Cotações                       │
│  └─ BCB: Indicadores Econômicos                        │
└─────────────────────────────────────────────────────────┘
```

### Estrutura de Diretórios

```
peixoto-fundos-net-logicapp/
├── src/
│   ├── config.py                 # Configurações centralizadas
│   ├── exceptions.py             # Exceções customizadas
│   ├── logger.py                 # Setup de logging
│   │
│   ├── models/
│   │   └── fundo.py              # Dataclasses de dados
│   │
│   ├── services/
│   │   ├── http_client.py        # Cliente HTTP com retry
│   │   ├── xml_parser.py         # Parser XML seguro
│   │   ├── fnet_service.py       # Serviço FNET (proventos)
│   │   ├── advfn_service.py      # Serviço ADVFN (cotações)
│   │   ├── economic_indicators_service.py  # Serviço BCB
│   │   └── fund_data_service.py  # Orquestrador principal
│   │
│   ├── utils/
│   │   ├── validators.py         # Validação CNPJ, ticker
│   │   └── formatters.py         # Formatação moeda, percentual
│   │
│   └── handlers/
│       └── fund_handlers.py      # Azure Functions handlers
│
├── tests/
│   ├── test_validators.py        # Testes de validação
│   └── test_formatters.py        # Testes de formatação
│
├── function_app.py               # Entry point Azure Functions
├── function_app_refatorado.py    # Alternativa refatorada
├── requirements.txt              # Dependências Python
├── host.json                     # Config Azure Functions
├── local.settings.json           # Configurações locais
├── .env.example                  # Template de variáveis
└── README.md                     # Este arquivo
```

---

## 🚀 Como Usar

### Requisitos

- Python 3.9+
- Azure Functions (ou local com `func` CLI)
- Dependências: `pip install -r requirements.txt`

### Setup Rápido

```bash
# 1. Clone o repositório
git clone <repo-url>
cd peixoto-fundos-net-logicapp

# 2. Configure ambiente Python
python -m venv .venv
.venv\Scripts\activate  # Windows
source .venv/bin/activate  # Linux/Mac

# 3. Instale dependências
pip install -r requirements.txt

# 4. Configure variáveis de ambiente
cp .env.example .env
# Edite .env conforme necessário

# 5. Execute testes (opcional)
pytest tests/ -v
```

### Uso via API HTTP

#### Requisição

```bash
# Via Query String
curl "http://localhost:7071/api/Obter_Dados_Por_Ticket_e_CNPJ?cnpj=17311079000174&ticker=SAIC11"

# Via JSON Body
curl -X POST http://localhost:7071/api/Obter_Dados_Por_Ticket_e_CNPJ \
  -H "Content-Type: application/json" \
  -d '{"cnpj": "17311079000174", "ticker": "SAIC11"}'
```

#### Resposta

```markdown
#Indicadores:
* Hoje é dia 07/06/2026
* IPCA acumulado de 12 meses: 4,50%
* SELIC: 10,50% a.a.
-----------

### SAIC11 - Informações sobre Pagamento de Proventos:

```
Rendimento;Amortização;Data do pagamento;Data-base
R$ 0,85;R$ 0,00;15/06/2026;08/06/2026
...
```

### SAIC11 - RESUMO HISTÓRICO DO VALOR DE COTAÇÃO: SAIC11 (Valores negociados em bolsa)

## Cotação média foi de (Formato CSV):

```
Período;Variação;Abertura;Máximo;Mínimo;Volume Médio;VWAP Diário
1 Semana;2,34%;R$ 98,50;R$ 100,20;R$ 97,80;1.234.567;99,12
...
```
```

### Uso Programático

```python
from src.services.fund_data_service import FundDataService

# Criar serviço
service = FundDataService()

try:
    # Obter dados estruturados
    dados = service.obter_dados_completos(
        cnpj="17311079000174",
        ticker="SAIC11"
    )
    
    print(dados)
    
finally:
    service.close()
```

### Integração com LLM

```python
from src.services.fund_data_service import FundDataService
import anthropic

# 1. Obter contexto estruturado
service = FundDataService()
contexto = service.obter_dados_completos("17311079000174", "SAIC11")
service.close()

# 2. Usar como prompt para LLM
client = anthropic.Anthropic()

response = client.messages.create(
    model="claude-3-5-sonnet-20241022",
    max_tokens=1024,
    messages=[
        {
            "role": "user",
            "content": f"""
Você é um analista de investimentos especializado em fundos imobiliários.
Analise o fundo imobiliário abaixo e forneça sua opinião sobre viabilidade de investimento:

{contexto}

Considere:
- Rentabilidade histórica
- Pagamento de proventos
- Condições macroeconômicas
- Evolução das cotações
"""
        }
    ]
)

print(response.content[0].text)
```

---

## 🔧 Componentes Principais

### FundDataService (Orquestrador)

Agrupa todos os serviços para recuperar dados completos de um fundo.

```python
service = FundDataService()

# Retorna string formatada
dados_texto = service.obter_dados_completos(cnpj, ticker)

# Retorna objeto estruturado
dados_estruturados = service.obter_dados_estruturados(cnpj, ticker)

service.close()
```

### FnetService

Coleta informações do FNET (BMF Bovespa):
- Proventos (rendimentos, amortizações)
- Informes mensais (patrimônio, rentabilidade, dividend yield)

### AdvfnService

Coleta histórico de cotações de um fundo:
- Preços de abertura, máxima, mínima
- Volume negociado
- VWAP (Volume Weighted Average Price)

### EconomicIndicatorsService

Coleta indicadores econômicos do Banco Central:
- Taxa SELIC (taxa de juros básica)
- IPCA (inflação acumulada)

### HttpClient

Cliente HTTP robusto com:
- Retry automático (3 tentativas)
- Timeout configurável
- Logging estruturado
- Tratamento de erro profissional

---

## 🧪 Testes

```bash
# Executar todos os testes
pytest tests/ -v

# Testes específicos
pytest tests/test_validators.py -v
pytest tests/test_formatters.py -v

# Com cobertura
pytest tests/ --cov=src --cov-report=html
```

---

## 📊 Exemplos de Uso

### Validação de Entrada

```python
from src.utils.validators import validar_cnpj, validar_ticker, formatar_cnpj

# Validar e formatar CNPJ
cnpj = validar_cnpj("17.311.079/0001-74")  # "17311079000174"
cnpj_fmt = formatar_cnpj(cnpj)              # "17.311.079/0001-74"

# Validar ticker
ticker = validar_ticker("saic11")           # "SAIC11"
```

### Formatação de Dados

```python
from src.utils.formatters import formatar_moeda, formatar_percentual

# Moeda brasileira
print(formatar_moeda(1234.56))      # R$ 1.234,56
print(formatar_moeda(1234.56, com_cifrão=False))  # 1.234,56

# Percentual
print(formatar_percentual(0.1234))  # 12,34%
print(formatar_percentual(0.1234, casas_decimais=3))  # 12,340%
```

### Logging

```python
from src.logger import get_logger

logger = get_logger(__name__)

logger.info("Iniciando processamento")
logger.debug("Valores: x=10, y=20")
logger.warning("Aviso: taxa alta")
logger.error("Erro crítico ocorreu")
```

---

## 🔐 Segurança e Configuração

### Variáveis de Ambiente

Crie um arquivo `.env` baseado em `.env.example`:

```env
# URLs
FNET_BASE_URL=http://fnet.bmfbovespa.com.br
ADVFN_BASE_URL=http://br.advfn.com
BCB_BASE_URL=https://www.bcb.gov.br

# Timeouts
REQUEST_TIMEOUT=30
EXTENDED_TIMEOUT=130

# Logging
LOG_LEVEL=INFO
```

### Exceções Customizadas

```python
from src.exceptions import (
    InvalidInputError,
    HttpRequestError,
    XmlParsingError,
    FundoNotFoundError
)

try:
    dados = service.obter_dados_completos(cnpj, ticker)
except InvalidInputError as e:
    print(f"Erro de validação: {e}")
except HttpRequestError as e:
    print(f"Erro de conexão: {e}")
except XmlParsingError as e:
    print(f"Erro ao processar dados: {e}")
```

---

## 📈 Fluxo de Dados

```
HTTP Request (CNPJ + Ticket)
        │
        ▼
   Validação
        │
        ├─ CNPJ válido?
        ├─ Ticker válido?
        │
        ▼
FundDataService
        │
        ├─> EconomicIndicatorsService
        │   └─> SELIC, IPCA (BCB)
        │
        ├─> FnetService
        │   ├─> Proventos
        │   └─> Informes
        │
        └─> AdvfnService
            └─> Cotações
        │
        ▼
Formatação & Estruturação
        │
        ▼
Prompt para LLM
```

---

## 🎯 Casos de Uso

### 1. Análise Fundamentalista Automatizada

```python
# Gerar análise para um fundo
contexto = service.obter_dados_completos("17311079000174", "SAIC11")

# Usar com LLM para análise
analise = llm.analyze(contexto)
print(analise)
```

### 2. Comparação de Múltiplos Fundos

```python
fundos = [
    ("17311079000174", "SAIC11"),
    ("42888360000111", "DCRA11"),
    ("30982974000189", "VXXV11")
]

for cnpj, ticker in fundos:
    contexto = service.obter_dados_completos(cnpj, ticker)
    print(f"Analisando {ticker}...")
```

### 3. Relatório Executivo

```python
# Gerar relatório para apresentação
contexto = service.obter_dados_completos(cnpj, ticker)

relatorio = f"""
# Análise de Investimento - {ticker}

## Contexto Econômico
{extrair_indicadores(contexto)}

## Performance
{extrair_cotacoes(contexto)}

## Pagamento de Proventos
{extrair_proventos(contexto)}
"""
```

---

## 🤝 Padrões de Design

A arquitetura implementa:

- ✅ **Clean Architecture**: Separação clara de responsabilidades
- ✅ **SOLID Principles**: Código mantível e escalável
- ✅ **Dependency Injection**: Services com dependências explícitas
- ✅ **Repository Pattern**: Isolamento de lógica de dados
- ✅ **Logging Pattern**: Rastreamento estruturado

---

## 📚 Tecnologias Utilizadas

| Tecnologia | Uso |
|------------|-----|
| **Python 3.9+** | Linguagem principal |
| **Azure Functions** | Serverless computing |
| **Requests** | Cliente HTTP |
| **lxml** | Parsing HTML/XML |
| **pytest** | Testes automatizados |
| **Logging** | Rastreamento (built-in) |

---

## 🐛 Troubleshooting

### "CNPJ inválido"
```python
# Certifique-se de usar exatamente 14 dígitos
cnpj = "17311079000174"  # ✅ Correto
cnpj = "17.311.079/0001-74"  # ✅ Também funciona (será limpo)
cnpj = "173110790001"  # ❌ Faltam dígitos
```

### "Timeout na requisição"
Aumente `REQUEST_TIMEOUT` em `.env`:
```env
REQUEST_TIMEOUT=60  # 60 segundos
EXTENDED_TIMEOUT=180  # 3 minutos para ADVFN
```

### "HTTP 429 - Too Many Requests"
O cliente HTTP faz retry automático. Se persistir, adicione delay entre requisições:
```python
import time
for cnpj, ticker in fundos:
    service.obter_dados_completos(cnpj, ticker)
    time.sleep(2)  # 2 segundos de espera
```

---

## 🚀 Próximos Passos

- [ ] Adicionar cache de resultados (Redis)
- [ ] Implementar webhooks para atualizações automáticas
- [ ] Criar Dashboard web (Next.js/React)
- [ ] Integração com mais LLMs (GPT, Grok, etc)
- [ ] Histórico de análises
- [ ] Alertas de preço
- [ ] Exportar relatórios em PDF

---

## 👨‍💻 Autores

### Francke Peixoto

**Desenvolvedor Full Stack | Especialista em Arquitetura de Software**

Francke Peixoto é um desenvolvedor apaixonado por clean code, arquitetura escalável e aplicações inteligentes. Com experiência em:

- 🏗️ **Arquitetura de Software**: Design de sistemas escaláveis
- 🤖 **AI/ML Integration**: Integração com LLMs e AI
- 📊 **Data Engineering**: Processamento e agregação de dados
- ☁️ **Cloud Architecture**: Azure, AWS
- 🔐 **Security & Compliance**: Boas práticas de segurança

**Filosofia**: *"Código limpo, bem testado e bem documentado é um ato de respeito pelo próximo desenvolvedor."*

📧 **Email**: francke.peixoto@outlook.com
🔗 **GitHub**: https://github.com/francke
💼 **LinkedIn**: https://linkedin.com/in/francke-peixoto

---

## 📄 Licença

Este projeto é licenciado sob a MIT License - veja [LICENSE](LICENSE) para detalhes.

---

## 🙏 Agradecimentos

- **BMF Bovespa (FNET)**: Por disponibilizar dados de fundos
- **ADVFN**: Por histórico de cotações
- **Banco Central do Brasil**: Por indicadores econômicos
- **Comunidade Python**: Pelas excelentes bibliotecas

---

## 📞 Suporte

Para dúvidas, sugestões ou issues:

1. Abra uma [Issue no GitHub](https://github.com/francke/peixoto-fundos-net-logicapp/issues)
2. Envie um email para `francke.peixoto@outlook.com`
3. Consulte a [Documentação Técnica](docs/ARCHITECTURE.md)

---

<div align="center">

**Desenvolvido com ❤️ por Francke Peixoto**

⭐ Se este projeto foi útil, deixe uma estrela! ⭐

</div>
