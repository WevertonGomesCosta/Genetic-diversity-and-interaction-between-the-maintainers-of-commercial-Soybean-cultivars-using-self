# Genetic Diversity and Interaction Between the Maintainers of Commercial Soybean Cultivars Using Self-Organizing Maps

Este repositório contém o código e a documentação do pipeline de análise de diversidade genética e interação entre mantenedores de cultivares comerciais de soja, utilizando **Random Forest**, **Análise de Correspondência Múltipla (MCA)** e **Mapas Auto-Organizáveis de Kohonen (SOM)**.

---

## 📂 Estrutura do Projeto

- `data/` → arquivos de dados de entrada (`data.xlsx`, `yield.xlsx`)
- `scripts/` → código em RMarkdown com as etapas do pipeline
- `README.md` → documentação do projeto

---

## 🚀 Pipeline Analítico

### 1. Preparação dos Dados
- Importação dos dados originais.
- Exclusão de mantenedores com ≤ 5 observações.
- Conversão de variáveis categóricas em fatores.

### 2. Seleção de Variáveis (Random Forest)
- Ajuste de modelo Random Forest para identificar descritores mais relevantes.
- Exclusão dos 4 descritores menos importantes.
- Visualização da importância das variáveis (índices de Acurácia e Gini).

### 3. Redução de Dimensionalidade (MCA)
- Aplicação da **Análise de Correspondência Múltipla**.
- Cálculo dos autovalores e variância explicada.
- Seleção das dimensões mais informativas.

### 4. Agrupamento (Self-Organizing Maps - SOM)
- Treinamento de mapas auto-organizáveis com topologia hexagonal.
- Visualização da convergência do treinamento.
- Análise das distâncias entre neurônios vizinhos.

### 5. Visualizações
- Distribuição dos mantenedores nos clusters SOM.
- Distribuição das categorias de descritores (cor da pubescência, tipo de crescimento, cor do hilo, etc.) por cluster.
- Gráficos de pizza sobrepostos ao grid SOM para interpretação biológica.

### 6. Produtividade
- Integração com dados de produtividade (`yield.xlsx`).
- Gráficos de evolução da produtividade por **evento transgênico** e por **tipo de crescimento** ao longo dos anos.

### 7. Conclusões
- O pipeline combina **Machine Learning (Random Forest)**, **Estatística Multivariada (MCA)** e **Redes Neurais Não Supervisionadas (SOM)**.
- Permite identificar padrões de diversidade genética entre mantenedores.
- Relaciona descritores morfológicos e moleculares com tendências de produtividade.

---

## 📊 Tecnologias Utilizadas

- **R** (pacotes: `tidyverse`, `FactoMineR`, `factoextra`, `kohonen`, `ggplot2`, `reshape2`)
- **RMarkdown** para documentação reprodutível
- **Git/GitHub** para versionamento

---

## 🔧 Como Reproduzir

1. Clone este repositório:
   ```bash
   git clone https://github.com/WevertonGomesCosta/Genetic-diversity-and-interaction-between-the-maintainers-of-commercial-Soybean-cultivars-using-self.git
   ```
2. Abra o projeto no **RStudio**.
3. Execute o arquivo `.Rmd` em `scripts/` para reproduzir todas as análises.

---

## 📌 Autor

- **Weverton Gomes Costa**  
  Universidade Federal de Viçosa – Programa de Pós-Graduação em Genética e Melhoramento

---

## 📜 Licença

Este projeto é distribuído sob a licença MIT. Consulte o arquivo `LICENSE` para mais detalhes.

Perfeito, Weverton! Aqui está a versão em **inglês** do seu `README.md`, estruturada de forma clara e profissional para o GitHub:

--- 

# Genetic Diversity and Interaction Between the Maintainers of Commercial Soybean Cultivars Using Self-Organizing Maps

This repository contains the code and documentation for the pipeline analyzing **genetic diversity** and **interaction among maintainers of commercial soybean cultivars**, using **Random Forest**, **Multiple Correspondence Analysis (MCA)**, and **Kohonen Self-Organizing Maps (SOM)**.

---

## 📂 Project Structure

- `data/` → input data files (`data.xlsx`, `yield.xlsx`)  
- `scripts/` → RMarkdown scripts with the full analysis pipeline  
- `README.md` → project documentation  

---

## 🚀 Analytical Pipeline

### 1. Data Preparation
- Import of raw datasets.  
- Removal of maintainers with ≤ 5 observations.  
- Conversion of categorical descriptors into factors.  

### 2. Variable Selection (Random Forest)
- Random Forest model fitted to identify the most relevant traits.  
- Exclusion of the 4 least important descriptors.  
- Visualization of variable importance (Accuracy and Gini indices).  

### 3. Dimensionality Reduction (MCA)
- Application of **Multiple Correspondence Analysis**.  
- Calculation of eigenvalues and explained variance.  
- Selection of the most informative dimensions.  

### 4. Clustering (Self-Organizing Maps - SOM)
- Training of SOM with hexagonal topology.  
- Visualization of training convergence.  
- Analysis of distances between neighboring neurons.  

### 5. Visualizations
- Distribution of maintainers across SOM clusters.  
- Distribution of trait categories (pubescence color, growth type, hilum color, etc.) within clusters.  
- Pie charts overlaid on the SOM grid for biological interpretation.  

### 6. Productivity
- Integration with productivity dataset (`yield.xlsx`).  
- Graphs of productivity trends by **event type** and **growth type** across years.  

### 7. Conclusions
- The pipeline combines **Machine Learning (Random Forest)**, **Multivariate Statistics (MCA)**, and **Unsupervised Neural Networks (SOM)**.  
- Identifies patterns of genetic diversity among maintainers.  
- Links morphological and molecular descriptors with productivity trends.  

---

## 📊 Technologies Used

- **R** (packages: `tidyverse`, `FactoMineR`, `factoextra`, `kohonen`, `ggplot2`, `reshape2`)  
- **RMarkdown** for reproducible documentation  
- **Git/GitHub** for version control  

---

## 🔧 How to Reproduce

1. Clone this repository:
   ```bash
   git clone https://github.com/WevertonGomesCosta/Genetic-diversity-and-interaction-between-the-maintainers-of-commercial-Soybean-cultivars-using-self.git
   ```
2. Open the project in **RStudio**.  
3. Run the `.Rmd` file in `scripts/` to reproduce the full analysis.  

---

## 📌 Author

- **Weverton Gomes Costa**  
  Federal University of Viçosa – Graduate Program in Genetics and Plant Breeding  

---

## 📜 License

This project is distributed under the MIT License. See the `LICENSE` file for details.