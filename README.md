# orgnamematch
This repository addresses a specific problem often faced by social scientists studying large-scale organizational network data: how to merge nodes that actually refer to the same entity. Here are some motivating examples:

1. A [large dataset of lobbying records](https://www.cambridge.org/core/journals/state-politics-and-policy-quarterly/article/chorus-a-new-dataset-of-state-interest-group-policy-positions-in-the-united-states/6827DC9EC72301016894F265777C0078) contains entries with the form "Org $i$ lobbied for / against Bill $j$." This is a bipartite multiplex network. Errors in data entry mean that in some cases, Orgs $i$ and $k$ are actually the same organization. The only way we can recognize these cases is by looking at the plaintext names of the two organizations and their locations in the network.  
2. A corporate board interlocks dataset where firms are nodes and edges represent shared directors—for example, "JP Morgan Chase & Co." vs. "J.P. Morgan Chase"—requires merging variants of the same company.  
3. A nonprofit affiliation network listing charitable organizations under slightly different labels, such as "World Health Organization" vs. "WHO" or "World Health Org.", all referring to the same entity.  
4. A political campaign finance dataset recording PAC contributions with names like "Sierra Club Political Committee" vs. "Sierra Club PC," which must be consolidated for accurate network analysis.  

## Model
We have two types of information to go off of: each node's position in the network, and the text of each node's name. Sometimes two names obviously reference the same organization, as in "Pfizer" vs. "Pfzer, Inc.". In this case nodes $i$ and $k$ almost certainly ought to be merged regardless of their positions in the network. Other times, two names may look quite similar but in fact reference different organizations, as in "American Medical Association" vs. "American Marketing Association". In these cases the network position of nodes $i$ and $k$ becomes more important.

We model this problem formally using a generative statistical model. We assume there is a ground truth graph $G = (V, E)$, where $E = \{E_1, \ldots E_l\}$ in the multiplex case. Each node $v_i \in V$ has an associated `name` $n_i$. In practice these names are strings. But for the purpose of our model, we assume each string can equally be represented by an embedding in a $d$‑dimensional vector space $D$. Let us call the input space of one-hot encoded names $N$. Then there is a one-to-one and onto mapping $f : N \rightarrow D$ which maps names $n_i$ to embeddings $d_i$.  

A "corruption" process $C$ acts on $G$ as follows:

- Each node $v_i \in V$ is split into $k$ nodes $\{v_i^{(1)} \ldots v_i^{(k)}\}$ with probability $p_k = e^{(-\beta k)}$, where $\beta$ is a scale parameter.  
- The edges incident on $v_i$ are allocated randomly among the $v_i^{(j)}$.  
- The `name` of each $v_i^{(j)}$ is corrupted. Formally, a corruption amounts to a random perturbation of the name's embedding $d_i \in D$: $d_i \rightarrow d_i + \eta$, where $\eta \sim \mathcal{N}(0, \sigma)$, i.e. the name perturbation is Gaussian distributed in embedding space.  

### Likelihood
Let 
* $G = (V, E)$ be the unobserved "true" graph with node-names $\{d_i\}_{i \in V}$ in embedding space $D$.
* $G' = (V', E')$ be the observed corrupted graph with embeddings $\{d'_m\}_{m \in V'}$.
* $M: V' \to V$ be the unknown many-to-one mapping of corrupted nodes back to originals.
* $k_i = |M^{-1}(i)|$ be the number of splits of true node $i$.

We assume the following factorization of the joint likelihood:
$$P(G', D', M | G, \beta, \sigma) = P(M | \beta) \cdot P(E' | E, M) \cdot P(D' | G, M, \sigma).$$

1. **Splitting prior**  
    For each $i \in V$,  
    $$p(k_i | \beta) = \frac{e^{-\beta k_i}}{Z(\beta)},$$  
    so  
    $$P(M | \beta) = \prod_{i \in V} \frac{e^{-\beta k_i}}{Z(\beta)}$$

2. **Edge-allocation likelihood**  
    Each true edge $(i,j) \in E$ is assigned to exactly one corrupted copy of $i$ and one of $j$, uniformly at random. If $\deg_i$ is the true degree of $i$, then  
    $$P(E' | E, M) = \prod_{(i,j) \in E} \frac{1}{k_i \cdot k_j}.$$

3. **Name-corruption likelihood**  
    Given true embedding $d_i$, each corrupted copy $m \in M^{-1}(i)$ has  
    $$d'_m \sim \mathcal{N}(d_i, \sigma^2 I).$$  
    Hence  
    $$P(D' | G, M, \sigma) = \prod_{i \in V} \prod_{m \in M^{-1}(i)} \mathcal{N}(d'_m; d_i, \sigma^2 I).$$

Putting it all together, the joint data likelihood is  
$$P(G', D' | G, \beta, \sigma) = \sum_{M} \left[ \prod_{i \in V} \frac{e^{-\beta k_i}}{Z(\beta)} \right] \left[ \prod_{(i,j) \in E} \frac{1}{k_i k_j} \right] \left[ \prod_{i \in V} \prod_{m \in M^{-1}(i)} \mathcal{N}(d'_m; d_i, \sigma^2 I) \right].$$

### Remarks and extensions

- One can place a prior on G (e.g.\ an ERGM) and on the true embeddings \{d_i\}.  
- The partition function Z(β) may be intractable; one typically uses pseudo‐likelihood or variational inference over M and G.  
- If names are observed as strings, incorporate a deterministic mapping f: N→D before corruption.  
- Inference alternates between estimating M (node merges) and the parameters (β,σ, and G).  
- The marginal likelihood over β,σ is obtained by integrating or summing out M and G.  
- Further model refinements could allow edge‐specific noise or non‐Gaussian name perturbations.  
