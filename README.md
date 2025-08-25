# Modelo Psicométrico e Ferramentas de Escoragem para a Escala MTDQ-12

Este repositório contém os recursos essenciais para a **Escala de Avaliação dos Efeitos da Musicoterapia em Grupo na Dependência Química (MTDQ-12)**, uma versão de 12 itens refinada e validada utilizando modelos da família Rasch.

O modelo e seus parâmetros foram desenvolvidos e validados no artigo:
> Pedrosa, F. G., & Gomes, C. M. A. (2025). *Refinamento de uma medida para a musicoterapia com pessoas com transtorno por uso de substâncias*. [_No prelo_].

O objetivo deste repositório é promover a ciência aberta, permitindo que pesquisadores e clínicos utilizem os parâmetros de item p

---

## Conteúdo do Repositório

*   `instrumento/`
    *   `MTDQ-12.pdf`: A versão final da escala com os 12 itens.
*   `parametros/`
    *   `parametros_eta_MTDQ12.rds`: Um vetor R contendo os 36 parâmetros de dificuldade de item (*eta*) do modelo final *Partial Credit Model* (PCM). Estes são os únicos valores necessários para calcular os escores.
*   `sintaxe/`
    *   `sintaxe_calculo_escores.R`: Um script R com a sintaxe completa e comentada para calcular os escores a partir de novos dados.
*   `README.md`: Este arquivo de instruções.

---

## Como Usar os Parâmetros para Estimar Escores

Siga os passos abaixo para carregar os parâmetros pré-calibrados e calcular os escores para um novo conjunto de dados.

### 1. Pré-requisitos

Você precisará do R e dos pacotes `TAM` e `dplyr` instalados.

```R
# Se não tiver os pacotes, instale-os primeiro
# install.packages(c("TAM", "dplyr"))

library(TAM)
library(dplyr)
```

### 2. Carregar os Parâmetros do Modelo

Faça o download do arquivo parametros_eta_MTDQ12.rds para o seu diretório de trabalho e use o seguinte comando para carregá-lo no R.

```R
# Certifique-se de que o caminho para o arquivo está correto
parametros_itens_fixos <- readRDS("parametros/parametros_eta_MTDQ12.rds")
```
### 3. Preparar seus Dados

Seus dados devem ser um data.frame ou matrix com 12 colunas, uma para cada item da MTDQ-12.
Importante: A escala original possui 5 pontos (1 a 5). O modelo Rasch foi calibrado com respostas numéricas que vão de 0 a 3. Você precisa recodificar suas respostas antes de estimar os escores, conforme a tabela abaixo:
Resposta Original (1-5)	           Valor Recodificado (0-3)
1 ("Nunca") ou 2 ("Raramente")	  0
3 ("Às vezes")	                    1
4 ("Muitas vezes")	              2
5 ("Sempre")	                    3

Aqui está um exemplo completo, desde dados fictícios até o cálculo dos escores.

```R
 --- Exemplo com dados fictícios ---

# Nomes dos 12 itens do modelo final, na ordem correta
nomes_dos_12_itens <- c("i3", "i5", "i6", "i7", "i9", "i10", 
                        "i11", "i13", "i14", "i16", "i18", "i20")

# Crie um data.frame de exemplo com respostas brutas (1-5) de 3 participantes
dados_brutos_exemplo <- data.frame(
  i3 = c(1, 4, 5), i5 = c(2, 3, 5), i6 = c(1, 5, 5),
  i7 = c(2, 2, 4), i9 = c(3, 4, 4), i10 = c(1, 3, 5),
  i11 = c(2, 4, 4), i13 = c(3, 5, 5), i14 = c(4, 4, 3),
  i16 = c(1, 2, 5), i18 = c(5, 5, 4), i20 = c(3, 4, 4)
)

# Função para recodificar os dados para o formato numérico (0 a 3)
recodificar_para_rasch <- function(dados){
  dados_recodificados <- dados %>%
    mutate(across(everything(), ~case_when(
      .x %in% c(1, 2) ~ 0,
      .x == 3 ~ 1,
      .x == 4 ~ 2,
      .x == 5 ~ 3,
      TRUE ~ NA_real_ # Mantém NAs
    )))
  return(dados_recodificados)
}

# Aplique a recodificação aos seus dados
dados_prontos_para_analise <- recodificar_para_rasch(dados_brutos_exemplo)

# Verifique se os nomes e a ordem das colunas correspondem aos do modelo
dados_prontos_para_analise <- dados_prontos_para_analise[, nomes_dos_12_itens]
```
### 4. Estimar os Escores (Theta)

Agora, usaremos a função tam.mml() do pacote TAM, fornecendo os parâmetros dos itens como fixos. Isso garante que os escores de habilidade (theta) sejam calculados na mesma escala do estudo de validação.

```R
# Criar a estrutura necessária para o TAM (matriz de design)
# Esta etapa garante que o modelo interprete corretamente os parâmetros fixos
dados_molde <- as.data.frame(matrix(0:3, nrow = 4, ncol = length(nomes_dos_12_itens)))
colnames(dados_molde) <- nomes_dos_12_itens
design_pcm <- TAM::designMatrices(resp = dados_molde, modeltype = "PCM")

# Criar a matriz de parâmetros fixos no formato que o TAM espera
matriz_parametros_fixos <- cbind(1:length(parametros_itens_fixos), parametros_itens_fixos)

# Estimar o modelo apenas para obter os escores das pessoas
modelo_com_escores <- TAM::tam.mml(
  resp = dados_prontos_para_analise, 
  A = design_pcm$A, 
  xsi.fixed = matriz_parametros_fixos,
  verbose = FALSE
)

# Extrair os escores de habilidade (EAP - Expected A Posteriori)
escores_participantes <- modelo_com_escores$person$EAP

# Adicionar os escores ao seu data.frame original para análise
dados_brutos_exemplo$theta_mtdq12 <- escores_participantes

# Veja o resultado final, agora com os escores calculados
print(dados_brutos_exemplo)
```
### Citação

Se você utilizar este modelo ou código em sua pesquisa, por favor, cite da seguinte forma:

> Pedrosa, F. G., & Gomes, C. M. A. (2025). Refinamento de uma medida para a musicoterapia com pessoas com transtorno por uso de substâncias. [_No prelo_].

E, quando publicado, por favor, cite também o artigo de validação associado.

Para o repositório em si, pode-se usar:

> Pedrosa, F. G., & Gomes, C. M. A. (2024). Modelo Psicométrico e Ferramentas de Escoragem para a Escala MTDQ-12 [Software]. https://github.com/FredPedrosa/MTDQ-12

_____

## Licença

Este projeto está licenciado sob uma versão modificada da GNU General Public License v3.0. O uso comercial não é permitido sem a permissão explícita e por escrito do autor. Para mais detalhes, consulte o arquivo de licença.
