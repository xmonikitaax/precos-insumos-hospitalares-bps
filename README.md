# Preços e concentração de mercado em compras públicas de insumos hospitalares, 2009–2023

Repositório: https://github.com/xmonikitaax/precos-insumos-hospitalares-bps

Rotinas de tratamento e análise dos dados do **Banco de Preços em Saúde (BPS)** do Ministério da Saúde, referentes ao período de **2009 a 2023**.

Repositório de apoio ao Trabalho de Conclusão de Curso do MBA em Data Science e Analytics para Operações — Escola Politécnica da Universidade de São Paulo (POLI USP PRO).

---

## Fonte dos dados

Compilações anuais do BPS, disponíveis publicamente em
`https://www.gov.br/saude/pt-br/acesso-a-informacao/banco-de-precos/bases-anuais-compiladas`

Extração realizada em **junho de 2026**. As compilações anuais são versões consolidadas e estáveis: arquivos rebaixados posteriormente apresentaram contagem de registros idêntica à da extração original.

Série deflacionada pelo **IPCA** (IBGE, Tabela 1737 do SIDRA), base **dezembro de 2023**.

---

## Ordem de execução

As quatro primeiras rotinas são sequenciais. As demais dependem delas, mas independem entre si.

| # | Rotina | O que faz |
|---|---|---|
| 1 | `01_INVENTARIO` | Diagnóstico dos arquivos originais: campos por ano, formato dos identificadores, cobertura de preenchimento |
| 2 | `02_CONVERSAO_CSV` | Converte as planilhas originais em texto delimitado, sem alterar conteúdo |
| 3 | `03_ETL_HARMONIZACAO` | Unifica nomenclatura, converte tipos, padroniza identificadores de item e fornecedor |
| 4 | `04_ETL_IPCA` | Constrói os fatores de deflação mensais a partir do número-índice do IPCA |
| 5 | `05_ETL_PRECO_REAL` | Série de preços nominais e reais, índice encadeado por item pareado, cenários de robustez |
| 6 | `06_ETL_HHI` | Índice de concentração por item e ano, classificação e restrições por número de fornecedores |
| 7 | `07_ETL_TERRITORIAL` | Recuperação da UF ausente por CNPJ (com validação) e índice de preço relativo por UF |
| 8 | `08_ETL_ESCALA` | Quintis de quantidade, associação entre escala e preço, tabela UF × quintil |
| 9 | `09_ETL_CORRELACAO` | Cinco desenhos de análise da relação entre concentração e preço |
| 10 | `10_VALIDACAO_INDEPENDENTE` | Reconstrói os indicadores a partir dos arquivos originais, sem reutilizar nada do pipeline |

---

## Como executar

Os dados **não estão versionados** neste repositório: são públicos e devem ser
baixados do portal do Ministério da Saúde (ver seção *Fonte dos dados*).

1. Crie uma pasta `dados/` na raiz do repositório e coloque nela os arquivos anuais do BPS
   e o CSV da Tabela 1737 do SIDRA, na estrutura descrita no notebook `03_ETL_HARMONIZACAO`.
2. Cada notebook inicia com o bloco de configuração `PASTA_DADOS`. O padrão aponta para
   `dados/`; para usar outro local, ajuste a variável ou defina `BPS_DADOS` no ambiente.
3. Execute os notebooks na ordem numérica.

Dependências: `polars`, `pandas`, `pyarrow`, `scipy`, `openpyxl`, `xlrd`.

---

## Memória de cálculo

Cada rotina que produz valores citados no trabalho contém células de **memória de cálculo**, que imprimem o caminho aritmético até cada número — não apenas o resultado.

O critério adotado foi: *um número que não pode ser refeito a partir do código não deve ser publicado*.

Valores citados de fonte externa — as faixas de classificação do índice de concentração e o número-índice do IPCA — não são calculados aqui; sua fundamentação está nas referências bibliográficas do trabalho.

---

## Correções de conversão identificadas na fonte

Três falhas de conversão foram identificadas durante o tratamento. Nenhuma gera mensagem de erro; todas produzem resultado incorreto de forma silenciosa. Estão documentadas em comentário no próprio código.

**Separador decimal.** Em parte dos anos o preço é gravado como célula numérica e convertido para texto com ponto decimal. A regra que remove todo ponto como separador de milhar inflava valores em até duas ordens de grandeza. A regra corrigida trata o ponto como milhar apenas quando há vírgula na mesma cadeia.

**Data da compra.** Em 2009–2012 a data vem cercada por espaço não separável, o que faz a conversão falhar silenciosamente e produzir valor nulo. Sem a correção, quatro anos ficam sem data e a deflação mensal torna-se inviável.

**Código do material.** O identificador aparece com e sem prefixo, e com seis ou sete dígitos conforme o ano. A regra de padronização foi definida a partir do diagnóstico de contagem de dígitos por ano, e não por suposição.

---

## Verificação

- Conservação de linhas e de soma de valores entre arquivos originais e base tratada, ano a ano
- Coerência interna entre preço unitário, quantidade e valor total
- Comparação da mesma referência anual obtida em dois canais oficiais distintos
- Análise de sensibilidade à defasagem entre data da compra e data de inserção do registro
- Reconstrução integral dos indicadores por procedimento independente

---

## Licença

Código sob licença MIT. Os dados originais são públicos e de responsabilidade do Ministério da Saúde.
