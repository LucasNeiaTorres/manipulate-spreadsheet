# manipulate-spreadsheet

Notebook que fecha uma planilha de notas no Google Sheets sem abrir a planilha: lê
faltas e três provas de cada aluno pela API do `gspread`, aplica as regras de
aprovação e escreve de volta, em cada linha, a situação final e — quando é o caso —
a nota mínima que o aluno precisa tirar no exame.

## Contexto

Exercício de automação de planilha: em vez de calcular a situação de cada aluno na
mão, o notebook autentica no Google (no Colab, com a credencial do próprio ambiente),
abre a planilha pelo nome e percorre as linhas dos alunos, uma a uma.

## As regras implementadas

Para cada aluno, a partir das faltas e da média das três provas:

| Condição | Situação escrita | Nota final |
|---|---|---|
| mais de 15 faltas | `Reprovado por Falta` | `0` |
| média < 50 | `Reprovado por Nota` | `0` |
| 50 ≤ média < 70 | `Exame Final` | a nota necessária (ver abaixo) |
| média ≥ 70 | `Aprovado` | `0` |

O teto de 15 faltas é 25% de 60 aulas.

## O detalhe interessante: a nota do exame sai de uma conta, não de uma constante

Para quem cai em `Exame Final`, o notebook não devolve um número fixo — ele calcula
a **menor nota que ainda aprova**. A regra da média final é `(NAF + m) / 2 ≥ 50`,
onde `m` é a média já obtida e `NAF` a nota do exame. Isolando `NAF`:

```
(NAF + m) / 2 ≥ 50   →   NAF + m ≥ 100   →   NAF ≥ 100 - m
```

Então a planilha recebe `100 - m`: exatamente o quanto o aluno precisa tirar no
exame para fechar em 50. É uma linha de código que carrega a álgebra toda — e é o
que separa este notebook de um `if` que chuta valores.

## Como rodar

1. Abra `manipulating_spreadsheet.ipynb` no Google Colab.
2. Ajuste o nome da planilha na chamada `gc.open(...)` para a sua, com o mesmo layout
   de colunas (faltas em `C`, provas em `D`–`F`, situação em `G`, nota final em `H`;
   alunos a partir da linha 4).
3. Execute as células — a autenticação usa a credencial do ambiente Colab, sem
   arquivo de chave.

## Limitações conhecidas

- **Uma escrita de célula por vez.** O notebook chama `page.update` célula a célula,
  dentro do laço — em planilha grande isso é lento e gasta cota da API do Sheets.
  Um `batch_update` faria o mesmo em muito menos chamadas.
- **Layout fixo.** As colunas e a faixa de linhas dos alunos estão embutidas no
  código; planilha com outro formato exige editar o notebook.
- **Sem resultado versionado.** Não é um projeto de métrica: as saídas do laço
  principal foram limpas, e o repositório não guarda nenhum número — só o código da
  regra.

## Estrutura

```
.
├── manipulating_spreadsheet.ipynb   # lê, aplica as regras e escreve de volta
├── LICENSE                          # MIT
└── README.md
```
