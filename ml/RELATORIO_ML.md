# RELATÓRIO DE MACHINE LEARNING: Bandeira Viva

**GS 2026.2 · IA & Machine Learning · FIAP** · Luiz Felipe Motta da Silva (RM 559126) · Nicolas Lorenzo Ferreira da Silva (RM 557962) · Pedro Henrique Faim dos Santos (RM 557440)
Notebook executado: `ml/bandeira_viva_ml.ipynb` · Dados: `ml/dados/README.md`

## 1. Problema e dados reais
O **Bandeira Viva** avisa quem vive e trabalha nas praias de São Vicente (SP) se a água tende a estar segura. A bandeira oficial da CETESB usa as **últimas cinco coletas semanais** e demora a reagir a pioras repentinas.

> **Pergunta de ML:** com base nas coletas recentes do ponto e da região e na época do ano, **a próxima coleta semanal vai passar de 100 UFC/100 mL de enterococos** (limite CONAMA 274/2000)?

É uma classificação binária supervisionada. Os dados são **reais e públicos**: resultados semanais de enterococos da **CETESB** (https://cetesb.sp.gov.br/praias/), em tabelas extraídas dos boletins (github.com/amandinhame/dataset-sp-water-quality).
- **Período:** jan/2012 a jul/2021.
- **Recorte:** 37 pontos de São Vicente, Santos, Praia Grande e Guarujá, com 16.492 coletas.
- **Coletas acima do limite:** 23,8% na região e **36,0% em São Vicente**, o pior município, com pico no verão chuvoso.

## 2. Escolha do modelo
Comparamos **Random Forest** e **XGBoost**. São modelos de árvores adequados a dados tabulares com relações não lineares e muito ruído (contagem de bactérias), pouco sensíveis à escala e com tratamento de desbalanceamento (`class_weight` / `scale_pos_weight`).
- **MLP** foi descartado porque o volume não justifica uma rede neural e a interpretação seria pior.
- **LSTM** foi descartado porque as séries por ponto são curtas e têm falhas.
- **KMeans** foi descartado porque é não supervisionado.

A escolha final foi feita **só pela validação cruzada no treino**: venceu o **Random Forest**.

## 3. Preparação e divisão treino/teste
**Limpeza:**
- ponto identificado por município + praia, porque há nomes repetidos;
- uma coleta por semana (36 duplicatas removidas);
- nenhum valor ausente ou inválido;
- escala `log(1+x)`.

**20 atributos**, usando apenas informação até a semana `t` para prever `t+1`, sem vazamento:
- últimas 4 coletas;
- média, máximo e número de coletas ruins nas últimas 5;
- histórico do ponto;
- nível e % de pontos ruins na região e no município, um indicador indireto de chuva;
- mês, temporada de verão e carnaval;
- município.

**Divisão temporal:**
- **treino** de 2012 a 2017: 10.692 exemplos, 23,5% positivos;
- **teste** de 2018 a jul/2021: 5.351 exemplos, 23,9% positivos, período nunca visto pelo modelo.

## 4. Treino com diferentes hiperparâmetros
- **Random Forest:** `n_estimators` {200, 500} × `max_depth` {4, 8, 16} × `min_samples_leaf` {1, 10, 40}, em **18 combinações**.
- **XGBoost:** `max_depth` {2, 4, 6} × `learning_rate` {0,03; 0,1} × `n_estimators` {150, 400}, em **12 combinações**.
- **Validação cruzada:** 5 dobras temporais **apenas no treino**, cortadas por semana.
- **Métrica:** PR-AUC, adequada para a classe minoritária.

| Melhor de cada | Hiperparâmetros | PR-AUC treino | PR-AUC validação |
|---|---|---|---|
| **Random Forest (escolhido)** | max_depth=4, min_samples_leaf=10, 200 árvores | 0,566 | **0,407** |
| XGBoost | max_depth=2, learning_rate=0,03, 150 árvores | 0,593 | 0,382 |

O limiar de alerta foi calibrado no treino, **por município**, para atingir recall de 75%, a meta da ficha.

## 5. Resultados em treino e teste

| Random Forest | Precisão | Recall | F1 | ROC-AUC | PR-AUC |
|---|---|---|---|---|---|
| Treino | 0,294 | 0,888 | 0,442 | 0,741 | 0,490 |
| Validação | 0,271 | 0,751 | 0,398 | 0,675 | 0,409 |
| **Teste** | **0,281** | **0,832** | **0,421** | **0,668** | **0,415** |
| Regra CETESB no teste | 0,297 | 0,457 | 0,360 | — | — |
| São Vicente: modelo × regra | 0,412 × 0,393 | 0,702 × 0,638 | 0,519 × 0,486 | — | — |

**Overfitting:**
- Árvores profundas decoram o treino. Com `max_depth=16`, o Random Forest chega a PR-AUC de 1,00 no treino, mas fica em 0,39 na validação.
- Mais profundidade nunca melhorou a validação, por isso a busca escolheu o modelo raso.
- O modelo final mantém uma diferença moderada entre treino e teste (ROC-AUC de 0,741 contra 0,668).
- Validação e teste ficaram próximos (0,675 e 0,668), o que mostra uma estimativa confiável.
- Os gráficos de cada item estão no notebook: hiperparâmetros, métricas, matrizes de confusão, curvas ROC e PR, importância e série de Itararé.

## 6. Conclusão para o cidadão
- **Aviso uma semana antes:** no teste, o modelo avisou **83 de cada 100 semanas** de água ruim, contra **46** da regra atual. A precisão foi a mesma: cerca de 3 em cada 10 alertas se confirmam nos dois casos.
- **Em São Vicente:** 70% contra 64% das semanas ruins avisadas.
- **Prudente de propósito:** cerca de 7 em cada 10 alertas não se confirmam. Um falso alarme custa um banho adiado, e um aviso perdido pode custar uma doença. Por isso o app mostra **risco**, não certeza, e mantém o boletim da CETESB como fonte oficial.
- **Limitações:** separação moderada (ROC-AUC de 0,67), sem chuva medida e sem maré, dados até 2021 e 2020 atípico.
- **Próximos passos:** incluir a chuva do INMET (estação A765) e do CEMADEN e a maré, e atualizar a base com os boletins recentes.
