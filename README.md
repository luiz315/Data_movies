# Análise de Filmes - TMDB 10k

> Dinheiro compra nota? Filmes estão ficando mais longos? Existe gênero arriscado?

Análise exploratória de 10.000 filmes para responder 3 perguntas de negócio com foco em qualidade dos dados e decisões conscientes de limpeza.

### 🔗 Dataset
- **Fonte:** TMDB Movies (Movies.csv)
- **Tamanho:** 10.000 linhas x 15 colunas
- **Período:** 1890 a 2018
- **Colunas principais:** `revenue`, `runtimes`, `genres`, `vote_average`, `vote_count`, `release_date`

---

### 🎯 Perguntas de Negócio

**P1: Filmes com maior bilheteria são melhor avaliados?**
**P2: A duração dos filmes está aumentando com o tempo?**
**P3: Existe um gênero arriscado que ou é muito bom ou muito ruim?**

---

### 🧹 Qualidade dos Dados e Decisões de Limpeza

Não foram encontradas linhas duplicadas. Principais pontos de atenção:

| Coluna | Faltantes | % | Decisão | Justificativa |
|---|---|---|---|---|
| **revenue** | 6.337 | 63,3% | Filtrar apenas `revenue > 0` para P1 | Valor 0 = ausência de informação no TMDB, não bilheteria nula. Incluir distorceria a correlação e quebraria o log. |
| **production_companies** | 1.485 | 14,8% | Manter como NA | Não impacta as 3 perguntas. |
| **runtimes** | 120 | 1,2% | **Não imputar** | Por se tratar da própria variável de estudo da P2 e a ausência estar concentrada nos anos 90/2000, imputar pela mediana reduziria artificialmente a variância. |
| **genres** | 243 | 2,4% | Manter e desconsiderar no explode | Perda pequena. |
| **release_date** | 8 | 0,08% | Conversão com `errors='coerce'` | 8 datas inválidas viraram `NaT`. Criação de `release_year` como Int64 nullable. |

> Decisão principal: quando a coluna faltante é a própria resposta que queremos medir, optamos por não imputar para preservar a variância original.

---

### 📊 Análises e Resultados

#### P1: Filmes com maior bilheteria são melhor avaliados?
**Resposta: Não. Correlação praticamente nula (0,05).**

- Subconjunto: 3.663 filmes com receita válida e `vote_count > 100`
- Transformação `log10(revenue)` aplicada devido à cauda longa ($12 a $900M)
- Correlação de Pearson `log_revenue` vs `vote_average` = **0,058**
- Scatter plot mostra nuvem dispersa: blockbusters de 100M com nota 4 e filmes de 1M com nota 8.

**Insight:** Qualidade percebida não é explicada pelo tamanho da bilheteria.

#### P2: A duração dos filmes está aumentando com o tempo?
**Resposta: Não. Duração estável em torno de 100 min.**

- Média geral: **100,3 min** (após remover 120 nulos)
- Década de 30: 86 min | Década de 60: **107,1 min (pico dos épicos)** | Década de 2000: 99,4 min | Década de 2010: 100,8 min
- Gráfico anual apresenta picos artificiais em 1915 (~195 min) devido a baixa amostragem (<5 filmes/ano). Agrupamento por década estabiliza a tendência.
- Últimas 4 décadas estáveis entre 99 e 102 min.

**Insight:** Contraria a percepção de que filmes atuais são mais longos.

#### P3: Existe um gênero arriscado?
**Resposta: Sim. Documentary é o mais polarizado.**

Após `split(',')` e `explode()` em `genres`: 10.000 → 23.646 registros.

| Gênero | Média | Desvio Padrão | Count |
|---|---|---|---|
| **Documentary** | 5,68 | **2,47** | 541 |
| Music | 5,98 | 1,90 | 497 |
| Horror | 4,87 | 1,65 | 1.522 |
| Comedy | 5,73 | 1,39 | 3.407 |
| Action | 5,57 | 1,37 | 2.029 |

Documentary tem desvio 80% maior que Action. Indica alta polarização: docs nota 9 e docs nota 2 na mesma categoria. Action e Comedy são os mais consistentes.

---

### 🛠️ Tecnologias
- Python, Pandas, NumPy
- Matplotlib / Seaborn
- Jupyter Notebook

### ▶️ Como Rodar
```bash
pip install pandas matplotlib seaborn
jupyter notebook analise_filmes.ipynb
```

### 📁 Estrutura
```
├── Movies.csv
├── analise_filmes.ipynb
└── README.md
```

### 🔮 Próximos Passos
- [ ] Analisar `vote_count` como peso para média ponderada
- [ ] Investigar por que Horror tem média mais baixa (4,87)
- [ ] Criar dashboard interativo com Streamlit
