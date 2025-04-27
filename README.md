# orgnamematch
This repository addresses a specific problem often faced by social scientists studying large-scale organizational network data: how to merge nodes that actually refer to the same entity. Here are some motivating examples:

1. A [large dataset of lobbying records](https://www.cambridge.org/core/journals/state-politics-and-policy-quarterly/article/chorus-a-new-dataset-of-state-interest-group-policy-positions-in-the-united-states/6827DC9EC72301016894F265777C0078) contains entries with the form "Org $`i`$ lobbied `for` / `against` Bill $`j`$." This is a bipartite multiplex network. Errors in data entry mean that in some cases, Orgs $`i`$ and $`k`$ are actually the same organization. The only way we can recognize these cases is by looking at the plaintext names of the two organizations and their locations in the network.


## Strategy
We have two types of information to go off of: each node's position in the network, and the text of each node's name. Sometimes two names obviously reference the same organization, as in "Pfizer" vs. "Pfzer, Inc.". In this case nodes $`i`$ and $`k`$ almost certainly ought to be merged regardless of their positions in the network. Other times, two names may look quite similar but in fact reference different organizations, as in [fill in example]. In these cases the network position of nodes $`i`$ and $`k`$ becomes more important.

We model this problem formally as 
