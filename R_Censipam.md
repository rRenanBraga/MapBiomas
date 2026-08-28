# Gráfico de linha - comparação do PIB


library(readxl)
library(tidyr)
library(dplyr)
pib <- read_excel("C:/Users/renan.reis/Downloads/pib.xlsx")
View(pib)

pib <- pib %>%
  mutate(
    ANO = as.integer(ANO),
    VALOR = as.numeric(
      gsub("\\.", "", gsub(",", ".", VALOR))
    )
  )

top5 <- pib %>%
  filter(ANO == 2021) %>%
  arrange(desc(VALOR)) %>%
  slice(1:5) %>%
  pull(NM_MUN)

top5


library(ggplot2)
library(scales)

ggplot(pib_top5, aes(x = ANO, y = VALOR, color = NM_MUN)) +
  geom_line(linewidth = 1.2) +
  geom_point(size = 2) +
  scale_x_continuous(breaks = 2011:2021) +
  scale_y_continuous(
    labels = scales::label_number(scale = 1e-6, suffix = " mi")
  ) +
  labs(
    title = "Comparação dos PIB dos Municípios da Faixa de Fronteira do Estado de Rondônia (2011-2021",
    x = "Ano",
    y = "PIB (R$ milhões)",
    color = "Município"
  ) +
  theme_minimal() +
  theme(
    plot.title = element_text(face = "bold", size = 16),
    axis.text.x = element_text(angle = 45, hjust = 1, size = 11),
    axis.text.y = element_text(size = 15),
    legend.title = element_text(size = 15, face = "bold"),
    legend.text = element_text(size = 15),
    legend.position = "bottom"
  )



2


library(readxl)
teste <- read_excel("C:/Users/renan.reis/Downloads/teste.xlsx")
View(teste)
 
# Gráfico simples, PLOT - Utilizado para números, caractes não são permitidos
 
plot(x = teste$SOMA_KM,
     pch = 16,
     col = "#2E9FDF",
     xlab = "soma",
     main = "Gráfico Simples")
 
# Gráfico com presença de texto
 
boxplot(SOMA_KM ~ VEGETACAO, data = teste,
xlab = "vegetação", ylab = "SOMA",
main = "Gráfico 2")
 
# BLOCKPLOT é uma representação gráfica que mostra a distribuição de dados espe
#cialmente útil para analisar e comparar diferentes grupos ou categorias de dados.
 
b <- boxplot(teste$SOMA_KM,
col = "lightblue",
pch = 16,
ylab = "SOMA KM",
main = "Gráfico de bloxplot"
)



3



library(readxl)
library(tidyr)
library(dplyr)
getwd()
setwd("C:/Users/felippe.lima/Documents/analise_estatistica")
dados <- read.csv("Biomas_agoravai.csv")
dados_duplicados <- dados %>%
  group_by(id) %>%
  filter(n() > 1) %>%
  ungroup()
##12.539 - Excluir dados duplicados
dados_unicos <- dados %>%
  group_by(id) %>%
  summarise(area_total = sum(area_km2, na.rm = TRUE),
            across(everything(), ~ first(.x))  
  ) %>%
  ungroup()
dados_unicos$persistencia_dias <- dados_unicos$persistencia_dias+1
biom_cerrado <- dados_unicos[dados_unicos$bioma == "Cerrado",]
 
variaveis <- biom_cerrado %>%
  select(area_total, media_frp, persistencia_dias)
 
variaveis_scaled <- scale(variaveis)
 
library(factoextra)
 
set.seed(123)
km <- kmeans(variaveis_scaled, centers = 6, nstart = 25)
biom_cerrado$cluster <- km$cluster
 
table(biom_cerrado$cluster)
 
 
biom_cerrado %>%
  group_by(cluster) %>%
  summarise(
    area_media = mean(area_total, na.rm = TRUE),
    frp_medio = mean(media_frp, na.rm = TRUE),
    persistencia_media = mean(persistencia_dias, na.rm = TRUE),
    n = n()
  )
 
##Tipologia sugerida
 
##Cluster 4: Eventos pequenos e típicos (curtos, baixa intensidade)
 
#Cluster 6: Eventos persistentes (pequenos/médios, duradouros)
 
#Cluster 2: Eventos pequenos e intensos
 
#Cluster 3: Eventos explosivos (curtos, muito intensos)
 
#Cluster 5: Grandes incêndios persistentes
 
#Cluster 1: Megaincêndios raros
 
biom_cerrado <- biom_cerrado %>%
  mutate(tipologia = case_when(
    cluster == 4 ~ "Eventos pequenos e típicos",
    cluster == 6 ~ "Eventos persistentes",
    cluster == 2 ~ "Eventos pequenos e intensos",
    cluster == 3 ~ "Eventos explosivos",
    cluster == 5 ~ "Grandes incêndios persistentes",
    cluster == 1 ~ "Megaincêndios raros",
    TRUE ~ "Não classificado"
  ))


  4


library(DBI)

library(RPostgres)

library(dplyr)
 
 
con <- dbConnect(

  RPostgres::Postgres(),

  dbname = "sig_sipam",

  host = "172.23.5.229",   # ou IP do servidor

  port = 5432,

  user = "renan.reis",

  password = "42041940Rbr?"

)
 
 
query <- "

