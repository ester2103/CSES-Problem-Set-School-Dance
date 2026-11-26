# Marco 3 — Rastreamento manual

**Problema:** Baile Escolar (emparelhamento máximo em grafo bipartido)
**Código:** `FordFulkerson.java` (algs4)


---

## 1. Instância e convenções

Entrada de exemplo: `n = 3`, `m = 2`, `k = 4`, com pares `1-1`, `1-2`, `2-1`, `3-1`.

### Numeração dos vértices

O `main` do algs4 usa `s = 0` e `t = V-1`. Escolhi esta numeração:

| Vértice | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
|---|---|---|---|---|---|---|---|
| Papel | `s` | M1 | M2 | M3 | G1 | G2 | `t` |

Logo `V = 7`.

### Arestas (`FlowEdge`), na ordem em que são inseridas

Todas com capacidade 1 e fluxo inicial 0.

| Aresta | from → to | Significado |
|---|---|---|
| e0 | 0 → 1 | `s → M1` |
| e1 | 0 → 2 | `s → M2` |
| e2 | 0 → 3 | `s → M3` |
| e3 | 1 → 4 | M1 – G1 |
| e4 | 1 → 5 | M1 – G2 |
| e5 | 2 → 4 | M2 – G1 |
| e6 | 3 → 4 | M3 – G1 |
| e7 | 4 → 6 | `G1 → t` |
| e8 | 5 → 6 | `G2 → t` |

Total: `E = 9` arestas (`n + k + m = 3 + 4 + 2`).

### Ordem de visita das arestas em `G.adj(v)`

Isso vem direto do código do algs4:

- `FlowNetwork.addEdge` faz `adj[v].add(e); adj[w].add(e);`, ou seja, coloca a mesma aresta nas listas do `from` e do `to`.
- `adj[v]` é um `Bag`, e `Bag.add` insere o novo item **no início** da lista ligada (`first = new Node; first.next = oldfirst`). A iteração começa em `first`, então percorre da aresta **inserida por último** para a primeira.

| Vértice | `adj[v]` (aresta → vizinho `w`) |
|---|---|
| 0 | e2 → 3, e1 → 2, e0 → 1 |
| 1 | e4 → 5, e3 → 4, e0 → 0 |
| 2 | e5 → 4, e1 → 0 |
| 3 | e6 → 4, e2 → 0 |
| 4 | e7 → 6, e6 → 3, e5 → 2, e3 → 1 |
| 5 | e8 → 6, e4 → 1 |
| 6 | e8 → 5, e7 → 4 |

### Métodos de `FlowEdge` usados no rastreamento (conforme `FlowEdge.java`)

- `residualCapacityTo(w)`: se `w == to`, vale `capacidade - fluxo` (aresta direta); se `w == from`, vale `fluxo` (aresta reversa, permite desfazer fluxo).
- `addResidualFlowTo(w, delta)`: se `w == to`, soma `delta` ao fluxo; se `w == from`, subtrai `delta`.
- `other(v)`: devolve a outra ponta da aresta.

---

## 2. Estruturas de dados e estados iniciais

| Estrutura | Tipo | Papel | Estado inicial |
|---|---|---|---|
| `G` | `FlowNetwork` | Rede com listas de adjacência (`Bag`) | `V = 7`, `E = 9`, todos os fluxos 0 |
| `value` | `double` | Valor do fluxo atual | `excess(G, t) = 0.0` |
| `marked[]` | `boolean[7]` | `marked[v]` = existe caminho `s → v` na rede residual | recriado a cada chamada de `hasAugmentingPath` (todos `false`) |
| `edgeTo[]` | `FlowEdge[7]` | Última aresta do caminho `s → v` (árvore da BFS) | recriado a cada chamada (todos `null`) |
| `queue` | `Queue<Integer>` | Fila da BFS | criada vazia a cada chamada |

Antes do laço principal, o construtor chama `isFeasible`, que confirma que o fluxo inicial (tudo 0) respeita as capacidades e a conservação de fluxo. Passa.

