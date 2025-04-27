# orgnamematch
This repository addresses a specific problem often faced by social scientists studying large-scale organizational network data: how to merge nodes that actually refer to the same entity. Here are some motivating examples:

1. A [large dataset of lobbying records](https://www.cambridge.org/core/journals/state-politics-and-policy-quarterly/article/chorus-a-new-dataset-of-state-interest-group-policy-positions-in-the-united-states/6827DC9EC72301016894F265777C0078) contains entries with the form "Org $`i`$ lobbied `for` / `against` Bill $`j`$." This is a bipartite multiplex network. Errors in data entry mean that in some cases, Orgs $`i`$ and $`k`$ are actually the same organization. The only way we can recognize these cases is by looking at the plaintext names of the two organizations and their locations in the network.
2. [fill in 1-3 more examples]

## Model
We have two types of information to go off of: each node's position in the network, and the text of each node's name. Sometimes two names obviously reference the same organization, as in "Pfizer" vs. "Pfzer, Inc.". In this case nodes $`i`$ and $`k`$ almost certainly ought to be merged regardless of their positions in the network. Other times, two names may look quite similar but in fact reference different organizations, as in [fill in example]. In these cases the network position of nodes $`i`$ and $`k`$ becomes more important.

We model this problem formally using a generative statistical model. We assume there is a ground truth graph $G = (V, E)$, where $`E = \{E_1, \ldots E_l\}`$ in the multiplex case. Each node $v_i \in V$ has an associated `name` $n_i$. In practice these names are strings. But for the purpose of our model, we assume each string can equally be represented by an embedding in a $`d-`$dimensional vector space $D$. Let us call the input space of one-hot encoded names $N$. Then there is a one-to-one and onto mapping $f : N \rightarrow D$ which maps names $n_i$ to embeddings $d_i$. 

A "corruption" process $C$ acts on $G$ as follows:

- Each node $v_i \in V$ is split into $k$ nodes $`\{v_i^{(1)} \ldots v_i^{(k)}\}`$ with probability $p_k = e^{-\beta k}$, where $\beta$ is a scale parameter.
- The edges incident on $v_i$ are allocated randomly among the $v_i^{(j)}$.
- The `name` of each $v_i^{(j)}$ is corrupted. Formally, a corruption amounts to a random perturbation of the name's embedding $d_i \in D$: $d_i \rightarrow d_i + \eta$, where $\eta \sim \mathcal{N}(0, \sigma)$, i.e. the name perturbation is Gaussian distributed in embedding space.
