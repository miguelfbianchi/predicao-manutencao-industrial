# ShieldPredict: Pipeline Inteligente de Manutenção Preditiva

O **ShieldPredict** é um software de inteligência artificial voltado para a Indústria 4.0, projetado para monitorar sensores industriais em tempo real, identificar padrões ocultos de estresse termomecânico e prever quebras mecânicas antes que elas causem paradas não planejadas na linha de produção.

---

## 1. O Problema

No cenário industrial moderno, as paradas não planejadas causadas por falhas em equipamentos geram prejuízos milionários e atrasos severos na cadeia de suprimentos. O **ShieldPredict** resolve esse problema ao atuar como um sistema de detecção precoce de anomalias mecânicas.

Utilizando uma base de dados histórica com 10.000 registros de sensores, o software analisa variáveis críticas (como torque, rotação e temperatura) para classificar o estado operacional da máquina entre:
*   **`Falha = 0`**: Funcionamento estável e seguro.
*   **`Falha = 1`**: Risco iminente de avaria ou quebra mecânica detectada.

O principal desafio analítico deste projeto reside no **severo desbalanceamento dos dados**, onde os eventos de quebra representam apenas **~3,5%** do histórico operacional total, exigindo um pipeline rigoroso de preparação de dados para evitar o fenômeno *Garbage In, Garbage Out*.

---

## 2. Arquitetura do Pipeline Técnico

O software foi desenvolvido seguindo um fluxo de trabalho estruturado e documentado em 7 fases sequenciais no Python:

### Detalhamento das Fases:
*   **Fase 1: Análise Exploratória (EDA):** Mapeamento de dimensões, resumos estatísticos com `.describe()` e geração de gráficos analíticos otimizados (Histogramas, Gráficos de Proporção Empilhados e Heatmap de Correlação) focados na redução da carga cognitiva.
*   **Fase 2: Limpeza e Tratamento (Data Prep):** Remoção de duplicadas, imputação estatística de dados ausentes com base no formato da distribuição e detecção visual de outliers via Boxplots.
*   **Fase 3: Feature Engineering:** Criação de novas variáveis físicas que traduzem o desgaste real da máquina.
*   **Fase 4: Divisão e Balanceamento:** Separação em 80% treino e 20% teste blindada com o parâmetro `stratify=y`. Aplicação da técnica **SMOTE** (Synthetic Minority Over-sampling Technique) *exclusivamente* nos dados de treino para eliminar o risco de vazamento de dados (*Data Leakage*).
*   **Fase 5: Escalonamento de Variáveis:** Aplicação do `StandardScaler` apenas para as colunas do modelo KNN. Os dados destinados à Árvore de Decisão foram mantidos na escala pura, visto que algoritmos de árvore são matematicamente imunes à escala por utilizarem cortes ortogonais isolados em cada eixo.
*   **Fase 6: Ajuste de Parâmetros e Combate ao Overfitting:** Varredura de hiperparâmetros. O KNN foi testado com variações de vizinhos (K) e a Árvore de Decisão com variações de profundidade (max_depth), monitorando o *Gap* de acurácia entre treino e teste.
*   **Fase 7: Avaliação e Veredito Final:** Comparação cruzada das taxas de acerto nos dados de teste nunca vistos pelos modelos para a seleção do classificador ideal de produção.

---

## 3. Tecnologias e Bibliotecas Utilizadas

*   **Linguagem Core:** Python 3.8+
*   **Manipulação e Tratamento de Dados:** Pandas, NumPy
*   **Machine Learning Traditional:** Scikit-Learn (sklearn)
*   **Algoritmo de Reamostragem:** Imbalanced-Learn (imblearn - SMOTE)
*   **Visualização de Dados & Storytelling:** Matplotlib, Seaborn

---

## 4. Desempenho, Veredito Final e Limitações

### Resultados dos Modelos Campeões
Após a varredura de hiperparâmetros, os modelos que apresentaram o melhor equilíbrio entre aprendizado e capacidade de generalização foram:

*   **KNN (K = 13 vizinhos):**
    *   Acurácia Treino: **92,04%** | Acurácia Teste: **90,30%**
    *   Gap de Generalização: **1,8878%**
