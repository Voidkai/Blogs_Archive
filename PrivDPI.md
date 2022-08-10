# PrivDPI——Literature Notes

$PrivDPI$, which reduces the setup delay while retaining similar privacy guarantee:

The intermediate values generated in each session can be reused across subsequent sessions for repeated tokens, which could further speedup token encryption.



## System Flow

$PrivDPI$ consists of the following six phases:

-  **Setup**: **MB** receives **a set of rule tuples** from **RG**. And,  **C** and **S** establish **a session key** through the TLS handshake protocol.
-  **Preprocessing**: **MB** interacts with **C** and **S** to establish **a set of reusable obfuscated rules**.
-  **Session Rule Preparation**: the reusable obfuscated rules generated in the Preprocessing phase are used as "seeds" to establish **the session rules** for the TLS session.
-  **Token Encryption**: **Each tokens is encrypted**, which will be used for matching during the token detection phase.
-  **Token Detection**:  **MB** first generated encrypted rules using the session rules generated in the session rule preparation phase. It then performs token detection based on the encrypted tokens received from **C** and the encrypted rules it generated.

Preliminaries:

- Notation : 
  - PPT: probabilistic polynomial-time
  - $\mathbb{N}$ : natural numbers
  - [N] :  the set {1,....,N}
  - [ i, j] : the set {i, i+1, i+2,....,j}
- Bilinear Maps:
  - $G$ and $G_T$: two multiplicative cyclic groups of prime order $p$.
- Computational Diffie-Hellman (CDH) Assumption.
- Decisional Diffie-Hellman (DDH) Assumption

Notation:

