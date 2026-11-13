# Dados: GS 2026.2 · IA & ML · Bandeira Viva

Todos os dados usados são **reais e públicos**. Não há nenhum dado simulado.

## Fonte

| Item | Detalhe |
|---|---|
| **Instituição** | CETESB, Companhia Ambiental do Estado de São Paulo |
| **Programa** | Monitoramento de balneabilidade das praias litorâneas (coleta semanal aos domingos) |
| **Variável** | Densidade de enterococos (UFC/100 mL), o indicador de contaminação fecal da Resolução CONAMA 274/2000 |
| **Publicação original** | Boletins semanais e relatórios de qualidade das praias da CETESB: https://cetesb.sp.gov.br/praias/ · metodologia: https://cetesb.sp.gov.br/praias/programa-de-monitoramento/ |
| **Extração tabular usada** | Repositório público `amandinhame/dataset-sp-water-quality`, que extraiu as tabelas dos PDFs semanais da CETESB com `tabula` e correções manuais: https://github.com/amandinhame/dataset-sp-water-quality (commit `7a3eb0d`, jul/2021) |
| **Período** | 03/01/2012 a 19/07/2021 |
| **Cobertura do arquivo bruto** | 74.056 coletas, 16 municípios, 176 pontos de coleta do litoral paulista |
| **Recorte usado no modelo** | 4 municípios da Baixada Santista central: São Vicente, Santos, Praia Grande e Guarujá (37 pontos, 16.528 coletas) |
| **Acesso** | Obtido em 13/11/2026 |

## Arquivos

| Arquivo | Conteúdo |
|---|---|
| `brutos/cetesb_enterococos_sp_2012_2021.csv` | Cópia fiel do arquivo `sp_beaches.csv` do repositório de extração. Colunas: `City`, `Beach`, `Date`, `Enterococcus`. SHA-256 `0d5ac2e4dc06705f8a8a2c3293c4ca462b5320480e33d56ab904054dddaf5ec4` |
| `processados/base_modelagem.csv` | Gerado pelo notebook: uma linha por ponto e semana, com os atributos e o alvo (`alvo` = próxima coleta acima de 100 UFC/100 mL) |

## Tratamento aplicado (no notebook)

1. **Recorte** dos 4 municípios da Baixada Santista central.
2. **Identificação do ponto** por município + praia, porque "Praia da Divisa" existe em São Vicente e em Praia Grande.
3. **Semana de coleta:** cada data é atribuída à semana que termina no domingo, já que algumas coletas de domingo foram registradas na segunda-feira. Fica 1 coleta por ponto por semana.
4. **Verificações:** nenhum valor ausente e nenhum valor menor ou igual a zero. Foram removidas 36 coletas duplicadas no mesmo ponto e na mesma semana, mantendo a última; restaram 16.492 coletas.
5. **Transformação** `log(1 + enterococos)` nos atributos, porque a distribuição vai de 1 a 5.000 UFC e tem cauda longa.
6. **Alvo** = próxima coleta do mesmo ponto acima de 100 UFC/100 mL. Só entram pares de coletas com 4 a 10 dias de intervalo, ou seja, a semana seguinte de fato.
7. **Calendário:** temporada de verão e semana de carnaval, calculada a partir da data da Páscoa (`dateutil.easter`).

## Limitações conhecidas dos dados

- 2020 e 2021 têm cerca de metade das coletas dos outros anos, por causa da suspensão do monitoramento na pandemia.
- A extração dos PDFs foi feita por terceiros. Conferimos a consistência (datas semanais, faixa de valores e pontos esperados), mas erros pontuais de digitação no PDF original podem existir.
- Dados de chuva (INMET/CEMADEN) e de maré não foram incorporados nesta versão e estão listados como próximo passo no relatório.
