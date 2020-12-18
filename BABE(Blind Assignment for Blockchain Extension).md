# BABE(Blind Assignment for Blockchain Extension)

## Overview of BABE

- In Quroboros Praos
  - each party is selected to produce a block in a slot with **the probability proportional to the party's stake**.
  - Compared with Quroboros Praos, in BABE, if parties are online in the slots that they are supposed to produce a block. However, if they were offline, then they are punished by having less chance to be selected in the future slots. 
  - In Ouroboros and Quroboros Praos, the best chain is the longest chain. In Quroboros Genesis, the best chain can be the longest chain or the chain which is forked long enough and denser than the other chains in some interval.
- There are two versions of BABE
  - The first version is the same as Ouroboros Praos
  - The second version solved the problem related to offline parties.

## BABE

There are sequential non-overlapping epochs ($e_1,e_2,...$), and each epoch contains a number of sequential slots($s_1^i,s_2^i,...,s_t^i$) up to some bound $t$. We randomly assign each slot to a party, many parties or none at the beginning of the epoch, as well as the assignment is blind, so the protocol is named Blind Assignment for Blockchain Extension.
s
Each party $P_j$ has at least two types of secret/ public key pairs:

- Account Keys ($sk_j^a, pk_j^a$) which are used to sign transactions.
- Session keys consists of two keys: Verifiable random function keys($sk_j^{vrf}, pk_j^{vrf}$) and the signing keys for blocks($sk_j^{sgn}, pk_j^{sgn}$).

VRF keys could long live, but the associated signing keys should be updated from time to time for forwarding security against attackers causing slashing.

For Each party $P_j$ :

- keeps a local set of blockchains $C_j=\{C_1,C_2,...,C_l\}$. These chains have some common blocks (at least the genesis block) until some height.
- has a local buffer that contains the transactions signed with the account keys to be added to blocks

## BABE with GRANDPA Validators $\approx$Ouroboros Praos

Define probability of being selected as:
$$
P_i = \phi_c(\alpha_i) = 1-(1-c)^{\alpha_i}
$$
where $\alpha_i$ is the relative stake of the party $P_i$ and $c$ is a constant. Importantly, the function $\phi$ is that it has the '**independent aggregation'** property, which informally means the probability of being selected as a slot leader does not increase as a party splits his stakes across virtual parties.

the threshold for each party $P_i$
$$
\tau_i = 2^{l_{vrf}}\phi_c(\alpha_i) = 2^{l_{vrf}}\times(1-(1-c)^{\alpha_i})
$$
where $l_{vrf}$ is the length of the VRF's first output.

### Three phases

#### 1. Genesis Phase

- Manually produce the unique genesis block
  - which contain a random number $r_1$ for use during the first epoch for slot leader assignments, and We might set $r_1 = 0$ or use public random number from the Tor network instead.
- Initial stake of the stake holders($st_1, st_2, ...,st_n$) and their session public keys and account public keys.

#### 2. Normal Phase

- Each slot leader should produce and publish a block. And other nodes attempt to update their chain by extension with new valid blocks they observe.

- $P_j$ has a set of chains $\mathbb{C}_j$ in the current slot $sl_k$ in the epoch $e_m$. We have a best chain $C$ selected in $sl_{k-1}$ by our selection scheme, and the length of $C$ is $l-1$.

- if the first output ($d$) of the following VRF is less than the threshold $\tau_j$ then he is the slot leader. So the more stake the $P_j$ have, the more he has a chance to be selected as a slot leader.
  $$
  VRF_{sk_j^{vrf}}(r_m||sl_k)\rightarrow (d,\pi)
  $$

- Generate or Validate

  - When $P_j$ generates a block. The block should contain the slot number, the hash of the previous block, the VRF output, trhansactions and the signature$\sigma = Sign_{sk_j^{sgn}}(sk_j||H_{j-1}||d||\pi||tx)$
  - When $P_j$ receivess a block $B=(sl,H,d^{'},\pi^{'},tx^{'},\sigma^{'})$ produced by a party $P_t$, it validates the block with $Validate(B)$.
    - if $Verify_{pk_t^{sgn}}(\sigma^{'})\rightarrow valid$.
    - if $sl\leq sl_k$.
    - if the party is the slot leader: $Verify_{pk_t^{vrf}}(\pi^{'},r_m||sl)\rightarrow valid$ and $d^{'} < \tau_t$.
    - if $P_t$ did not produce another block for another chain in slot $sl$ (no double signature).
    - if there exists a chain $C^{'}$ with the header $H$

#### 3. Epoch Update





## Best Chain Selection





## Relative Time



