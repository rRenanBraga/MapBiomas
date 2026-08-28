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
