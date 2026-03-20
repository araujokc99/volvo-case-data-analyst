# Teste Técnico — Analista de Dados
**Candidata:** Ketlyn Araújo  
**Dataset:** Car Insurance Data (Kaggle — sagnik1511/car-insurance-data)  
**Objetivo:** Analisar a taxa de sinistro (`OUTCOME`) por segmentos e identificar sinais associados a maior risco, apoiando precificação, underwriting e retenção.

---

## Arquivos entregues

| Arquivo | Descrição |
|---|---|
| `Ketlyn Araujo - Teste Analista de dados 1.ipynb` | Notebook com EDA completa, tratamento, KPIs, segmentação, insights e modelo |
| `Ketlyn Araujo - Teste Analista de dados 2.PBIX` | Dashboard Power BI com 3 páginas (Visão Geral, Drivers de Risco e Recomendações) |
| `README.md` | Este arquivo |

---

## Como executar o notebook

### Pré-requisitos

- Python 3.8+
- Conta no Kaggle com API configurada (`~/.kaggle/kaggle.json`)

### Instalação das dependências

```bash
pip install kagglehub pandas numpy matplotlib seaborn scikit-learn
```

### Execução

1. Abra o arquivo `.ipynb` no Jupyter Notebook, JupyterLab ou VS Code
2. Execute as células em ordem sequencial (Kernel > Restart & Run All)
3. O dataset é baixado automaticamente via `kagglehub` na célula de carregamento
4. O arquivo `car_insurance_tratado.csv` é gerado ao final e usado como fonte do Power BI

### Para abrir o dashboard

- Abra o arquivo `.PBIX` no **Power BI Desktop** (versão março/2024 ou superior)
- A fonte de dados aponta para o CSV exportado pelo notebook — atualize o caminho se necessário em *Transformar Dados > Configurações da Fonte*

---

## Estrutura do notebook

| Seção | Conteúdo |
|---|---|
| 0. Setup | Importações e configuração de estilo |
| 1. Carregamento | Download via kagglehub e leitura do CSV |
| 2. Auditoria | Tipos, missing, duplicatas, outliers, balanceamento de classes |
| 3. Tratamento | Estratégia de imputação e feature engineering |
| 4. KPIs | Total de clientes, sinistros e taxa de sinistro |
| 5. Segmentação | Taxa de sinistro por 7 dimensões + heatmaps |
| 6. Insights | 5+ achados quantificados com implicação de negócio |
| 7. Bônus — Modelo | Regressão Logística com AUC-ROC, F1 e feature importance |
| 8. Exportação | Geração do CSV tratado para o Power BI |

---

## Decisões tomadas

### Valores ausentes (missing)
- **Variáveis numéricas:** imputação pela **mediana** — escolha robusta a outliers, preserva a distribuição sem distorcer extremos
- **Variáveis categóricas:** imputação com `"unknown"` — evita exclusão de registros e mantém rastreabilidade do dado faltante

### Duplicatas
- Removidas via `drop_duplicates()` — evita dupla contagem nos KPIs e distorção nas taxas de sinistro

### Outliers
- **Mantidos intencionalmente.** Em seguros, valores extremos (ex.: muitas multas, alta quilometragem) representam justamente os perfis de maior risco que se quer identificar. Removê-los distorceria a análise de segmentação.

### Feature Engineering
- `RISK_SCORE`: soma ponderada de `SPEEDING_VIOLATIONS`, `DUIS` e `PAST_ACCIDENTS`, segmentada em perfis Baixo / Médio / Alto / Muito Alto
- `CREDIT_SCORE_FAIXA` e `MILEAGE_FAIXA`: variáveis contínuas convertidas em quartis para facilitar a segmentação visual

### Modelo (Bônus)
- **Regressão Logística** escolhida pela interpretabilidade dos coeficientes — adequada para contexto de underwriting, onde explicar a decisão é tão importante quanto a acurácia
- Validação com **AUC-ROC** e **F1-Score** em conjunto de teste (20%, estratificado)
- Features categóricas codificadas com `LabelEncoder`; features numéricas normalizadas com `StandardScaler`

---

## Principais achados

1. **Jovens (16–25) com pouca experiência (0–9 anos)** apresentam a maior taxa de sinistro do dataset — candidatos prioritários a fator de risco adicional na precificação
2. **Infrações de velocidade** funcionam como preditor direto: clientes com ≥ 1 multa têm taxa de sinistro significativamente superior aos sem multas
3. **DUI é o maior preditor individual** — clientes com histórico de DUI apresentam diferencial de risco expressivo frente aos sem ocorrência
4. **Credit score inversamente correlacionado**: menor quartil de crédito → maior taxa de sinistro; variável relevante para o modelo atuarial
5. **Veículos mais antigos** (antes de 2015) apresentam taxa de sinistro superior aos mais novos

---

## Limitações do dataset

- Dataset sintético/público do Kaggle — distribuições podem não refletir a realidade de um portfólio real de seguros
- Ausência de variável temporal: não é possível analisar sazonalidade ou tendência ao longo do tempo
- Variáveis categóricas como `AGE` e `DRIVING_EXPERIENCE` já vêm em faixas — granularidade limitada para modelagem contínua
- Desbalanceamento moderado de classes em `OUTCOME` pode afetar métricas de precisão; considerar técnicas de resampling (SMOTE) em modelos futuros
- Sem variáveis de localização geográfica, o que limita análises regionais relevantes para precificação
