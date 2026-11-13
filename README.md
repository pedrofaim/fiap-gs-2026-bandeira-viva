# Bandeira Viva: Machine Learning

> **Saber se a praia está boa antes de sair de casa.**
> CitizenTech da Global Solution 2026.2 · FIAP Engenharia de Software · Inteligência Artificial & Machine Learning

## Equipe

| Integrante | RM |
|---|---|
| Luiz Felipe Motta da Silva | 559126 |
| Nicolas Lorenzo Ferreira da Silva | 557962 |
| Pedro Henrique Faim dos Santos | 557440 |

## Ficha da CitizenTech

| Campo | Conteúdo |
|---|---|
| **Nome** | Bandeira Viva |
| **Cidade e bairro** | São Vicente (SP), Baixada Santista. Piloto em Itararé (Posto 2) e Gonzaguinha |
| **Cidadão beneficiado** | Quem trabalha e vive na areia: ambulantes, quiosqueiros, pescadores e banhistas moradores |
| **Problema** | A informação oficial sobre a qualidade da água sai uma vez por semana, em linguagem técnica, e demora a refletir pioras repentinas |
| **Como o ML ajuda** | Prevê, para cada ponto, o risco de a coleta da próxima semana passar do limite de enterococos, para o app avisar antes da bandeira oficial mudar |
| **ODS** | 3 (saúde e bem-estar), 11 (cidades sustentáveis), 13 (ação climática) e 6 (saneamento) |

## Entrega de IA & ML

| Entregável | Onde está |
|---|---|
| Notebook executado | [`ml/bandeira_viva_ml.ipynb`](ml/bandeira_viva_ml.ipynb) · [abrir no nbviewer](https://nbviewer.org/github/pedrofaim/fiap-gs-2026-bandeira-viva/blob/main/ml/bandeira_viva_ml.ipynb) |
| Relatório (2 páginas) | [`ml/RELATORIO_ML.md`](ml/RELATORIO_ML.md) |
| Documentação dos dados (fonte, link, período) | [`ml/dados/README.md`](ml/dados/README.md) |
| Dependências | [`ml/requirements.txt`](ml/requirements.txt) |
| Figuras e resultados | [`ml/figuras/`](ml/figuras/) · [`ml/resultados/`](ml/resultados/) |
| Vídeo de apresentação (3min42) | [`ml/video/bandeira-viva-ml.mp4`](ml/video/bandeira-viva-ml.mp4) |

### Resumo

- **Pergunta:** a próxima coleta semanal de um ponto da Baixada Santista vai passar de 100 UFC/100 mL de enterococos?
- **Dados reais:** resultados semanais de enterococos da **CETESB**, de 2012 a 2021, em 37 pontos de São Vicente, Santos, Praia Grande e Guarujá.
- **Modelos:** Random Forest (escolhido) e XGBoost. Foram 30 combinações de 3 hiperparâmetros, com validação cruzada temporal só no treino.
- **Teste (2018–2021, nunca visto pelo modelo):** avisou com antecedência **83%** das semanas de água ruim, contra **46%** da regra atual de cinco semanas, com a mesma precisão (cerca de 3 em cada 10 alertas se confirmam).
- **Para o cidadão:** um alerta de risco uma semana antes, que erra para o lado da prudência, sem substituir o boletim oficial da CETESB.

## Como executar

```bash
cd ml
python -m venv .venv
# Windows: .venv\Scripts\activate    ·    Linux/macOS: source .venv/bin/activate
pip install -r requirements.txt
jupyter nbconvert --to notebook --execute --inplace bandeira_viva_ml.ipynb   # ou abra no Jupyter/VS Code e execute tudo
```

A execução completa leva cerca de 2 minutos. O notebook regenera `dados/processados/`, `figuras/` e `resultados/`.

## Estrutura

```
ml/
├── bandeira_viva_ml.ipynb      notebook executado (do problema à conclusão)
├── RELATORIO_ML.md             relatório
├── requirements.txt
├── dados/
│   ├── README.md               fonte, link, período e tratamento
│   ├── brutos/                 resultados da CETESB (arquivo original)
│   └── processados/            base de modelagem gerada pelo notebook
├── figuras/                    gráficos usados no relatório e no vídeo
├── resultados/                 métricas (JSON) e todas as combinações de hiperparâmetros (CSV)
└── video/                      vídeo de apresentação
```