---

## 3. Rastreamento

Notação: `res(v→w)` é a capacidade residual de `v` para `w`, calculada por `e.residualCapacityTo(w)`.

### Iteração 1

`hasAugmentingPath`: `marked[0] = true`, fila = `[0]`.

| Passo | Desenfileira `v` | Arestas examinadas (na ordem de `adj[v]`) | Decisão | Fila depois |
|---|---|---|---|---|
| 1 | 0 | e2 (→3): res = 1; e1 (→2): res = 1; e0 (→1): res = 1 | marca 3, 2, 1; `edgeTo[3]=e2`, `edgeTo[2]=e1`, `edgeTo[1]=e0` | `[3, 2, 1]` |
| 2 | 3 | e6 (→4): res = 1; e2 (→0): res = fluxo = 0 | marca 4; `edgeTo[4]=e6` | `[2, 1, 4]` |
| 3 | 2 | e5 (→4): 4 já marcado; e1 (→0): res = 0 | nada novo | `[1, 4]` |
| 4 | 1 | e4 (→5): res = 1; e3 (→4): já marcado; e0 (→0): res = 0 | marca 5; `edgeTo[5]=e4` | `[4, 5]` |
| 5 | 4 | e7 (→6): res = 1; e6, e5, e3 (para trás): res = 0 | marca 6; `edgeTo[6]=e7`; **`marked[t] = true`**, o `while` termina | `[5, 6]` |

**Caminho** (de `t` para trás, seguindo `edgeTo`): `6 ← e7 ← 4 ← e6 ← 3 ← e2 ← 0`.
Ou seja, `s → M3 → G1 → t`.

- **Gargalo:** `min(1, 1, 1) = 1`.
- **Aumento:** `e7.flow = 1`, `e6.flow = 1`, `e2.flow = 1`.
- **`value = 0 + 1 = 1`.**

### Iteração 2

Fluxos atuais: e2 = e6 = e7 = 1; os demais 0. Novos `marked` e `edgeTo` (vazios). Fila = `[0]`.

| Passo | Desenfileira `v` | Arestas examinadas | Decisão | Fila depois |
|---|---|---|---|---|
| 1 | 0 | e2 (→3): res = 1 − 1 = **0**; e1 (→2): res = 1; e0 (→1): res = 1 | marca 2, 1; `edgeTo[2]=e1`, `edgeTo[1]=e0` | `[2, 1]` |
| 2 | 2 | e5 (→4): res = 1; e1 (→0): res = 0 | marca 4; `edgeTo[4]=e5` | `[1, 4]` |
| 3 | 1 | e4 (→5): res = 1; e3 (→4): já marcado; e0 (→0): res = 0 | marca 5; `edgeTo[5]=e4` | `[4, 5]` |
| 4 | 4 | e7 (→6): res = **0** (saturada); e6 (→3): `3` é o `from` de e6, então res = fluxo = **1** (aresta reversa); e5, e3 (para trás): res = 0 | marca 3 pela aresta reversa; `edgeTo[3]=e6` | `[5, 3]` |
| 5 | 5 | e8 (→6): res = 1; e4 (→1): res = 0 | marca 6; `edgeTo[6]=e8`; **`marked[t] = true`** | `[3, 6]` |

**Caminho:** `6 ← e8 ← 5 ← e4 ← 1 ← e0 ← 0`, ou seja, `s → M1 → G2 → t`.

- **Gargalo:** 1.
- **Aumento:** `e8.flow = 1`, `e4.flow = 1`, `e0.flow = 1`.
- **`value = 1 + 1 = 2`.**

> **Ponto interessante:** no passo 4 a BFS descobriu a aresta reversa `G1 → M3` (`edgeTo[3] = e6`), que permitiria desfazer a dupla M3–G1. Mas essa aresta **não faz parte do caminho final**, porque a BFS chegou a `t` antes por outro lado. Por isso `e6` continua com fluxo 1.

### Iteração 3

