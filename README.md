# PROJETO

ESTE PROJETO É A REPRODUÇÃO DO ALGORITMO DA AT KEARNEY QUE DETERMINA O RISCO DE CADA CLIENTE POR FERRAMENTA DE COBRANÇA

Cientista de Dados: Erich Alves

Engenheiro de Dados: Victor Faro

Produção: 29 / Agosto / 2019

Atualização: 28/ Maio / 2020 --- Lucas G. Rezende 

## Dependências:

- Python
  - Numpy
  - Pandas
  - Scikit-Learn
  - Matplotlib
  - Joblib
- Arquivos
  - Arquivo de Faturamento
  - Arquivo de Notas



## Quanto aos arquivos:

Os arquivos são gerados pelo JOB: **JOB_DUNNIN_ATKEARNEY** dentro do repositorio **REPO_GEGC_SDA**.

Estes por fim são automaticamente movimentados até a pasta default do projeto dentro do servidor da Cobrança.

Obs.: A empresa a ser carregada é configurada através das variáveis de sistema dentro do proprio DS (por padrão a empresa carregada é **CEMAR**)



## Execução do JOB:

Cada pipeline tem seu paralelismo configurado pelo parametro `workers` sendo indicado utilizar no máximo o número de cores - 1.
Considerando que cada processamento alcança picos de até 14GB de RAM, talvez o indicado seja utilizar somente 1 por conta das configurações da máquina.

Para a execução da pipeline é necessário abrir um terminal/prompt na pasta onde esta localizado o script python chamado "ParallelManager.py".

```shell
# Atual caminho do projeto
D://AUTO_RUN//ATKEARNEY//dunnin_atkearney/
```



Dentro da pasta executar o comando:

```shell
python ParallelManager.py --company CEMAR --workers 2 
```

 Variáveis:

- --company : Variável que define qual empresa será processada
- --workers: Número de instancias paralelas que executarão o JOB



## Saídas:

O JOB cria uma pasta de data dentro da pasta da empresa que foi configurada no parametro `--company` .

Nesta pasta, são criados diversos arquivos, dentre estes somente 3 são importantes:

*Modelos*
- modelo_espontaneo.joblib
- modelo_disjuntor.joblib
- modelo_poste.joblib
- modelo_ramal.joblib

*Resultados*
- predict_disjuntor.pkl
- predict_poste.pkl
- predict_espontaneo.pkl
- predict_ramal.pkl

*Avaliação dos Resultados*
- distribuicao_espontaneo.png
- roc_espn.png
- roc_poste.png
- roc_ramal.png
- roc_disjuntor


A informação é persistida dentro do banco EQTLINFO dentro dos owners de gerência de cobrança de cada empresa. A tabela que salva a informação dos scores de ação é a **SCORE_ATKEARNEY** e a tabela do score de espontaneidade é a **SPONTANEOUS_ATK**.



## Processando novos Scores

### Scores de Ação

Para gerar novos scores é necessário compilar uma base com as seguintes informações de entrada:

