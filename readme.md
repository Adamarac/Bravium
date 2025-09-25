# Análise de Reviews de Clientes

Este projeto realiza a análise de satisfação de clientes a partir de avaliações textuais, notas de review e informações de entrega. O pipeline combina **processamento de linguagem natural (NLP)**, **análise de sentimento** e **classificação semântica de categorias** para gerar métricas de satisfação e motivos dos comentários.

## Etapas do Pipeline

### 1. Pré-processamento de texto
- Conversão para minúsculas, remoção de espaços extras, URLs, e-mails, números, emojis e caracteres especiais.
- Normalização de acentos e remoção de stopwords em português.
- Tokenização e lematização usando **spaCy**.

### 2. Criação de targets
- **`target_1`**: normalização da nota do review entre -1 e 1.
- **`target_2`**: sentimento do texto combinado (título + comentário) usando **FastText** e lexicon manual. 
- **`target_3`**: atraso de entrega normalizado. 
- **`target_5`**: inconsistência entre nota e atraso de entrega.
- **`target_6`**: qualidade do comentário baseada no tamanho do texto.
- **`final_score`**: score ponderado combinando os targets.
- **`categorical_score`**: binarização do score final (0 = negativo, 1 = positivo). Se o score for menor que 0 negativo e se for maior positivo.

### 3. Treinamento de modelo de classificação
- Modelo: **XGBoost Classifier**.
- Features: `target_1`, `target_2`, `target_3`, `target_5`, `target_6`.
- Target: `categorical_score`.
- Divisão de dados: 70% treino / 30% teste.
- Parâmetros principais: `max_depth=3`, `n_estimators=100`, `learning_rate=0.1`, `eval_metric='mlogloss'`.
- Avaliação: acurácia e classification report. Também realizado **cross-validation estratificado (5 folds)** para estimativa mais robusta.

### 4. Classificação semântica de categorias
- Treinamento de **FastText** com os tokens dos comentários (vector_size=50, window=3, min_count=1, epochs=10).
- Dicionário de categorias (`entrega`, `qualidade`, `atendimento`, `preço`, `embalagem`, `disponibilidade`, `funcionamento`, `experiencia_compra`, `satisfacao_geral`) e sinônimos.
- Cálculo de scores por categoria:
  - Match direto com sinônimos.
  - Similaridade semântica via vetores FastText (cosine similarity, threshold=0.3).
- Categoria dominante é escolhida como **motivo principal do comentário**.

### 5. Geração de motivos e exportação
- Templates pré-definidos combinam categoria dominante e sentimento para gerar textos descritivos (ex.: "elogios ao atendimento").
- Conversão do `categorical_score` para rótulos textuais "positivo" ou "negativo".
- Exportação do DataFrame final para CSV (`analysis_df.csv`) descartando colunas intermediárias.

## Resultados
- **`motivo_avaliacao`**: descrição textual do motivo do comentário.
- **`sentimento`**: rótulo de sentimento geral do review.
- CSV pronto para análise e visualização.

## Possíveis melhorias
- **Otimização de parâmetros do modelo**: ajustar e realizar **tuning** para melhorar acurácia.  
- **Classificação multilabel**: atualmente cada comentário é atribuído a uma única categoria; permitir múltiplas categorias aumentaria a granularidade e assertividade da análise.  
- **Armazenamento vetorial**: salvar vetores FastText ou embeddings em uma base vetorial para evitar retrain a cada execução e agilizar consultas.  
- **Escalabilidade e performance**: processamento paralelo de texto e treinamento do modelo, e otimização do uso de memória para datasets maiores.  
- **Aumentar ou melhorar o dicionário de sinônimos**: expandir o lexicon atual ou adotar abordagens mais avançadas (ex.: embeddings contextualizados) para melhorar a precisão da classificação semântica.