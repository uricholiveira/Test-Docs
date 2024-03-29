# Carta espelho

## Resumo dos campos

- **Quantidade fornecida**: Alterar/criar para ter duas opções (Fluxômetro/Trena)
- **Adicionar campos:**
    - Fluxômetro inicial
    - Fluxômetro final
    - Booleano para a pergunta (Houve volume retido no mangote?)
    - Lista de mangotes?
- **Remover obrigatoriedade:**
    - Quantidade Solicitada em Toneladas

## Campos automáticos

Os campos automáticos serão preenchidos pelo backend através das fórmulas descritas abaixo.

- **Quantidade Fornecida:** ``Será atualizado para um campo Select (Fluxômetro, Trena)``

> Importante acrescentar: Fluxometro inicial / Fluxometro final <br/>
> E a pergunta: Se houve volume retido no mangote? Sim ou não. <br/>
> Se a resposta for "não": 0 volume retido <br/>
> Se a resposta for "Sim" acrescentar outra pergunta: Quantos mangotes conectados? <br/>
> - Volume por mangote de 4": 0,102 Tons
> - Volume por mangote de 6": 0,229 Tons

- **Toneladas:**

<tabs>
<tab title="Fluxometro">

> É a diferença entre o (Fluxometro final - Fluxometro inicial) <format color="IndianRed">- Se houve produto retido no
> mangote.</format>
</tab>
<tab title="Trena">

> É a diferença entre (massa total inicial "Tons" - massa total final) <format color="IndianRed">- Se houve produto
> retido no mangote (Tons)</format>
</tab>
</tabs>

- **Litros A20:**

<tabs>
<tab title="Fluxometro">
Toneladas - Densidade A20

> O Resultado da fórmula anterior, subtraído pela densidade a 20º.
</tab>
<tab title="Trena">
VolA20

> É a diferença entre <format color="IndianRed">(Vol a20 total inicial - Vol a20 total final) - Se houve produto retido
> no mangote (A20) </format>.
</tab>
</tabs>

- **Volume ambiente:**

<tabs>
<tab title="Fluxometro">
Litros A20 - <format color="IndianRed">Fator de Correção</format>

> O resultado da fórmula anterior, subtraído pelo fator de correção.
</tab>
<tab title="Trena">
Toneladas - Densidade A20

> É a diferença entre <format color="IndianRed">(Vol . amb total inicial - Vol amb. total final) - Se houve produto
> retido no mangote (Vol. amb.)</format>
</tab>
</tabs>

- **Temperatura (Média dos tanques):**
  Soma do Volume dos tanques / Soma da temperatura dos tanques

> Arredondar resultado para 0, ou 5

![temperatura_media_tanques.png](temperatura_media_tanques.png)

- **Volume A20:**
  Quantidade Solicitada em Toneladas - Densidade A20

- **Volume Ambiente:**
  Quantidade Solicitada em Volume A20 - <format color="IndianRed">Fator de Correção</format>

- **Vazão valor:**
  Quantidade Fornecida (Fluxo ou Trena) em Volume.Ambiente - <format color="IndianRed">Tempo operacional (Diferença
  entre hora de início e término da operação)</format>

- **Porcentagem:**
  (Vazão valor - Vazão Solicitada pelo Navio) * 100

- **Fluxômetro:**
  Quantidade Fornecida em Toneladas (Por Fluxômetro)

- **Trena:**
  Quantidade Fornecida em Toneladas (Por Trena)

- **Diferença:**
  Quantidade Fornecida em Toneladas (Por Fluxômetro) - Quantidade Fornecida em Toneladas (Por Trena)

- **Percentual:**
  (Diferença / Fluxometro) * 100
> Alerta para diferenças iguais ou superiores a -0,50%
> 
> Cor vermelha para diferenças negativas.

## Dúvidas

1. 