Fluxos atuais: e0 = e2 = e4 = e6 = e7 = e8 = 1; e1 = e3 = e5 = 0. Fila = `[0]`.

| Passo | Desenfileira `v` | Arestas examinadas | Decisão | Fila depois |
|---|---|---|---|---|
| 1 | 0 | e2 (→3): res = 0; e1 (→2): res = 1; e0 (→1): res = **0** | marca 2; `edgeTo[2]=e1` | `[2]` |
| 2 | 2 | e5 (→4): res = 1; e1 (→0): res = 0 | marca 4; `edgeTo[4]=e5` | `[4]` |
| 3 | 4 | e7 (→6): res = **0**; e6 (→3): `3` é o `from`, res = fluxo = 1; e5 (→2): res = 0; e3 (→1): res = 0 | marca 3 pela aresta reversa; `edgeTo[3]=e6` | `[3]` |
| 4 | 3 | e6 (→4): res = 1 − 1 = 0; e2 (→0): res = fluxo = 1, mas 0 já está marcado | nada novo | `[]` |

A fila esvazia sem marcar `t`: `marked[6] = false`, então `hasAugmentingPath` devolve `false` e o `while` do construtor termina.

**Estado final das estruturas:**

| v | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
|---|---|---|---|---|---|---|---|
| `marked[v]` | true | false | true | true | true | false | false |
| `edgeTo[v]` | null | null | e1 | e6 | e5 | null | null |

### Fluxo em cada aresta ao longo das iterações

| Aresta | Início | Após it. 1 | Após it. 2 | Após it. 3 |
|---|---|---|---|---|
| e0 (`s→M1`) | 0 | 0 | 1 | 1 |
| e1 (`s→M2`) | 0 | 0 | 0 | 0 |
| e2 (`s→M3`) | 0 | 1 | 1 | 1 |
| e3 (M1–G1) | 0 | 0 | 0 | 0 |
| e4 (M1–G2) | 0 | 0 | 1 | 1 |
| e5 (M2–G1) | 0 | 0 | 0 | 0 |
| e6 (M3–G1) | 0 | 1 | 1 | 1 |
| e7 (`G1→t`) | 0 | 1 | 1 | 1 |
| e8 (`G2→t`) | 0 | 0 | 1 | 1 |
| **`value`** | 0 | 1 | 2 | **2** |

---

## 4. Resultado

- **Fluxo máximo:** `value = 2`.
- **Duplas:** arestas menino → menina com fluxo 1: e4 (M1 → G2) e e6 (M3 → G1).
- **Corte mínimo** (`inCut(v) = marked[v]`): lado de `s` = `{0, 2, 3, 4}` = `{s, M2, M3, G1}`.
  - Arestas que saem desse conjunto: e0 (0 → 1) e e7 (4 → 6), cada uma com capacidade 1.
  - Capacidade do corte = 1 + 1 = **2** = valor do fluxo. É exatamente o que o método `check` verifica.

Saída do programa para a instância:

```
2
1 2
3 1
```

Que é igual à saída do exemplo do enunciado.

---

## 5. Justificativa das decisões e do resultado

- **Por que BFS:** o código procura o caminho com **menos arestas** (`shortest augmenting path`). Isso evita o pior caso do Ford-Fulkerson com escolha arbitrária de caminhos e dá o limite `O(E · V · (E + V))` descrito no comentário da classe.
- **Por que `res > 0` decide se a aresta pode ser usada:** uma aresta direta só pode receber mais fluxo se `capacidade − fluxo > 0`. Uma aresta reversa só pode ser usada se já carrega fluxo (`fluxo > 0`). Por isso `e2` (saturada) foi ignorada na iteração 2, e `e6` pôde ser percorrida ao contrário nas iterações 2 e 3.
- **Por que o gargalo é 1:** todas as capacidades são 1, então cada caminho aumenta o fluxo em exatamente 1. Por isso há no máximo `min(n, m) = 2` aumentos úteis nesta instância.
- **Por que termina:** o fluxo é inteiro e cresce 1 a cada iteração, limitado por `min(n, m)`. Quando a BFS não alcança `t`, não existe caminho aumentante.
- **Por que o resultado é ótimo:** se `t` não é alcançável na rede residual, o conjunto de vértices alcançáveis define um corte `(S, T)` em que toda aresta de `S` para `T` está saturada. O valor do fluxo é igual à capacidade desse corte, e pelo teorema do **fluxo máximo / corte mínimo** nenhum fluxo pode ser maior.
- **Por que 2 é o máximo do problema:** só há 2 meninas, e cada uma pode estar em no máximo uma dupla.

