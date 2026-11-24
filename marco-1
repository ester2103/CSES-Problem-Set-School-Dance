# Marco 1 — Formulação

**Problema:** Baile Escolar (emparelhamento máximo em grafo bipartido)

## 1. Resumo do problema

Uma escola tem `n` meninos e `m` meninas. Há `k` pares potenciais, ou seja, pares (menino, menina) dispostos a dançar juntos. Cada pessoa pode participar de no máximo uma dupla. O objetivo é formar o maior número possível de duplas e mostrar quais são elas.

## 2. Entrada, saída, objetivo e restrições

| Item | Descrição |
|---|---|
| **Entrada** | Primeira linha: `n`, `m`, `k`. Depois, `k` linhas com `a b`, indicando que o menino `a` e a menina `b` aceitam dançar juntos. |
| **Saída** | Um inteiro `r` (número máximo de duplas), seguido de `r` linhas `a b` com as duplas formadas. Qualquer solução válida é aceita. |
| **Objetivo** | Maximizar `r`, sem repetir nenhum menino nem nenhuma menina. |
| **Restrições** | 1 ≤ n, m ≤ 500; 1 ≤ k ≤ 1000; 1 ≤ a ≤ n; 1 ≤ b ≤ m. |
| **Limites** | Tempo: 1,00 s. Memória: 512 MB. |

## 3. Modelagem em grafo

- **Vértices:** os `n` meninos (conjunto A), as `m` meninas (conjunto B), mais uma origem `s` e um destino `t` para o modelo de fluxo.
- **Arestas:**
  - `s → a`, para cada menino `a`;
  - `a → b`, para cada par potencial `(a, b)`;
  - `b → t`, para cada menina `b`.
- **Direção:** sempre no sentido `s → menino → menina → t`.
- **Capacidades:** todas iguais a 1. Isso garante que cada menino e cada menina sejam usados no máximo uma vez.
- **Pesos:** não há. Só importa a quantidade de duplas, não um custo.

## 4. Família do problema

**Fluxo máximo.** Este é o caso clássico de *emparelhamento máximo em grafo bipartido*, que se reduz a fluxo máximo em uma rede de capacidades unitárias: o valor do fluxo máximo de `s` a `t` é igual ao tamanho do emparelhamento máximo.

Não se trata de:

- **Árvore geradora mínima**, pois não buscamos conectar todos os vértices com custo mínimo;
- **Caminho mínimo**, pois não há custos nem distância entre dois vértices a minimizar.

## 5. Exemplo

Entrada:

```
3 2 4
1 1
1 2
2 1
3 1
```

Saída (uma das soluções válidas):

```
2
1 2
3 1
```
