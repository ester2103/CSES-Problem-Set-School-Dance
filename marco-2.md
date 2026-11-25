# Marco 2 — Escolha do algoritmo

**Problema:** Baile Escolar (emparelhamento máximo em grafo bipartido)

## 1. Algoritmo escolhido

**Ford-Fulkerson**, com busca de caminhos aumentantes por DFS, aplicado à rede de fluxo construída no Marco 1.

## 2. Justificativa

- **Kruskal/Prim** resolvem árvore geradora mínima: ligam todos os vértices com custo mínimo. Aqui não há custos nem necessidade de conectar todos os vértices.
- **Dijkstra** resolve caminho mínimo entre vértices. Aqui não buscamos um caminho curto, e sim o máximo de duplas.
- O problema pede o **máximo** de algo sujeito a **capacidades** (cada pessoa só pode ser usada uma vez). Isso é **fluxo máximo**, e o Ford-Fulkerson é o algoritmo clássico para ele.
- Como todas as capacidades são inteiras (iguais a 1), o Ford-Fulkerson sempre termina e produz um fluxo inteiro. Cada aresta `menino → menina` com fluxo 1 é uma dupla.
- **Edmonds-Karp** (Ford-Fulkerson com BFS) também serviria, mas o limite do Ford-Fulkerson com DFS, `O(F · E)`, já é pequeno para estas restrições.

## 3. Rede de fluxo

| Elemento | Definição |
|---|---|
| Vértices | `s` (origem), `n` meninos, `m` meninas, `t` (destino): `V = n + m + 2` |
| Arestas `s → a` | uma para cada menino `a`, capacidade 1 |
| Arestas `a → b` | uma para cada par potencial `(a, b)`, capacidade 1 |
| Arestas `b → t` | uma para cada menina `b`, capacidade 1 |
| Número de arestas | `E = n + k + m` |

### Funcionamento

1. Começar com fluxo 0 em todas as arestas.
2. Construir a **rede residual**: cada aresta com capacidade restante gera uma aresta direta, e cada aresta com fluxo gera uma aresta reversa que permite desfazer esse fluxo.
3. Fazer uma DFS de `s` até `t` na rede residual. Se encontrar um caminho, enviar 1 unidade de fluxo por ele (o gargalo é sempre 1).
4. Repetir o passo 3 até não existir mais caminho de `s` até `t`.
5. O valor do fluxo é `r`. As duplas são as arestas `menino → menina` com fluxo 1.

### Corretude

Pelo teorema do **fluxo máximo / corte mínimo**, o fluxo é máximo quando não há mais caminho aumentante de `s` até `t` na rede residual. Nesse ponto, o fluxo é igual à capacidade de um corte mínimo. Como o fluxo inteiro corresponde a um emparelhamento, o emparelhamento obtido é máximo.

## 4. Complexidade de pior caso

Parâmetros: `V = n + m + 2 ≤ 1002`, `E = n + k + m ≤ 2000`, `F ≤ min(n, m) ≤ 500`.

### Tempo: `O(F · (V + E))`

- **No máximo `F` aumentos:** cada caminho aumentante soma exatamente 1 ao fluxo, porque as capacidades são 1.
- **`F ≤ min(n, m)`:** cada dupla usa 1 menino e 1 menina, então não há mais duplas do que meninos nem do que meninas.
- **Cada DFS custa `O(V + E)`:** ela visita cada vértice uma única vez (vértices já visitados são marcados) e percorre cada aresta uma única vez.
- **Estimativa:** `500 · (1002 + 2000) ≈ 1,5 · 10⁶` operações, muito abaixo do limite de 1 s.

### Espaço: `O(V + E)`, ou seja, `O(n + m + k)`

- Lista de adjacência com as arestas da rede e suas arestas reversas: `2 · E` entradas.
- Vetor de capacidades residuais: `2 · E` valores.
- Vetor de visitados da DFS: `V` posições.
- Pilha de recursão da DFS: no máximo `V` níveis.

Tudo isso soma alguns milhares de inteiros, muito abaixo do limite de 512 MB.

## 5. Representação da instância pequena

Entrada: `n = 3`, `m = 2`, `k = 4`, com pares `1-1`, `1-2`, `2-1`, `3-1`.

Arestas da rede (todas com capacidade 1):

| De | Para |
|---|---|
| `s` | M1, M2, M3 |
| M1 | G1, G2 |
| M2 | G1 |
| M3 | G1 |
| G1, G2 | `t` |

## 6. Validação: passo a passo


| Iteração | Caminho aumentante encontrado | Fluxo |
|---|---|---|
| 1 | `s → M1 → G1 → t` | 1 |
| 2 | `s → M2 → G1 → M1 → G2 → t` (usa a aresta reversa `G1 → M1`) | 2 |
| 3 | Nenhum caminho de `s` até `t` | fim |

**Iteração 2:** o menino 2 só aceita a menina 1, que já está com o menino 1. O caminho passa pela aresta reversa `G1 → M1`, que desfaz a dupla `M1-G1`, e leva o menino 1 para a menina 2.

**Iteração 3:** os vértices alcançáveis a partir de `s` na rede residual são `s`, M2, M3 e G1. O corte formado pelas arestas `s → M1` e `G1 → t` tem capacidade 2, igual ao fluxo. Logo, o fluxo 2 é máximo.

**Saída obtida:** arestas `menino → menina` com fluxo 1 são `M1 → G2` e `M2 → G1`.

```
2
1 2
2 1
```

O valor `r = 2` coincide com o exemplo do enunciado. A lista de duplas é diferente (`1 2` e `3 1` no enunciado), mas também é válida, pois qualquer solução ótima é aceita. O valor 2 é o máximo possível, pois há apenas 2 meninas.
