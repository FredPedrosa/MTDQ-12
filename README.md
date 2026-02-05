# Escala MTDQ-12: Modelo Psicométrico e Ferramentas de Escoragem

Este repositório contém os recursos técnicos e a sintaxe de cálculo para a **Escala de Avaliação dos Efeitos da Musicoterapia em Grupo na Dependência Química (MTDQ-12)**. 

A MTDQ-12 é uma medida fundamental, unidimensional e invariante, desenvolvida para mensurar a percepção de benefícios de intervenções musicoterapêuticas sistematizadas. Esta versão de 12 itens foi refinada e validada através de modelagem Rasch (**Partial Credit Model - PCM**), garantindo propriedades métricas superiores para uso clínico e em pesquisa.

O modelo foi desenvolvido por:
> Pedrosa, F. G., & Gomes, C. M. A. (2025). *Refinamento de uma medida para a musicoterapia com pessoas com transtorno por uso de substâncias*. [_No prelo_].

---

## 📂 Estrutura do Repositório

*   `/parametros`: Arquivo `parametros_eta_MTDQ12.rds` contendo os parâmetros de dificuldade (*eta*) dos itens calibrados via CML.
*   `/sintaxe`: Script R para estimativa de escores (Thetas) em novos bancos de dados.

---

## ⚙️ Como Estimar os Escores (Habilidade Rasch)

Para utilizar a MTDQ-12 como uma régua de medida em novos estudos, é necessário transformar os escores brutos em medidas intervalares (Logits) utilizando os parâmetros fixos deste estudo.

### 1. Preparação e Recodificação
O modelo Rasch foi calibrado colapsando as categorias originais de 5 pontos para **4 pontos (0 a 3)** para corrigir desordens de limiares. Antes de rodar a sintaxe, você deve recodificar seus dados:

| Resposta Original (Likert 1-5) | Valor para o Modelo (Rasch 0-3) |
| :--- | :---: |
| 1 ("Nunca") ou 2 ("Raramente") | **0** |
| 3 ("Às vezes") | **1** |
| 4 ("Muitas vezes") | **2** |
| 5 ("Sempre") | **3** |

### 2. Sintaxe no R (Utilizando o pacote `eRm`)

A utilização do pacote `eRm` é fundamental por utilizar a **Máxima Verossimilhança Condicional (CML)**.

```R
# Instalação do pacote se necessário
# install.packages("eRm")
library(eRm)

# 1. Carregar seus dados recodificados (escala 0-3)
# O dataframe deve conter as 12 colunas na ordem: 
# i3, i5, i6, i7, i9, i10, i11, i13, i14, i16, i18, i20
meus_dados <- read.csv("meus_dados_recodificados.csv")

# 2. Carregar os parâmetros eta (.rds) calibrados no estudo de Pedrosa & Gomes (2025)
# O arquivo .rds garante a integridade dos parâmetros salvos
parametros_fixos <- readRDS("parametros_eta_MTDQ12.rds")

# 3. Estimar os parâmetros de pessoa (Theta) usando parâmetros de item fixos
# O parâmetro item_params deve receber o vetor de etas carregado
res_pp <- person.parameter(res = NULL, X = meus_dados, item_params = parametros_fixos)

# 4. Extrair os escores individuais em Logits
escores_participantes <- res_pp$theta.table$`Person Parameter`

# 5. Adicionar os escores ao seu banco de dados para análise clínica
meus_dados$theta_mtdq12 <- escores_participantes
```

## 🧠 Propriedades da MTDQ-12

Esta versão otimizada apresenta:
1. Confiabilidade de Separação de Rasch: 0.82 (Capacidade de distinguir diferentes níveis de percepção de benefício).
2. Invariância da Medida: Estabilidade dos parâmetros entre sexos e níveis de traço latente.
3. Alta Validade Concorrente: Correlação de r = 0,96 com os escores fatoriais da escala original de 20 itens, garantindo a mesma precisão diagnóstica com maior parcimônia.

## 📄 Citação Recomendada

Para utilizar a escala ou estes scripts, cite o artigo de referência:

> Pedrosa, F. G., & Gomes, C. M. A. (2025). Refinamento de uma medida para a musicoterapia com pessoas com transtorno por uso de substâncias. [No prelo].

## ⚖️ Licença

Este projeto está licenciado sob a **Creative Commons Attribution-NonCommercial 4.0 International (CC BY-NC 4.0)**.

*   **Uso Permitido:** O uso para fins de pesquisa acadêmica, científica e prática clínica é livre, desde que citada a fonte original.
*   **Uso Comercial:** O uso para fins comerciais, lucrativos ou venda de serviços baseados nesta ferramenta é terminantemente proibido sem a autorização expressa, por escrito, dos autores.

Para detalhes completos da licença, visite [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/deed.pt-br).
