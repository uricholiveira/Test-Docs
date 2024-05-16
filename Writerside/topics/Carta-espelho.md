# Carta espelho

## Campos automáticos

### Temperatura:
> Sempre arredondar resultado para 0, ou 5

![temperatura_media_tanques.png](temperatura_media_tanques.png)
<tabs>
<tab title="Fluxometro">

>  Somar Volume Ambiente, e Temperatura apenas das **Condições Inicias**, e dividí-las (Volume Ambiente / Temperatura)

</tab>
<tab title="Trena">

>  Somar Volume Ambiente, e Temperatura de ambas as condições (inicias/finais), e dividí-las (Volume Ambiente / Temperatura)

</tab>
</tabs>

### Fator de Correção

> Com base na Densidade20, e com a Temperatura (Média dos Tanques), realizar busca na tabela de fator da embarcação.

### Toneladas

<tabs>
<tab title="Fluxometro">

> É a diferença entre o (Fluxometro final - Fluxometro inicial) <format color="IndianRed">- Se houve produto retido no
> mangote.</format>

<br />

```Javascript
let mirror.supply.initialFlowmeter = 3
let mirror.supply.finalFlowmeter = 6
let mirror.supply.hoses = [{..., quantity: 1}, {..., quantity: 1}]

let result = (mirror.supply.finalFlowrate - 
              mirror.supply.initialFlowrate) - 
              mirror.supply.hoses.sum(x => x.quantity);

console.log(result) // (5 - 3) - 2 = 1
```

</tab>
<tab title="Trena">

> É a diferença entre (massa total inicial "Tons" - massa total final) <format color="IndianRed">- Se houve produto
> retido no mangote (Tons)</format>

<br/>

```Javascript
let mirror.conditions = [{..., finalTons: 1}, 
                                {..., finalTons: 3}]
let mirror.supply.hoses = [{..., quantity: 1}, {..., quantity: 1}]

let result = mirror.conditions.sum(x => x.finalTons) -  
             mirror.supply.hoses.sum(x => x.quantity);

console.log(result) // 4 - 1 = 3
```

</tab>
</tabs>

### Litros A20

<tabs>
<tab title="Fluxometro">
Toneladas - Densidade A20

> O Resultado da fórmula anterior, subtraído pela densidade a 20º.

<br/>

```Javascript
let mirror.supply.tons = 5
let mirror.supply.density20 = 3

let result = mirror.supply.tons - mirror.supply.density20

console.log(result) // 5 - 3 = 2
```

</tab>
<tab title="Trena">
VolA20

> É a diferença entre <format color="IndianRed">(Vol a20 total inicial - Vol a20 total final) - Se houve produto retido
> no mangote (A20) </format>.

<br/>

```Javascript
// type: 1 = Inicial
// type: 2 = Final
let mirror.conditions.mensurations = 
                              [
                                {..., type: 1, environment20: 4}, 
                                {..., type: 2, environment20: 1}
                              ]
let mirror.supply.hoses = [{..., quantity: 1}, {..., quantity: 1}]

let result = (mirror.conditions.mensurations
              .filter(x => x.type === 1)
              .sum(x => x.environment20) -  
              mirror.conditions.mensurations
              .filter(x => x.type === 2)
              .sum(x => x.environment20)
              ) -
             mirror.Supply.Hoses.sum(x => x.quantity);

console.log(result) // (4 - 1) - 2 = 1
```

</tab>
</tabs>

### Volume ambiente

<tabs>
<tab title="Fluxometro">

> Litros A20 - <format color="IndianRed">Fator de Correção</format>

</tab>
<tab title="Trena">
Toneladas - Densidade A20

- Vol . amb total inicial = Soma do Volume Ambiente das Condições Iniciais
- Vol . amb total final = Soma do Volume Ambiente das Condições Finais

> É a diferença entre <format color="IndianRed">(Vol . amb total inicial - Vol amb. total final) - Quantidade Retida do Mangote</format>
</tab>
</tabs>

### Trim (Da Condição)
  Calado Proa - Calado Popa

### Fator (Do tanque)
  Com base na Altura e no Trim, buscar o Fator na tabela **Tonnage** da Capacidade do Tanque.

### Volume A20 (Do tanque)
  Volume Ambiente (Da Quantidade Fornecida) x Fator (Do tanque)

### Volume Ambiente (Do tanque)
  Volume A20 (Do Tanque) - Fator (Do tanque)

### Tonelada (Do tanque)
  Volume A20 (Do Tanque) x Densidade (Do tanque)

### Tempo Operacional:
Esse campo não é apresentado na tela, porém é usado no cálculo abaixo:

= (Hora inicial - final) em horas

### Vazão:
<tabs>
<tab title="Fluxometro">

> Volume Ambiente (Quantidade Fornecida) - <format color="IndianRed">Tempo Operacional</format>

</tab>
<tab title="Trena">

> Volume Ambiente Final (Quantidade Solicitada/Por Trena) - <format color="IndianRed">Tempo Operacional</format>

</tab>
</tabs>

### Porcentagem 
(Vazão - Vazão Solicitada pelo Navio) * 100

### Fluxômetro
Toneladas (Quantidade Fornecida)

### Trena
Toneladas Final (Quantidade Solicitada/Por Trena)

### Diferença
Fluxômetro - Trena

### Percentual
(Diferença / Fluxometro) * 100