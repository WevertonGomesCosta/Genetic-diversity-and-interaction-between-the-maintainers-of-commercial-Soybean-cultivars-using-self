# 🌱 Genetic diversity and interaction between the maintainers of commercial Soybean cultivars using selfing

[![DOI](https://zenodo.org/badge/DOI/10.1002/csc2.20816.svg)](https://doi.org/10.1002/csc2.20816)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/wevertoncosta/)
[![ORCID](https://img.shields.io/badge/ORCID-0000--0003--0742--5936-green?style=flat&logo=orcid&logoColor=white)](https://orcid.org/0000-0003-0742-5936)
[![Lattes](https://img.shields.io/badge/Lattes-CNPq-blue)](http://lattes.cnpq.br/2723811288754046)
[![GitHub](https://img.shields.io/badge/GitHub-Profile-black?style=flat&logo=github)](https://github.com/WevertonGomesCosta)
[![Google Scholar](https://img.shields.io/badge/Google%20Scholar-Profile-4285F4?style=flat&logo=google-scholar&logoColor=white)](https://scholar.google.com.br/citations?hl=pt-BR&user=eJNKcHsAAAAJ)
[![LICAE](https://img.shields.io/badge/LICAE-UFV-blue?style=flat&logo=academia&logoColor=white)](https://www.licae.ufv.br/)

---

## 📖 About this repository
=======
# Genetic diversity and interaction between the maintainers of commercial Soybean cultivars using self-organizing maps

## Loading Libraries

This repository contains the **code, data, and reproducible analyses** associated with the article:

**Costa, W.G., et al. (2025). Genetic diversity and interaction between the maintainers of commercial Soybean cultivars using selfing. *Crop Science*.**  
DOI: [10.1002/csc2.20816](https://doi.org/10.1002/csc2.20816)

👉 This project is part of the activities of the [**LICAE (Laboratory of Computational Intelligence and Statistical Learning) at UFV**](https://www.licae.ufv.br/), in collaboration with EMBRAPA and partner institutions.

The analyses were implemented in **R** using a combination of **Random Forest**, **Multiple Correspondence Analysis (MCA)**, and **Self-Organizing Maps (SOM)** to explore the genetic diversity and interactions among maintainers of commercial soybean cultivars.

---

## 🌐 Companion Website

A fully reproducible version of the analysis is available here:  
👉 [GitHub Pages — Reproducible Workflow](https://wevertongomescosta.github.io/Genetic-diversity-and-interaction-between-the-maintainers-of-commercial-Soybean-cultivars-using-self/)

---

## 📌 Workflow Overview

1. **Data preprocessing**  
2. **Variable selection (Random Forest)**  
3. **Dimensionality reduction (MCA)**  
4. **Clustering (SOM)**  
5. **Visualization & interpretation**

---

## 📊 Results

- Identification of **distinct groups of maintainers** of soybean cultivars.  
- Insights into **genetic diversity** and **interaction patterns**.  
- Visualizations linking **morphological descriptors** and **transgenic events** with cultivar performance.  

---

## 🚀 How to Reproduce

### 1. Clone the repository
```bash
git clone https://github.com/WevertonGomesCosta/Genetic-diversity-and-interaction-between-the-maintainers-of-commercial-Soybean-cultivars-using-self.git
cd Genetic-diversity-and-interaction-between-the-maintainers-of-commercial-Soybean-cultivars-using-self
```

### 2. Install dependencies in R
```r
install.packages(c("tidyverse", "FactoMineR", "factoextra", "randomForest", "kohonen"))
```

### 3. Run the analysis
Open the main RMarkdown file and knit it to reproduce the results.  
The outputs (figures, tables) will be saved in the `output/` folder.

---

## 📜 Citation

If you use this repository, please cite the article:

> Costa, W.G., et al. (2025). Genetic diversity and interaction between the maintainers of commercial Soybean cultivars using selfing. *Crop Science*. https://doi.org/10.1002/csc2.20816

---

## 📂 Repository Structure
```
├── data/          # Input data
├── analysis/      # RMarkdown scripts
├── output/        # Figures and tables
├── docs/          # GitHub Pages site (workflowr)
└── README.md
```

---

## 📑 License
This project is licensed under the **Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License**.  
See the [LICENSE](LICENSE) file for details.