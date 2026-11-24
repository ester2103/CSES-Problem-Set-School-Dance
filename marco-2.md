# Marco 2 — Escolha do algoritmo

**Problema:** Baile Escolar (emparelhamento máximo em grafo bipartido)

## 1. Algoritmo escolhido

**Ford-Fulkerson** (busca de caminhos aumentantes por DFS), na versão especializada para emparelhamento bipartido conhecida como **algoritmo de Kuhn**.

## 2. Justificativa

- **Kruskal/Prim** resolvem árvore geradora mínima e **Dijkstra** resolve caminho mínimo. Nenhum deles maximiza uma quantidade sujeita a capacidades, portanto não se aplicam.
- No modelo de fluxo do Marco 1, o fluxo máximo é igual ao emparelhamento máximo. Como as capacidades são 1, o fluxo é inteiro: cada aresta `a → b` com fluxo 1 é uma dupla.
- O fluxo máximo é no máximo `F = min(n, m) ≤ 500`. O Ford-Fulkerson faz no máximo `F` aumentos, e cada aumento custa uma busca em `O(V + E)`. Esse limite é pequeno para as restrições dadas.
- **Edmonds-Karp** (BFS) também serviria, mas não é necessário, pois o limite `F · E` já cabe no tempo.
- O algoritmo de Kuhn é o mesmo Ford-Fulkerson sem construir `s` e `t` explicitamente. Para cada menino, busca-se um caminho aumentante que alterna arestas fora e dentro do emparelhamento atual.

### Funcionamento resumido

1. Começar com emparelhamento vazio.
2. Para cada menino, tentar encontrar uma menina livre entre as que aceitam dançar com ele.
3. Se a menina já estiver emparelhada, tentar realocar o menino que está com ela para outra menina (recursivamente), liberando-a.
4. Se a busca tiver sucesso, o emparelhamento cresce em 1. Caso contrário, o menino fica sem par.
5. Ao final, as duplas são as arestas do emparelhamento.

### Corretude

Pelo **teorema de Berge**, um emparelhamento é máximo se, e somente se, não existe caminho aumentante. O algoritmo tenta aumentar o emparelhamento para cada menino. Se não existe caminho aumentante a partir dele naquele momento, também não existirá depois, então ele pode ser descartado. Ao final não restam caminhos aumentantes e o emparelhamento é máximo.

## 3. Complexidade de pior caso

Parâmetros: `V = n + m ≤ 1000`, `E = k ≤ 1000`, `F ≤ min(n, m) ≤ 500`.

| Recurso | Complexidade | Estimativa |
|---|---|---|
| **Tempo** | `O(F · (V + E))`, que na forma de Kuhn equivale a `O(n · k)` | ≈ 500 · 2000 = 10⁶ operações, bem abaixo de 1 s |
| **Espaço** | `O(n + m + k)` | Lista de adjacência e vetores de emparelhamento e de visitados, muito abaixo de 512 MB |

## 4. Representação da instância pequena

Entrada: `n = 3`, `m = 2`, `k = 4`, com arestas `1-1`, `1-2`, `2-1`, `3-1`.

Lista de adjacência (meninos → meninas):

| Menino | Meninas |
|---|---|
| 1 | 1, 2 |
| 2 | 1 |
| 3 | 1 |

## 5. Validação da entrada e da saída esperada

Execução passo a passo:

1. **Menino 1:** tenta a menina 1, que está livre. Emparelha `1-1`.
2. **Menino 2:** só aceita a menina 1, que está com o menino 1. Tenta realocar o menino 1 para a menina 2, que está livre. Resultado: `1-2` e `2-1`.
3. **Menino 3:** só aceita a menina 1, que está com o menino 2. O menino 2 não tem alternativa, então não há caminho aumentante. O menino 3 fica sem par.

**Resultado:** 2 duplas (`1-2` e `2-1`). O valor `r = 2` coincide com a saída esperada. A lista de duplas difere da do enunciado (`1-2`, `3-1`), mas é igualmente válida, pois qualquer solução ótima é aceita.

O valor 2 é de fato o máximo: há apenas 2 meninas (`m = 2`), então não é possível formar mais de 2 duplas.