---

## 6. Explicação individual de uma etapa: `hasAugmentingPath`

```java
private boolean hasAugmentingPath(FlowNetwork G, int s, int t) {
    edgeTo = new FlowEdge[G.V()];
    marked = new boolean[G.V()];

    Queue<Integer> queue = new Queue<Integer>();
    queue.enqueue(s);
    marked[s] = true;
    while (!queue.isEmpty() && !marked[t]) {
        int v = queue.dequeue();

        for (FlowEdge e : G.adj(v)) {
            int w = e.other(v);

            if (e.residualCapacityTo(w) > 0) {
                if (!marked[w]) {
                    edgeTo[w] = e;
                    marked[w] = true;
                    queue.enqueue(w);
                }
            }
        }
    }
    return marked[t];
}
```

**O que a etapa faz:** procura, na **rede residual**, um caminho de `s` até `t` com o menor número de arestas, usando BFS. Devolve `true` se existe e deixa o caminho registrado em `edgeTo[]`.

**Linha a linha:**

1. `edgeTo = new FlowEdge[G.V()]; marked = new boolean[G.V()];` — zera o estado da busca. É necessário porque a rede residual muda a cada aumento. (Nas iterações 1, 2 e 3 acima, esses vetores começaram vazios.)
2. `queue.enqueue(s); marked[s] = true;` — a BFS começa em `s`, que já nasce marcado.
3. `while (!queue.isEmpty() && !marked[t])` — continua enquanto há vértices a explorar e `t` ainda não foi alcançado. Na iteração 1, o `while` parou assim que `t` foi marcado; na iteração 3, parou porque a fila esvaziou.
4. `int v = queue.dequeue();` — pega o vértice mais antigo da fila. Essa ordem FIFO é o que garante que os vértices sejam descobertos em ordem de distância, o que faz o caminho ser o mais curto.
5. `for (FlowEdge e : G.adj(v))` — percorre todas as arestas incidentes em `v`, **tanto as que saem quanto as que entram**. Isso é o que permite enxergar as arestas reversas.
6. `int w = e.other(v);` — descobre o vizinho do outro lado da aresta.
7. `e.residualCapacityTo(w) > 0` — decide se dá para "andar" de `v` até `w`. Se `w` é o `to`, a conta é `capacidade − fluxo`; se `w` é o `from`, é o `fluxo`. Exemplo na iteração 2, passo 4: de `4` para `3` pela aresta e6 (3→4), `w = 3` é o `from`, e `res = fluxo = 1`. A aresta reversa foi usada na exploração.
8. `if (!marked[w])` — só visita cada vértice uma vez. É por isso que a BFS custa `O(V + E)`.
9. `edgeTo[w] = e; marked[w] = true; queue.enqueue(w);` — guarda **por qual aresta** chegamos a `w` (para reconstruir o caminho depois), marca `w` e o coloca na fila.
10. `return marked[t];` — se `t` foi marcado, existe caminho aumentante. Caso contrário, o fluxo atual já é máximo.

**Ligação com o resto do código:** o construtor usa `edgeTo[]` para percorrer o caminho de trás para frente (`for (int v = t; v != s; v = edgeTo[v].other(v))`), primeiro calculando o gargalo e depois aumentando o fluxo com `addResidualFlowTo`. Ao final, `marked[]` fica com o conjunto alcançável a partir de `s`, que é o corte mínimo devolvido por `inCut`.
