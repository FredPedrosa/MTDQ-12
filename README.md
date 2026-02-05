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
if(!require(PP)) install.packages("PP"); library(PP)

# 1. Base simulada de dados de teste (4 participantes, 12 itens)
set.seed(123)
dados_teste <- as.data.frame(matrix(sample(0:3, 48, replace = TRUE), nrow = 4, ncol = 12))
colnames(dados_teste) <- c("i3", "i5", "i6", "i7", "i9", "i10", "i11", "i13", "i14", "i16", "i18", "i20")

# 2. Carregar os parâmetros (agora como matriz de thresholds)
# Se você ainda não salvou como matriz, vamos converter o seu RDS de etas atual:
# (Supondo que parametros_fixos seja o seu vetor de etas do RDS anterior)
# Se você já salvou como matriz no passo 1 acima, ignore este 'if'
if(is.vector(parametros_fixos)) {
  # Converte vetor de etas para matriz de thresholds que o PP exige
  # (Considerando 12 itens e 3 limiares por item)
  pre_matrix <- matrix(thresholds(pcm_model)$threshtable[[1]][, -1], ncol = 3)
} else {
  pre_matrix <- parametros_fixos[, -1] # Remove a coluna 'Location' se ela existir
}


# 2. Carregar os parâmetros calibrados (RDS)
par_fixos <- readRDS("parametros_eta_MTDQ12.rds")

# 3. Formatar matriz de parâmetros (3 limiares x 12 itens)
matriz_par <- matrix(par_fixos, nrow = 3, ncol = 12, byrow = FALSE)

# 4. Estimar as habilidades (Thetas)
# O uso de slopes = 1 mantém as propriedades do modelo Rasch (PCM)
estimativa <- PP::PP_gpcm(respm = as.matrix(dados_teste), 
                          thres = matriz_par, 
                          slopes = rep(1, 12), 
                          type = "mle")

# 5. Salvar escores no banco de dados
dados_teste$escore_rasch <- estimativa$resNP$resMLE

# 5. Extrair os escores corretamente
# No objeto GPCM, os resultados ficam em calculo$resPP$resPP
escores_rasch <- calculo$resPP$resPP[, "estimate"]
erros_padrao  <- calculo$resPP$resPP[, "SE"]

cat("\nEscores Rasch (Logits):\n")
print(escores_rasch)
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
