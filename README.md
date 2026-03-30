# Classificação de Sentimento Binário com KNIME

Pipeline completo de classificação de sentimento (positivo vs. negativo) construído no **KNIME Analytics Platform 5.11**, como exercício de mestrado na UFRJ.

## Objetivo

Construir, analisar e avaliar um classificador de textos para sentimento binário utilizando o KNIME, compreendendo de forma prática todas as etapas do pipeline: pré-processamento, vetorização, treinamento e avaliação.

## Dataset

**Sentiment Labelled Sentences** — [UCI Machine Learning Repository](https://archive.ics.uci.edu/ml/datasets/Sentiment+Labelled+Sentences)

O dataset contém frases curtas rotuladas como positivas (1) ou negativas (0), extraídas de três fontes:

| Fonte  | Descrição             | Instâncias |
|--------|-----------------------|------------|
| Amazon | Reviews de produtos   | ~1.000     |
| Yelp   | Reviews de estabelecimentos | ~1.000 |
| IMDB   | Reviews de filmes     | ~1.000     |

As três fontes foram concatenadas, totalizando aproximadamente **3.000 frases**.

## Arquitetura do Pipeline

```
┌─────────────┐   ┌─────────────┐   ┌─────────────┐
│ File Reader  │   │ File Reader  │   │ File Reader  │
│  (Amazon)    │   │   (Yelp)     │   │   (IMDB)     │
└──────┬───────┘   └──────┬───────┘   └──────┬───────┘
       │                  │                  │
       └──────────────────┼──────────────────┘
                          │
                   ┌──────▼──────┐
                   │ Concatenate  │
                   └──────┬──────┘
                          │
                   ┌──────▼──────┐
                   │Column Rename │  (Column0→text, Column1→label)
                   └──────┬──────┘
                          │
                   ┌──────▼──────────┐
                   │Number To String  │  (label: int → string)
                   └──────┬──────────┘
                          │
              ┌───────────▼────────────┐
              │  Strings to Document   │  (tokenizer: OpenNLP English)
              └───────────┬────────────┘
                          │
                   ┌──────▼──────────┐
                   │ Case Converter   │  (lowercase)
                   └──────┬──────────┘
                          │
                   ┌──────▼──────────────┐
                   │ Punctuation Erasure  │
                   └──────┬──────────────┘
                          │
                   ┌──────▼──────────────┐
                   │  Stop Word Filter    │  (English built-in)
                   └──────┬──────────────┘
                          │
                   ┌──────▼──────────────┐
                   │  Snowball Stemmer    │  (English)
                   └──────┬──────────────┘
                          │
                   ┌──────▼──────────────────┐
                   │  Bag of Words Creator    │  (etapa intermediária)
                   └──────┬──────────────────┘
                          │
                   ┌──────▼──────┐
                   │     TF       │  (Term Frequency)
                   └──────┬──────┘
                          │
                   ┌──────▼──────┐
                   │     IDF      │  (Inverse Document Frequency)
                   └──────┬──────┘
                          │
                   ┌──────▼──────────────────┐
                   │   Table Partitioner      │  (70/30, stratified, seed=42)
                   └──────┬─────────┬────────┘
                    (treino)       (teste)
                          │              │
           ┌──────────────┼──────────┐   │
           │              │          │   │
     ┌─────▼─────┐ ┌─────▼─────┐ ┌──▼──▼────┐
     │Naive Bayes│ │ Logistic  │ │ Decision │
     │  Learner  │ │Regression │ │   Tree   │
     │           │ │  Learner  │ │  Learner │
     └─────┬─────┘ └─────┬─────┘ └────┬─────┘
           │              │             │
     ┌─────▼─────┐ ┌─────▼─────┐ ┌────▼─────┐
     │Naive Bayes│ │ Logistic  │ │ Decision │
     │ Predictor │ │Regression │ │   Tree   │
     │           │ │ Predictor │ │ Predictor│
     └─────┬─────┘ └─────┬─────┘ └────┬─────┘
           │              │             │
     ┌─────▼─────┐ ┌─────▼─────┐ ┌────▼─────┐
     │  Scorer   │ │  Scorer   │ │  Scorer  │
     └───────────┘ └───────────┘ └──────────┘
```

## Resultados

### Métricas Gerais

| Métrica | Naive Bayes | Logistic Regression | Decision Tree |
|---------|:-----------:|:-------------------:|:-------------:|
| Acurácia | 51,73% | 51,83% | **57,78%** |
| Cohen's Kappa | 0,023 | 0,032 | 0,154 |
| Classificações Corretas | 5.675 | 5.686 | 6.339 |
| Classificações Erradas | 5.296 | 5.285 | 4.632 |

### Métricas Derivadas (Classe Positiva)

| Métrica | Naive Bayes | Logistic Regression | Decision Tree |
|---------|:-----------:|:-------------------:|:-------------:|
| Precisão | 52,83% | 52,13% | **57,71%** |
| Recall | 16,60% | 24,28% | **52,78%** |
| F1-Score | 25,26% | 33,12% | **55,14%** |

### Matrizes de Confusão

**Naive Bayes**

|  | Pred. Negativo | Pred. Positivo |
|--|:-:|:-:|
| **Real Negativo** | 4.780 ✅ | 799 ❌ |
| **Real Positivo** | 4.497 ❌ | 895 ✅ |

**Logistic Regression**

|  | Pred. Negativo | Pred. Positivo |
|--|:-:|:-:|
| **Real Negativo** | 4.377 ✅ | 1.202 ❌ |
| **Real Positivo** | 4.083 ❌ | 1.309 ✅ |

**Decision Tree**

|  | Pred. Negativo | Pred. Positivo |
|--|:-:|:-:|
| **Real Negativo** | 3.493 ✅ | 2.086 ❌ |
| **Real Positivo** | 2.546 ❌ | 2.846 ✅ |

## Análise

### Desempenho

Nenhum dos três modelos apresentou desempenho satisfatório. O baseline aleatório para um problema binário balanceado é de ~50%, e o melhor modelo (Decision Tree) atingiu apenas 57,78%. Os valores de Cohen's Kappa próximos de zero confirmam concordância quase nula além do acaso.

### Erros Mais Frequentes

Todos os modelos apresentam **falsos negativos como erro dominante** — frases positivas classificadas incorretamente como negativas. O Naive Bayes acerta apenas 16,60% dos positivos, o Logistic Regression 24,28%, e o Decision Tree 52,78%. Isso indica maior dificuldade dos modelos em identificar sentimentos positivos.

### Influência do Pré-processamento

O pré-processamento influencia significativamente os resultados. A remoção de stopwords pode ter sido prejudicial, pois palavras como "not", "no" e "very" carregam informação sentimental importante (ex: "not good" vira "good" após remoção). A representação TF-IDF foi aplicada no nível de termos individuais sem agregação por documento (Document Vector), limitando a capacidade discriminativa dos modelos.

### Overfitting vs. Underfitting

Os modelos apresentam **underfitting**, não overfitting. Mesmo o Decision Tree sem poda (configuração que favorece overfitting) alcançou apenas 57,78%. A limitação está na representação dos dados, não na capacidade dos modelos.

## Configurações dos Modelos

| Modelo | Configuração |
|--------|-------------|
| Naive Bayes Learner | Classification column: `label`; Default probability: 0,0001 |
| Logistic Regression Learner | Target: `label`; Solver: Stochastic Average Gradient; Coluna `text` excluída |
| Decision Tree Learner | Class column: `label`; Quality: Gini Index; No Pruning; Reduced Error Pruning; Min records: 2 |

## Pré-requisitos

- **KNIME Analytics Platform 5.11**
- Extensão: **KNIME Textprocessing**

## Estrutura do Repositório

```
├── README.md                                  # Este arquivo
├── relatorio_classificacao_sentimento.pdf      # Relatório completo em PDF
├── data/
│   ├── amazon_cells_labelled.txt              # Dataset Amazon
│   ├── yelp_labelled.txt                      # Dataset Yelp
│   └── imdb_labelled.txt                      # Dataset IMDB
└── knime-workflow.knwf                    # Workflow exportado do KNIME
```

> **Nota:** Para exportar o workflow do KNIME, vá em `File > Export KNIME Workflow` e salve o arquivo `.knwf`.

## Como Executar

1. Instale o [KNIME Analytics Platform 5.11](https://www.knime.com/downloads)
2. Instale a extensão **KNIME Textprocessing** via `File > Install KNIME Extensions`
3. Importe o workflow via `File > Import KNIME Workflow`
4. Ajuste os caminhos dos arquivos de dados nos nós File Reader
5. Execute o workflow completo (botão verde ou Shift+F7)

## Possíveis Melhorias

- Utilizar o nó **Document Vector** para agregar a representação TF-IDF ao nível de documento
- Implementar **n-gramas** (bigramas, trigramas) para capturar contexto entre palavras
- Preservar **negadores** (not, no, never) na etapa de remoção de stopwords
- Explorar representações mais avançadas como **word embeddings**
- Testar outros algoritmos como **SVM** ou **Random Forest**

## Tecnologias

- KNIME Analytics Platform 5.11
- KNIME Textprocessing Extension
- OpenNLP English Word Tokenizer
- Snowball Stemmer
- TF-IDF (Term Frequency - Inverse Document Frequency)