WITH mes_evento AS (

  SELECT

    mv.id_evento,

    date_trunc('month', mv.dt_passagem)::date AS mes,

    MAX(mv.area_acumulada_ha) AS area_cumul_ha

  FROM queimadas.mv_indicadores_queimadas mv

  JOIN ibge_bc250_2021.lml_unidade_federacao_a uf

    ON uf.sigla = 'RO'

   AND ST_Intersects(uf.geom, mv.geom_acumulada)

  WHERE mv.dt_passagem >= DATE '2024-01-01'

    AND mv.dt_passagem <  DATE '2025-11-01'

  GROUP BY mv.id_evento, date_trunc('month', mv.dt_passagem)

)

SELECT * 

FROM mes_evento

ORDER BY mes, id_evento

"
 
 
dados_ro <- dbGetQuery(con, query)
 
library(ggplot2)

library(scales)
 
dados_ro$mes <- as.Date(dados_ro$mes)
 
dados_mensal <- dados_ro %>%

  group_by(mes) %>%

  summarise(

    area_total_cumul_ha = sum(area_cumul_ha, na.rm = TRUE),

    qtd_eventos_no_mes = n_distinct(id_evento)

  ) %>%

  arrange(mes)  
 
# Gráfico 1: área total por mês
 
g1 <- ggplot(dados_mensal, aes(x = mes, y = area_total_cumul_ha)) +

  geom_col(fill = "#0072B2") +

  scale_x_date(date_labels = "%b/%Y", date_breaks = "1 month") +

  labs(

    title = "Área total queimada por mês - Rondônia (2024-2025)",

    x = "Mês",

    y = "Área acumulada (ha)"

  ) +

  theme_minimal(base_size = 13) +

  theme(axis.text.x = element_text(angle = 45, hjust = 1))
 
print(g1)
 
 
# Gráfico 2: quantidade de eventos por mês
 
g2 <- ggplot(dados_mensal, aes(x = mes, y = qtd_eventos_no_mes)) +

  geom_col() +

  scale_x_date(date_labels = "%b/%Y", date_breaks = "1 month") +

  labs(

    title = "Quantidade de eventos de queimadas por mês - Rondônia (2024-2025)",

    x = "Mês",

    y = "Número de eventos"

  ) +

  theme_minimal(base_size = 13) +

  theme(axis.text.x = element_text(angle = 45, hjust = 1))
 
# imprimir g2

print(g2)

 
declaração fins.pdf




5 - gráfico de barra



# Carregar os pacotes necessários
library(readxl)
library(ggplot2)
library(dplyr)
library(tidyr)
library(scales) # Para formatar números
 
# Ler o arquivo Excel
TABELA_VALOR_CELIBRADO <- read_excel("C:/Users/renan.reis/Downloads/TABELA_VALOR_CELIBRADO.xlsx")
 
# Verificar os nomes das colunas
names(TABELA_VALOR_CELIBRADO)
 
# Filtrar dados
dados_limpos <- TABELA_VALOR_CELIBRADO %>%
  filter(!is.na(Município) & !is.na(`Entidade Vinculada`) & !is.na(`Valor Celebrado`))
 
# OPÇÃO 1: Mostrar com 2 casas decimais (recomendado)
formatar_valor <- function(valor) {
  if (valor >= 1e6) {
    return(paste0(round(valor / 1e6, 1), "M"))
  } else if (valor >= 1e3) {
    return(paste0(round(valor / 1e3, 1), "K"))
  } else {
    return(as.character(round(valor, 0)))
  }
}
 
# Cores altamente contrastantes
cores_contraste <- c(
  "#E6194B", "#3CB44B", "#FFE119", "#4363D8", "#F58231",
  "#911EB4", "#42D4F4", "#F032E6", "#BFEF45", "#469990",
  "#DCBEFF", "#9A6324", "#FFFAC8", "#800000", "#AAFFC3"
)
 
ggplot(dados_limpos, aes(x = Município, y = `Valor Celebrado`, fill = `Entidade Vinculada`)) +
  geom_bar(stat = "identity", position = position_dodge(width = 0.8), width = 0.7) +
  geom_text(aes(label = sapply(`Valor Celebrado`, formatar_valor)), 
            position = position_dodge(width = 0.8),
            vjust = -0.5, size = 2.8, fontface = "bold") +
  scale_y_continuous(labels = function(x) {
    ifelse(x >= 1e6, paste0(x/1e6, "M"), 
           ifelse(x >= 1e3, paste0(x/1e3, "K"), x))
  }) +
  scale_fill_manual(values = cores_contraste) +
  labs(
    title = "Investimentos Federais Celebrados entre 5 Municípios da Faixa de Fronteira do Estado Rondônia",
    x = "Município",
    y = "Valor Celebrado (R$)",
    fill = "Órgão Superior"
  ) +
  theme_minimal() +
  theme(
    plot.title = element_text(hjust = 1.0, size = 20, face = "bold"),
    axis.text.x = element_text(angle = 45, hjust = 1, size = 10),
    axis.title = element_text(size = 12, face = "bold"),
    legend.title = element_text(face = "bold"),
    legend.text = element_text(size = 10)
  )




  