*   **Árvore de Decisão (max_depth = 4):**
    *   Acurácia Treino: **91,64%** | Acurácia Teste: **89,95%**
    *   Gap de Generalização: **1,8415%**

### Veredito de Seleção
Considerando estritamente os testes realizados e adotando a **Acurácia** como métrica única de avaliação, ambos os modelos entregaram um desempenho final quantitativamente muito semelhante. No entanto, para a implantação neste cenário, o **KNN com 13 vizinhos foi o modelo selecionado**. 

A justificativa baseia-se na estabilidade: o KNN demonstrou um comportamento consistente ao longo do ajuste de parâmetros, enquanto as Árvores de Decisão apresentaram oscilações estruturais que comprometem sua confiabilidade para o ambiente produtivo.

### Análise Crítica e Limitações do Modelo
Em termos práticos e operacionais, **nenhum dos dois modelos desenvolvidos é útil para a empresa atualmente**. 

A base histórica apresenta uma proporção severamente desbalanceada de **96,5% de operação normal** para apenas **3,5% de falhas**. Isto significa que um algoritmo simplista que chutasse estritamente "Operação Normal (0)" para 100% dos casos obteria uma acurácia de **96,5%**. Como o nosso melhor modelo atingiu **90,30% no teste**, sua performance geral é estatisticamente pior do que o *baseline* de um "chute" cego. 

O uso isolado da Acurácia não informa bem a eficácia real do sistema, reforçando a necessidade obrigatória de avaliar o pipeline sob o prisma de outras métricas de classificação.

---

## 5. Próximas Melhorias e Próximos Passos

Para retirar o projeto do estado de obsolescência matemática e viabilizar seu ganho financeiro na fábrica, as seguintes melhorias foram mapeadas e serão aplicadas:

### 1. Reengenharia de Features e Conhecimento Técnico
*   **Reavaliação de Dimensionalidade:** A redução de variáveis executada pode ter sido agressiva demais, eliminando sinais fracos de falha. O conjunto de dados será expandido.
*   **Mineração de Textos/Pistas:** Serão geradas novas variáveis preditoras utilizando Engenharia de Recursos reversa com base nas pistas textuais e colunas de "motivos de falha" contidas nas anotações de engenharia.
*   **Features de Especialista:** Criação de variáveis mais elaboradas e complexas baseadas no conhecimento técnico específico de mecânica de motores industriais.

### 2. Testes de Amostragem Comparativos
*   O pipeline atual utilizou a técnica de superamostragem sintética (SMOTE). Será estruturado um teste comparativo robusto utilizando técnicas de **Undersampling (Subamostragem)** para analisar se a remoção de dados normais redundantes clareia a fronteira de decisão dos algoritmos.

### 3. Melhorias de Arquitetura de Software
*   **Modularização do Pipeline:** Transfomação do código monolítico do notebook em módulos Python (`.py`) independentes (ingestão, tratamento, treino e avaliação). Isso trará flexibilidade para automatizar rotinas de testes com outros modelos e algoritmos.
*   **Script Executável (.py):** Geração de um script final unificado e executável via terminal para permitir uma execução simplificada, ágil e pronta para agendamentos automáticos na infraestrutura local da empresa.


---

## 6. Como Executar o Projeto

1.  **Clone o repositório:**
    ```bash
    git clone https://github.com
    cd shieldpredict
    ```

2.  **Instale as dependências obrigatórias:**
    ```bash
    pip install pandas numpy matplotlib seaborn scikit-learn imbalanced-learn
    ```

3.  **Baixe a base de dados:**
    Certifique-se de fazer o download do arquivo `manutencao_preditiva.csv` disponibilizado pelo Departamento de Engenharia no link do Google Drive e coloque-o na pasta raiz do projeto.

4.  **Execute o Notebook:**
    Abra o seu ambiente de preferência (Jupyter, VS Code ou Google Colab) e execute o arquivo `pipeline_preditivo.ipynb`.

---
*Desenvolvido como projeto prático de encerramento de módulo em Ciência de Dados.*
