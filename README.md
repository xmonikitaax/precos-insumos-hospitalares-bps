# Preços e concentração de fornecedores em compras públicas de insumos hospitalares, 2009–2023

Repositório: https://github.com/xmonikitaax/precos-insumos-hospitalares-bps

Rotinas de tratamento e análise dos dados do **Banco de Preços em Saúde (BPS)** do Ministério da Saúde, de **2009 a 2023**. É o repositório de apoio ao Trabalho de Conclusão de Curso do MBA em Data Science e Analytics para Operações, Escola Politécnica da Universidade de São Paulo (POLI USP PRO).

**Painel do trabalho (Power BI):** [abrir o painel publicado](https://app.powerbi.com/view?r=eyJrIjoiODVjNTVjZDMtNjQ2My00NjJiLWJhZDctODIxNzM5MmI3Mzk2IiwidCI6ImZiOTYwY2JmLTVjOTAtNGViMi1hN2VmLWNhMzg5NzBkNjI5MCJ9)

---

## O que este repositório faz

O trabalho mede, nas compras públicas de insumos hospitalares registradas no BPS:
- como o **preço real** dos mesmos itens mudou de 2009 a 2023;
- quão **concentrado** é o fornecimento de cada item;
- como o preço varia com o **estado**, a **modalidade de compra** e o **tamanho da compra**.

Os resultados estão num **painel em Power BI**, com três páginas, que é a base do texto do trabalho. Todo número do texto e todas as tabelas que o painel lê saem destas 16 rotinas. Executadas em ordem, a partir dos arquivos originais, elas refazem tudo.

---

## Do arquivo original à base do painel

| Etapa | Compras | Rotina |
|---|---|---|
| Linhas nos arquivos anuais do BPS, 2009–2023 | 883.655 | 01, 02 |
| Menos as cópias exatas (linhas idênticas em todas as colunas) | 872.804 | 03 |
| Só compradores públicos, pela natureza jurídica do CNPJ (saem 665 sem CNPJ e 36.043 de natureza privada) | 836.096 | 03C, 03B |
| Só as 16 classes de produtos de saúde do Ministério da Saúde (saem 2.167 compras de 144 códigos, e 1 compra sem preço ou código) | **833.928** | 03D, 04B |
| Compras válidas para preço (saem 31.268 com preço ou quantidade implausível) | **802.660** | 04B |
| Compras válidas em item-ano com 3 compras válidas ou mais | **747.684** | 04B |

O item é o código CATMAT mais a unidade de fornecimento: são 19.979 itens.

---

## As regras principais

Todo número do painel responde a uma de duas perguntas:

| Pergunta | Compras que entram | Por quê |
|---|---|---|
| **Quantas vezes compraram, e quem participou?** (compras, órgãos, fornecedores, itens, meses) | **todas** | a compra aconteceu, mesmo quando o preço ou a quantidade foi registrado de forma implausível |
| **Quanto pagaram?** (preço, valor, comparação de preço) | **só as válidas** | um preço digitado errado estragaria a conta |

**Compra implausível:** preço mais de 20 vezes acima ou abaixo da mediana do item no ano, ou quantidade mais de 200 vezes acima da mediana, em item-ano com 3 compras ou mais (rotina 04B). Com 1 ou 2 compras, não há com o que comparar, e a compra fica válida sem ter sido conferida.

**Mínimos de compras:**

| Mínimo | Onde vale | Por quê |
|---|---|---|
| 3 compras válidas no item-ano | mediana de preço, índice de preço real, modalidade, tamanho da compra, tipo de comprador, preço por estado | com 1 ou 2, a "mediana" é o preço de uma compra ou a média de duas |
| 7 compras no item-ano (pelo valor, 7 válidas) | concentração (HHI) e fatia do maior fornecedor | com poucas compras, qualquer fornecedor parece dominante; só a partir de 7 o HHI pode ficar abaixo de 1.500 (faixa do CADE) |
| 10 compras válidas no item-ano | P10 e P90 do preço | com menos, "10% das compras" é menos de uma compra |
| 500 compras no estado, e item comprado em 5 estados | comparação de preço entre estados | estados com poucas compras dão índice instável |

**Casas decimais:** nenhuma coluna que o painel lê é arredondada para menos de 6 casas: as que eram gravadas com 1 a 3 casas passaram a 6, como o HHI, e as demais não são arredondadas. O arredondamento para 1 a 3 casas é feito só na hora de mostrar, no painel e no texto (D88).

As decisões citadas no código como D1, D2… estão descritas no Registro de decisões do trabalho.

---

## As 16 rotinas, na ordem de execução

| # | Rotina | O que faz | Etapa |
|---|---|---|---|
| 1 | `01_INVENTARIO` | Descreve a estrutura dos arquivos originais: campos de cada ano, formato dos códigos e preenchimento | Extração |
| 2 | `02_CONVERSAO_CSV` | Converte cada arquivo anual do BPS de Excel para texto, sem nenhum tratamento | Extração |
| 3 | `03_ETL_HARMONIZACAO` | Unifica nomes e tipos dos campos, padroniza os códigos de item e fornecedor, corrige a leitura da fonte (datas, separador decimal, valor total, colunas trocadas), retira as cópias exatas e agrupa as modalidades | Processamento |
| 4 | `03C_CLASSIFICACAO_RECEITA` | Busca a natureza jurídica de cada comprador no cadastro do CNPJ da Receita Federal (dezembro de 2023). Não altera a base | Processamento |
| 5 | `03B_CLASSIFICACAO_COMPRADOR` | Separa as compras de órgãos e entidades públicas, pela natureza jurídica (Decreto-Lei 200/1967) | Processamento |
| 6 | `03D_CLASSE_CATMAT` | Busca no catálogo oficial de materiais a classe de cada código e a compara com a informada pelo BPS. Não altera a base: a classe é aplicada na 04B | Processamento |
| 7 | `04_ETL_IPCA` | Calcula o fator de correção pela inflação de cada mês, em reais de dezembro de 2023 | Processamento |
| 8 | `04B_BASE_ANALITICA` | Define o item, corrige os preços pela inflação, aplica a classe do produto e retira os códigos fora das 16 classes de saúde, marca as compras implausíveis e calcula a mediana do item no ano e as faixas de quantidade, uma vez só, para todas as rotinas seguintes | Processamento |
| 9 | `05_ETL_PRECO_REAL` | Índice de preço real dos mesmos itens (com o mínimo de 3 compras válidas e, como teste, sem ele), robustez por permanência do item e teste dos compradores municipais | Análise |
| 10 | `06_ETL_HHI` | HHI de cada item em cada ano, pelo número de compras e pelo valor | Análise |
| 11 | `07_ETL_TERRITORIAL` | Índice de preço de cada estado em relação à referência nacional, por ano e por período | Análise |
| 12 | `08_ETL_ESCALA` | Preço de cada compra em relação à mediana do item no ano, por faixa de quantidade | Análise |
| 13 | `09_ETL_CORRELACAO` | Relação entre a mudança da concentração e a mudança do preço do mesmo item, em anos seguidos | Análise |
| 14 | `12_INDICADORES_PAINEL` | Gera, a partir da mesma base, todas as tabelas que o painel lê, inclusive a seção 12B (Concentração e preço) | Análise e painel |
| 15 | `10_VALIDACAO_INDEPENDENTE` | Refaz os principais números a partir dos arquivos originais, com código próprio e as mesmas regras, e compara com os resultados das demais rotinas | Verificação |
| 16 | `11_AUDITORIA` | Guarda, para cada decisão de tratamento, o teste que a sustenta, com uma tabela em CSV na pasta `Auditoria` | Verificação |

Por que essa ordem:
- a 03B usa o resultado da 03C, e a 03D usa a base pública da 03B;
- a 04B lê tudo o que vem antes; as rotinas 05 a 09 partem da 04B, sem recalcular nenhuma regra (a 09 usa também o HHI da 06, e a 08 confere os seus estados com os da 07); a 12 parte da 04B e usa a natureza jurídica da 03C;
- a 12 copia os resultados das rotinas 05 a 08, confere com a 05 e a 06, e a seção 12B usa os pares da 09;
- a 10 e a 11 conferem as saídas anteriores.

---

## Fontes dos dados

Os dados **não estão neste repositório**: são públicos e devem ser baixados das fontes abaixo.

| Fonte | O que é | Usada em |
|---|---|---|
| **Banco de Preços em Saúde** | Compilações anuais do BPS, 2009–2023: https://www.gov.br/saude/pt-br/acesso-a-informacao/banco-de-precos/bases-anuais-compiladas (extração em junho de 2026) | 01 a 03, 10, 11 |
| **Cadastro do CNPJ** | Dados abertos da Receita Federal, pasta `2023-12`: arquivos `Empresas0` a `Empresas9`, `Naturezas`, `Estabelecimentos0` a `Estabelecimentos9` e `Municipios`: https://arquivos.receitafederal.gov.br/index.php/s/YggdBLfdninEJX9?dir=/2023-12 | 03, 03C, 10 |
| **IPCA** | IBGE, Tabela 1737 do SIDRA, base dezembro de 2023 | 04, 10 |
| **Catálogo de materiais (CATMAT)** | API de dados abertos do Compras.gov.br, módulo Material (`4_consultarItemMaterial`). A 03D guarda as respostas brutas, que a 10 relê | 03D, 10 |
| **Fonte B do BPS** | Arquivos do BPS em dados.gov.br, usados só para comparar as duas fontes (D21) | 11, seção 17 |

---

## Como executar

As rotinas foram executadas no **Google Colab**, com os dados no Google Drive.

1. Cada rotina começa com o bloco `PASTA_DADOS`. Ele aponta para a pasta do Drive usada no trabalho. Para usar outra pasta, defina a variável de ambiente `BPS_DADOS`.
2. Dentro dessa pasta, as rotinas leem e gravam estas subpastas:
   - **entradas:** `Base Tratamento Minimo` (arquivos originais do BPS), `IPCA Bruto/tabela1737.csv`, `Base Receita Federal` (subpastas `CNPJ`, `Natureza`, `Estabelecimento` e `Municipio`) e `Comparacao Fontes` (só para a seção 17 da rotina 11);
   - **saídas:** as demais pastas, criadas pelas próprias rotinas (`Base Conversao CSV`, `Base Harmonizacao Campos`, `Base Harmonizacao Publica`, `Base Classificacao Receita`, `Base Classificacao CATMAT`, `IPCA Tratado`, `Base Analitica`, as pastas `… - Publico`, `Base Painel` e `Auditoria`).
3. Execute na ordem da tabela acima.
4. Cada rotina instala as versões que usa na primeira célula (`polars==1.35.2`, `pandas==2.2.3`, `pyarrow`, `scipy`, `xlrd`, `openpyxl`, `requests`, `python-calamine==0.8.3`).

A pasta `Base Painel` é a que o painel em Power BI lê.

---

## Verificação

- **Memória de cálculo:** cada rotina imprime o caminho até os números que o texto cita, e não só o resultado.
- **Auditoria (rotina 11):** cada decisão de tratamento tem um teste e uma tabela em CSV na pasta `Auditoria`. Exemplos:
  - o índice de preço com outras réguas de implausíveis;
  - a decomposição dos elos do índice;
  - o teste dos compradores municipais com o mínimo de 3 compras;
  - a comparação das duas fontes do BPS.
- **Validação independente (rotina 10):** refaz a base a partir dos arquivos originais do BPS, da Receita Federal, do SIDRA e do catálogo, com código próprio em pandas. Os 17 indicadores comparados são iguais aos do pipeline, entre eles 833.928 compras, 31.268 implausíveis, as parcelas de item-ano concentrados e o índice de preço de 2023 (86,9 com o mínimo de 3 compras; 77,1 sem ele).
- **Rotina 12:**
  - a seção 11 confere as tabelas do painel contra as rotinas de origem;
  - a seção 12B só grava depois de três testes: pares iguais aos da rotina 09, medianas iguais e os demais arquivos da `Base Painel` inalterados.
- **Conferência de 25/10/2026:** depois da mudança do índice para o mínimo de 3 compras válidas, a `Base Painel` foi comparada, tabela por tabela e pelo conteúdo, com a cópia anterior. Todas as tabelas ficaram iguais, exceto `res_indice` e `res_robustez`, que ganharam as colunas do índice novo. A saída dessa célula foi retirada desta versão publicada, porque dependia da cópia anterior, que só existia naquela execução. O código da célula foi mantido.

---

## Correções identificadas na fonte

Nenhuma destas falhas gera mensagem de erro: todas produzem resultado errado em silêncio. Estão documentadas no código da rotina 03 e testadas na seção 0 da rotina 11.

- **Separador decimal:** em parte dos anos, o preço vem com ponto decimal, e a regra que tirava todo ponto como milhar inflava valores. O ponto só é tratado como milhar quando há vírgula na mesma cadeia.
- **Data da compra:** em 2009–2012, a data vem com espaço não separável, e a conversão falhava em silêncio.
- **Código do material:** aparece com e sem prefixo, e com seis ou sete dígitos conforme o ano.
- **Município de 2018–2020:** a coluna tem outro nome nesses anos e era descartada.
- **Cópias exatas:** linhas idênticas em todas as colunas (10.851 no total, 9.707 só em 2023) são retiradas.
- **Valores implausíveis:** há compras com preço ou quantidade digitados com erro de ordens de grandeza. Elas ficam na base, entram nas contagens e saem das contas de preço e de valor.

---

## Licença

Código sob licença MIT. Os dados originais são públicos e de responsabilidade do Ministério da Saúde, da Receita Federal e do IBGE.