| NOTA_SERVICO       | CONTA_CONTRATO     | DATA_SELECIONADO   | Flag_Pago         | Perc_Debito_Antigo                            | Flag_Rect    | Flag_Cort_Disj                | Flag_Cort_Post            | Flag_Cort_Ramal           | Flag_Cort_Outros                | Flag_Selec_Visi                | Flag_Exec_Visi          | Flag_Selec_Preneg                       | Flag_Exec_Preneg                   | Consumo_Medio                  | Timing                           | Flag_Parcelamento       | Tempo_Ultimo_Pagamento                 | Perc_Div                                                     | Perc_Cort_Pag             | Flag_Cort  | Perc_Cont_Susp                                | Perc_Nao_Perm                          | Flag_Selec         | Perc_Zero                             | Perc_Fat_Pg                 | Flag_Pub                   | Flag_BaixaRenda            | Flag_Resid              | Flag_Rural        | Perc_Par                                      | Flag_Parc     | Perc_Atraso_30d                                   | Flag_CNR       | Perc_Pg_CNR           | Perc_Nao_Perm_Visi                     | Perc_Nao_Perm_Preneg                            | Perc_Cort_Pag_Disj                 | Perc_Cort_Pag_Post                | Perc_Cort_Pag_Ramal              |
| ------------------ | ------------------ | ------------------ | ----------------- | --------------------------------------------- | ------------ | ----------------------------- | ------------------------- | ------------------------- | ------------------------------- | ------------------------------ | ----------------------- | --------------------------------------- | ---------------------------------- | ------------------------------ | -------------------------------- | ----------------------- | -------------------------------------- | ------------------------------------------------------------ | ------------------------- | ---------- | --------------------------------------------- | -------------------------------------- | ------------------ | ------------------------------------- | --------------------------- | -------------------------- | -------------------------- | ----------------------- | ----------------- | --------------------------------------------- | ------------- | ------------------------------------------------- | -------------- | --------------------- | -------------------------------------- | ----------------------------------------------- | ---------------------------------- | --------------------------------- | -------------------------------- |
| [auto explicativo] | [auto explicativo] | [auto explicativo] | Flag de ação Pago | Percentual de Debito que NÃO é motivo de ação | Se é recorte | Se foi executado no disjuntor | Se foi executado no Poste | Se foi executado no Ramal | Se foi executado em outro local | Se foi selecionado para visita | Se foi visita executada | Se foi selecionado para pre negativação | Se foi executada a pre negativação | Consumo médio 1 ano até a ação | Flag de Atraso maior que 30 dias | Se possuía parcelamento | Atraso minimo desde o ultimo pagamento | percentual médio do valor das faturas em relação a dívida total | Percentual de cortes pago | Se é Corte | Percentual de retornos de "CONTINUA SUSPENSO" | Percentual de "NAO PERMITIU SUSPENSAO" | Se foi selecionado | Percentual de dividas de valor zerado | Percentual de faturas pagas | Se é classe  Poder Publico | Se é subclasse Baixa Renda | Se é classe Residencial | Se é classe Rural | Percentual da divida derivada de parcelamento | Flag Parcelas | Percentual de dívida com atraso dentro de 30 dias | Se possuía CNR | Percetual do CNR pago | Percentual de "NAO PERMITIU" na visita | Percentual de "NAO PERMITIU" na pre negativacao | Percentual de Corte Disjuntor Pago | Percentual de Corte no Poste Pago | Percentual deCorte no Ramal Pago |
| varchar            | varchar            | date               | bool              | number                                        | bool         | bool                          | bool                      | bool                      | bool                            | bool                           | bool                    | bool                                    | bool                               | number                         | bool                             | bool                    | number                                 | number                                                       | number                    | bool       | number                                        | number                                 | bool               | number                                | number                      | bool                       | bool                       | bool                    | bool              | number                                        | bool          | number                                            | bool           | number                | number                                 | number                                          | number                             | number                            | number                           |



Após a obtenção dos scores, é necessário "PIVOTAR" os scores de cada tipo de nota agregando por "CONTA CONTRATO".

Por fim, obtem-se uma tabela do seguinte formato:

| CONTA CONTRATO | SCORE CORTE DISJUNTOR | SCORE CORTE POSTE | SCORE CORTE RAMAL | SCORE CORTE OUTROS | SCORE VISITA | SCORE PRENEGATIVAÇÃO |
| -------------- | --------------------- | ----------------- | ----------------- | ------------------ | ------------ | -------------------- |
| varchar        | number                | number            | number            | number             | number       | number               |



### Scores de Risco

Para gerar novos scores é necessário compilar uma base com as seguintes informações de entrada:

#TODO

Com a obtençao dos scores de risco de cada cliente é necessário agrupaos conforme os limiares:

- Alto - Menor que 10%
- Médio - Menor que 50%
- Baixo - Maior ou igual a 50%
