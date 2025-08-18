# Modelo Psicométrico da Escala MTDQ-18

Este repositório contém os recursos para a **Escala de Avaliação dos Efeitos da Musicoterapia em Grupo na Dependência Química (MTDQ-18)**, uma versão otimizada e psicometricamente robusta de 18 itens.

O modelo foi desenvolvido e validado no artigo:
> *Artigo em processo de revisão por pares (blind review).*

O objetivo deste repositório é promover a ciência aberta, permitindo que pesquisadores e clínicos utilizem o modelo pré-calibrado para estimar os escores dos participantes de forma padronizada e confiável.

---

## Conteúdo do Repositório

*   **/modelo/**
    *   `modelo_MTDQ18_GRM.rds`: O objeto `mirt` contendo o modelo final calibrado da Teoria de Resposta ao Item (TRI).
*   **/artigo/**
    *   `artigo_validacao_mtdq18.pdf`: O manuscrito completo descrevendo o processo de validação e os resultados psicométricos.
*   **/sintaxe/**
    *   `sintaxe_analise_completa.Rmd`: Arquivo R Markdown com a sintaxe completa utilizada para todas as análises de validação.
*   `README.md`: Este arquivo de instruções.

---

## Como Usar o Modelo para Estimar Escores

Siga os passos abaixo para carregar o modelo pré-calibrado e calcular os escores para um novo conjunto de dados.

### 1. Pré-requisitos

Você precisará do R e dos pacotes `mirt` e `dplyr` instalados.

```R
# Se não tiver os pacotes, instale-os primeiro
# install.packages(c("mirt", "dplyr"))

library(mirt)
library(dplyr)
```

### 2. Carregar o Modelo
Faça o download do arquivo modelo_MTDQ18_GRM.rds (da pasta /modelo/) para o seu diretório de trabalho e use o seguinte comando para carregá-lo no R.

```R
# Certifique-se de que o caminho para o arquivo está correto
modelo_mtdq18 <- readRDS("modelo/modelo_MTDQ18_GRM.rds")
```
### 3. Preparar seus Dados

Seus dados devem ser um data.frame ou matrix com 18 colunas, uma para cada item da MTDQ-18.
Importante: A escala original possui 5 pontos (1 a 5), mas nosso modelo foi otimizado para 4 categorias. Você precisa recodificar suas respostas antes de estimar os escores.
Resposta Original (5 pontos)	Valor Recodificado (4 pontos):
1 ("Nunca")	1
2 ("Raramente")	1
3 ("Às vezes")	2
4 ("Muitas vezes")	3
5 ("Sempre")	4

Aqui está uma função para facilitar a recodificação:
```R
# --- Exemplo com dados fictícios ---

# Nomes dos 18 itens do modelo (certifique-se que seus dados tenham esses nomes)
nomes_itens <- c("i1", "i2", "i3", "i4", "i5", "i6", "i7", "i9", "i10", 
                 "i11", "i13", "i14", "i15", "i16", "i17", "i18", "i19", "i20")

# Crie um data.frame de exemplo com respostas brutas (1-5) de 3 participantes
dados_brutos_exemplo <- data.frame(
  i1=c(1,3,5), i2=c(2,4,4), i3=c(1,2,3), i4=c(3,3,4), i5=c(2,5,5), 
  i6=c(4,4,5), i7=c(1,2,4), i9=c(2,3,3), i10=c(3,4,5), i11=c(2,3,4),
  i13=c(4,5,5), i14=c(3,4,5), i15=c(4,5,5), i16=c(5,5,5), i17=c(4,4,5),
  i18=c(5,5,5), i19=c(4,5,5), i20=c(3,3,4)
)

# Função para recodificar os dados
recodificar_mtdq <- function(dados){
  dados_recodificados <- dados %>%
    mutate(across(everything(), ~case_when(
      .x %in% c(1, 2) ~ 1,
      .x == 3 ~ 2,
      .x == 4 ~ 3,
      .x == 5 ~ 4,
      TRUE ~ NA_real_ # Mantém NAs, se houver
    )))
  return(dados_recodificados)
}

# Aplique a recodificação aos seus dados
# (Aqui usamos os dados de exemplo)
seus_dados_prontos <- recodificar_mtdq(dados_brutos_exemplo)
```
### 4. Estimar os Escores (Theta)

Use a função fscores() do pacote mirt para calcular o escore do traço latente (theta) para cada participante.

```R
# Calcule os escores usando o método EAP (valor padrão e recomendado)
escores_participantes <- fscores(modelo_mtdq18, response.pattern = seus_dados_prontos)

# Veja os resultados (uma matriz com uma coluna "F1" para o traço latente)
print(escores_participantes)
```
### Citação

Se você utilizar este modelo ou código em sua pesquisa, por favor, cite da seguinte forma:

Pedrosa, F. G. (2025). MTDQ-18. [Software]. Recuperado de https://github.com/FredPedrosa/song_scores/

E, quando publicado, por favor, cite também o artigo de validação associado.

_____

## Licença

Este projeto está licenciado sob uma versão modificada da GNU General Public License v3.0. O uso comercial não é permitido sem a permissão explícita e por escrito do autor. Para mais detalhes, consulte o arquivo de licença.