![](https://cdn.kaixuan.site/PrivDPI/notation.PNG#vwid=1441&vhei=356)



## Protocol

### Setup:

For a rule set $\{r_i \in \mathcal{R}\}_{i \in [N]}$ where $\mathcal{R}$ is the rule domain, **RG** chooses a random secret $\alpha \in \mathbb{Z}_p$, random $s_i \in \mathbb{Z}_p$ for each $r_i$ , and sets $A = g^\alpha, R_i = g^{\alpha r_i + s_i}$ for $i \in [N]$. **RG** then signs $\{R_i\}_{i \in [N]}$ with its private key to obtain a set of signatures $\{sig(R_i)\}_{i \in [N]}$. Finally, **RG** sends the Rule tuple to **MB**: 
$$
(\{s_i, R_i,sig(R_i)\}_{i \in [N]}) \rightarrow (\{s_i,g^{\alpha r_i+s_i},sig(g^{\alpha r_i+s_i})\}_{i \in [N]})
$$
**C** and **S** derive the following three keys using sk(session key):

- $k_{rand}$: will be used as a seed for generating randomness.
- $k$: used for generating reusable obfuscated rules and session rules.
- $k_{TLS}$: is the regular TLS key, used to encrypted the traffic.



### Preprocessing Protocol:

#### Input:

**MB** has inputs  the rule tuple from **RG** and **C** and **S** have common input k;

#### protocol:

1. **C** computes $K_c = g^k$(using her $k$), and sends $K_c$ to **MB**. Similarly, **S** computes $K_s = g^k$(using her $k$) and sends $K_s$ to **MB**.
2. **MB** checks whether $K_c$ equals $K_s$, If not, it halts and outputs $\perp$ . Otherwise, it sends $(\{R_i,sig(R_i)\}_{i \in [N]})$ to **C**.
3. **C** does as follows:
   1. For $i \in [N]$, check if $Sig(R_i)$ is a valid signature on $R_i$ using the public key of the signature scheme used by **RG**. If not , halt and output $\perp$ .
   2. Compute $K_i = (R_i \cdot K_c)^k = g^{k \alpha r_i + k s_i + k^2} $for $i \in [N]$. Finally, send $\{K_i\}_{i \in [N]}$ to **MB**.
4. For $i \in [N]$, **MB** verifies whether the equation $e(K_i,g) = e(R_i \cdot K_c,K_c)$ holds. If not, it halts and output $\perp$ . Otherwise, fir $i \in [N]$, it calculates the reusable obfuscated rule $I_i = K_i/(K_c)^{s_i} = g^{k \alpha r_i + k^2}$ for future use.

#### Notes:

**MB** and **C** ensure the identification for each other By checking whether $K_c$ equals $K_s$ ,and $Sig(R_i)$ is a valid signature, then  **MB** calculate the reusable obfuscated rule $I_i$.



### Session Rule Preparation Protocol:

#### Input:

**MB** has inputs the reusable obfuscated rule $\{I_i\}_{i \in [N]}$ from last phrase, which MB calculated it. **C** and **S** have common input $k$ for the first session and $\hat{k}$ for a subsequent session.

#### Protocol:

**Case 1**: For the first session, **MB** sets $\{S_i = I_i\}_{i \in [N]}$ as the session rule.

**Case 2**: For a subsequent session, the protocol calculate as follows:

1. **C** sends $\widehat{K}_c = g^{\widehat{k}}$ to **MB** , Similarly, **S** sends $\widehat{K}_s = g^{\hat{k}}$ to **MB**.
2. **MB** checks whether $\widehat{K}_c$ equals $\widehat{K}_s$. If not, it halts and output $\perp$. Otherwise, for $i \in [N]$, it calculates $S_i = I_i \cdot \widehat{K}_c$ as session rule.

#### Notes:

the first session, the session rule equals the reusable obfuscated rule. If the session is not the first, check whether the client and server are valid, then the session rule equals that the obfuscated rule times $g^{\hat{k}}$. 



### Token Encryption Algorithm:

#### Input:

A counter table $CT_c$, a token $t$, and $k$(for the first session). and $\widehat{K}_c$ , $A = g^\alpha$.

#### Algorithm:

**Case 1**: For the first session, the algorithm operates as follows: 

1. For every token $t$, there are two cases:
   - If there does not  exist a tuple corresponding to $t$ in $CT_c$:compute $T_t = A^{kt} \cdot g^{k^2} = g^{k\alpha t+k^2}$, set $ct_t \leftarrow 0, ct \leftarrow ct_t$, and insert tuple $(t,T_t,ct_t)$ into $CT_t$.
   - If there exist a tuple $(t^{'},T_{t^{'}},ct_{t^{'}})$ in $CT_c$ satisfying $t^{'} = t$: set $T_t\leftarrow T_{t^{'}}, ct_{t^{'}} \leftarrow ct_{t^{'}}+1$ and $ct \leftarrow ct_{t^{'}}$
2. Compute the encryption of $t$ as $C_t = H(\mathrm{salt} + ct,T_t)$.

**Case 2**: For a new session, the algorithm operates as follows:

1. Compute $T_c = A^{kt} \cdot g^{k^2} \cdot \widehat{K}_c = g^{k\alpha t+k^2+\hat{k}}$.
2. For every token $t$, we have the following cases:
   - If there does not exists a tuple corresponding to $t$ in $CT_c$: set $ct_t \leftarrow 0, ct \leftarrow ct_t$, and insert tuple $(t,T_t,ct_t)$ into $CT_t$.
   - If there exist a tuple $(t^{'},T_{t^{'}},ct_{t^{'}})$ in $CT_c$ satisfying $t^{'} = t$ and $T_{t^{'}}\neq T_t$:set $T_{t^{'}}\leftarrow T_t, ct_{t^{'}} \leftarrow 0$ and $ct \leftarrow ct_{t^{'}}$
   - If there exist a tuple $(t^{'},T_{t^{'}},ct_{t^{'}})$ in $CT_c$ satisfying $t^{'} = t$ and $T_{t^{'}}=T_t$:set $ct_{t^{'}} \leftarrow ct_{t^{'}}+1$ and $ct \leftarrow ct_{t^{'}}$
3. Compute the encryption of $t$ as $C_t = H(\mathrm{salt} + ct, T_t)$



### Token Detection Algorithm:

#### Input:

A counter table $CT_{mb}$ and a fast search tree.

#### Scheme:

For each encrypted token $C_t$ in the traffic stream, if there exists a $C_{r_i}$ equals $C_t$ for some $i\in[N]$,do

1. Take the corresponding action dictated by the security policy.
2. Delete the node in tree corresponding to $r_i$ and insert $C_{r_i} = H(\mathrm{salt} +ct_i +1, S_i)$.
3. Set $ct_{r_i}\leftarrow ct_{r_i} +1$.