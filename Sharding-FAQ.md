### What is sharding?

Currently, all blockchain protocols rely on a model where all nodes
store all state (account balances, contract code and storage, etc) and
process all transactions. This provides a large amount of security, but
greatly limits blockchains’ scalability: a blockchain cannot process
more transactions than a single node in the network. In large part because of this,
Bitcoin is limited to \~3-7 transactions per second, Ethereum to 7-15,
etc. However, this poses a question: are there ways to create a new kind
of blockchain mechanism, one which departs from the model where
literally every computer in the network literally checks every
transaction, and instead only requires small subset of
nodes to verify each transaction? As long as there are sufficiently many
nodes verifying each transaction that the system is still highly secure,
but sufficiently few that the system can process many groups of
transactions in parallel, could we not use such a technique to greatly
increase a blockchain's throughput?

### What are some trivial but flawed ways of solving the problem?

There are three main categories of “easy solutions”. The first is to
simply give up on scaling individual blockchains, and instead assume
that users will be using many different “altcoins”. This greatly
increases throughput, but comes at a cost of security: an N-factor
increase in throughput using this method necessarily comes with an
N-factor decrease in security. Hence, it is arguably non-viable for more
than small values of N.

The second is to simply increase the block size limit. This can work to
some extent, and in some situations may well be the correct
prescription as block sizes may well be constrained more by politics
than by realistic technical considerations,
but regardless of one’s beliefs about any individual case such an
approach inevitably has its limits: if one goes too far, then nodes 
running on consumer hardware will drop out, the
network will start to rely exclusively on a very small number of
supercomputers running the blockchain, and this can lead to great
centralization risk.

The third is “merge mining”, a technique where there are many chains,
but all chains share the same mining power (or in proof of stake systems
stake). Currently, Namecoin gets a large portion of its security from
the Bitcoin blockchain by doing this. If all miners participate, this
theoretically can increase throughput by a factor of N without
compromising security. However, this also has the problem that it
increases the computational and storage load on each miner by a factor
of N, and so in fact this solution is simply a stealthy form of block
size increase. If only a few miners participate in merge-mining each
chain, then the centralization risk is mitigated, but the security
benefits of merge mining are also greatly reduced.

### This sounds like there’s some kind of scalability trilemma at play. What is this trilemma and can we break through it?

The trilemma claims that blockchain systems can only at most have two of
the following three properties:

-   **Decentralization** (defined as the system being able to run in a
    scenario where each participant only has access to O(c) resources,
    ie. a regular laptop or small VPS)
-   **Scalability** (defined as being able to process O(n) \> O(c)
    transactions)
-   **Security** (defined as being secure against attackers with up to O(n)
    resources)

In the rest of this document, we’ll continue using **c** to refer to the
size of computational resources (including computation, bandwidth and
storage) available to each node, and **n** to refer to the size of the
ecosystem in some abstract sense; we assume that transaction load, state
size, and the market cap of a cryptocurrency are all proportional to **n**.

### Some people claim that because of Metcalfe’s law, the market cap of a cryptocurrency should be proportional to n\^2, and not n. Do they have a point?

No.

### Why not?

Metcalfe’s law claims that the value of a network is proportional to the
square of the number of users, because if a network has n users then the
network has value for each user, but then the value for each individual
user it itself proportional to the number of users because if a network
has n users that’s n-1 potential connections through the network that
each individual user could benefit from.

In practice, [empirical research
suggests](https://en.wikipedia.org/wiki/Metcalfe%27s_law) that
the value of a network with n users is close to “n^2 proportionality for
small values of n and (n × log n) proportionality for large values of
n.” This makes sense because for small values, the argument holds true,
but once a system gets bigger, two effects slow the growth down. First,
growth in practice often happens in communities, and so in a
medium-scale network the network often already provides most of the
connections that each user cares about. Second, connections are often
substitutes from each other, and you could argue that people only derive
\~O(log(k)) value from having k connections - having 23 brands of
deodorant to choose from is nice, but it’s not that much better than
having 22 choices, whereas the difference between one choice and zero
choices is very significant.

Furthermore, even if the value of a cryptocurrency is proportional to
O(k \* log(k)) with k users, if we accept the above explanation as the
reason why this is the case, then that also implies that transaction volume is
also O(k \* log(k)), as the log(k) value per user theoretically comes
from that user exercising log(k) connections through the network, and
state size should also in many cases grow with O(k \* log(k)) as there
are at least some kinds of state that are relationship-specific rather
than user-specific. Hence, assuming n = O(k \* log(k)) and basing
everything off of **n** (size of the ecosystem) and **c** (a single node’s
computing power) is a perfectly fine model for us to use.

### What are some moderately simple but only partial ways of solving the scalability problem?

Many sharding proposals (eg. [this early BFT sharding proposal from Loi
Luu et al at
NUS](https://www.comp.nus.edu.sg/~loiluu/papers/elastico.pdf),
as well as [this Merklix
tree](http://www.deadalnix.me/2016/11/06/using-merklix-tree-to-shard-block-validation)<sup>[1](#ftnt_ref1)</sup> approach
that has been suggested for Bitcoin)
attempt to either only shard transaction processing or only shard state,
without touching the other<sup>[2](#ftnt_ref2)</sup>. These efforts are admirable
and may lead to some gains in efficiency, but they run into the
fundamental problem that they only solve one of the two bottlenecks. We
want to be able to process 10,000+ transactions per second without
either forcing every node to be a supercomputer or forcing every node to
store a terabyte of state data, and this requires a comprehensive
solution where the workload of state storage, transaction processing and
even transaction downloading and re-broadcasting are all spread out
across nodes.

Particularly, note that this requires changes at the P2P level, as a
broadcast model is not scalable since it requires every node to download
and re-broadcast O(n) data (every transaction that is being sent),
whereas our decentralization criterion assumes that every node only has
access to O(c) resources of all kinds.

### What about approaches that do not try to “shard” anything?

[Bitcoin-NG](https://www.google.com/url?q=http://hackingdistributed.com/2015/10/14/bitcoin-ng/&sa=D&ust=1480305371200000&usg=AFQjCNFUUZstMrcmGGD_-KtPFIH_woQnsA) can
increase scalability somewhat by means of an alternative blockchain
design that makes it much safer for the network if nodes are spending large portions of
their CPU time verifying blocks. In simple PoW blockchains, there are
high centralization risks and the safety of consensus is weakened if
capacity is increased to the point where more than about 5% of nodes’
CPU time is spent verifying blocks; Bitcoin-NG’s design alleviates this
problem. However, this can only increase the scalability of transaction
capacity by a constant factor of perhaps
5-50x<sup>[3](#ftnt_ref3),[4](#ftnt_ref4)</sup>, and does not increase the
scalability of state. That said, Bitcoin-NG-style approaches are not
mutually exclusive with sharding, and the two can certainly be
implemented at the same time.

Channel-based strategies (lightning network, Raiden, etc) can scale
transaction capacity by a constant factor but cannot scale state storage, and also come
with their own unique sets of tradeoffs and limitations particularly involving denial-of-service attacks; on-chain
scaling via sharding (plus other techniques) and off-chain scaling via
channels are arguably both necessary and complementary.

There exist approaches that use advanced cryptography, such as
[Mimblewimble](https://scalingbitcoin.org/papers/mimblewimble.txt) and
strategies based on ZK-SNARKs, to solve one specific part of the scaling
problem: initial full node synchronization. Instead of verifying the
entire history from genesis, nodes could verify a cryptographic proof
that the current state legitimately follows from the history. These
approaches do solve a legitimate problem, although it is worth noting
that one can rely on cryptoeconomics instead of pure cryptography to
solve the same problem in a much simpler way - see Ethereum’s current
implementations of [fast syncing](https://github.com/ethereum/go-ethereum/pull/1889) and [warp syncing](https://github.com/ethcore/parity/wiki/Warp-Sync). Neither solution does
anything to alleviate state size growth or the limits of online transaction
processing.

### State size, history, cryptoeconomics, oh my! Define some of these terms before we move further!

-   **State**: a set of information that represents the “current state” of a
    system; determining whether or not a transaction is valid, as well
    as the effect of a transaction, should in the simplest model depend
    only on state. Examples of state data include the UTXO set in
    bitcoin, balances + nonces + code + storage in ethereum, and domain
    name registry entries in Namecoin.
-   **History**: an ordered list of all transactions that have taken place
    since genesis. In a simple model, the present state should be a
    deterministic function of the genesis state and the history.
-   **Transaction**: an object that goes into the history. In practice, a
    transaction represents an operation that some user wants to make,
    and is cryptographically signed.
-   **State transition function**: a function that takes a state, applies a
    transaction and outputs a new state. The computation involved may
    involve adding and subtracting balances from accounts specified by
    the transaction, verifying digital signatures and running contract
    code.
-   **Merkle tree**: a cryptographic hash tree structure that can store a
    very large amount of data, where authenticating each individual
    piece of data only takes O(log(n)) space and time. See
    [here](https://easythereentropy.wordpress.com/2014/06/04/understanding-the-ethereum-trie) for
    details. In Ethereum, the transaction set of each block, as well as
    the state, is kept in a Merkle tree, where the roots of the trees
    are committed to in a block.
-   **Receipt**: an object that represents an effect of a transaction that
    is not stored in the state, but which is still stored in a Merkle
    tree and committed to in a block so that its existence can later be
    efficiently proven even to a node that does not have all of the
    data. Logs in Ethereum are receipts; in sharded models, receipts are
    used to facilitate asynchronous cross-shard communication.
-   **Light client**: a way of interacting with a blockchain that only
    requires a very small amount (we’ll say O(1), though O(log(c)) may also be accurate in some cases) of computational
    resources, keeping track of only the block headers of the chain by
    default and acquiring any needed information about transactions,
    state or receipts by asking for and verifying Merkle proofs of the
    relevant data on an as-needed basis.
-   **State root**: the root hash of the Merkle tree representing the
    state<sup>[5](#ftnt_ref5)</sup>

<center><img src="https://github.com/vbuterin/diagrams/raw/master/scalability_faq/image02.png" width="450"></img><br>
<small><i>The Ethereum state tree, and how the state root fits into the block
structure</i></small></center>

### What is the basic idea behind sharding?

We split the state up into K = O(n / c) partitions that we call
“shards”. For example, a sharding scheme on Ethereum might put all
addresses starting with 0x00 into one shard, all addresses starting with
0x01 into another shard, etc. In simpler forms of sharding, each shard
also has its own transaction history, and the effect of transactions in
some shard k are limited to the state of shard k. However, the effect of a transaction
may depend on <i>events that earlier took place in other shards</i>; a
canonical example is transfer of money, where money can be moved from
shard i to shard j by first creating a “debit” transaction that destroys
coins in shard i, and then creating a “credit” transaction that creates
coins in shard j, pointing to a receipt created by the debit transaction
as proof that the credit is legitimate.

<img src="https://github.com/vbuterin/diagrams/raw/master/scalability_faq/image01.png" width="400"></img>

In more complex forms of sharding, transactions may in some cases have
effects that spread out across several shards and may also synchronously
ask for data from the state of multiple shards.

One can view shards in the simpler schemes as being loosely connected,
semi-independent blockchains that are all part of a common network. In a
simple version of the scheme, each user maintains a light client on all
shards, and validators fully download and track a few shards that they
are assigned to at some particular time; this approach can support
values of n up to O(c\^2), where the number of shards is K = O(c). More
complex versions use shards-of-shards schemes to increase the maximum n
that can be supported up to O(exp(c)).

A major challenge with sharding is determining the mechanism by which
the histories of each shard are agreed upon and the state of each shard
is determined - the question is, can we break the trilemma and do this
in a way where each shard has an “economic strength” of O(n), despite
only having O(n / c) worth of economic power actively verifying that
shard at any one time? As this document argues, there are many
challenges and tradeoffs involved, but most likely yes we
can<sup>[6](#ftnt_ref6)</sup>.

### How can different kinds of applications fit into a sharded blockchain?

The easiest scenario to satisfy is one where there are very many
applications that individually do not have too many users only very
occasionally and loosely interact with each other; in this case,
applications can simply live on separate shards and use cross-shard
communication via receipts to talk to each other. Note that in all
models proposed here, **users and application developers can freely
choose which shard to publish a contract or send a transaction on**.

If applications do need to talk to each other, the challenge is much
easier if the interaction can be made asynchronous - that is, if the
interaction can be done in the form of the application on shard A
generating a receipt, a transaction on shard B “consuming” the receipt
and performing some action based on it, and possibly sending a
“callback” to shard A containing some response. Generalizing this
pattern is easy, and is not difficult to incorporate into a high-level
programming language.

However, note that the in-protocol mechanisms
that would be used for asynchronous cross-shard communication would be
different and have weaker functionality compared to the mechanisms that
are available for intra-shard communication. Some of the functionality
that is currently available in non-scalable blockchains would, in a
scalable blockchain, only be available for intra-shard
communication.<sup>[7](#ftnt_ref7)</sup>.

Doing what one wants to do on a
blockchain using only asynchronous tools is not always easy. To see why,
consider the following example courtesy of Andrew Miller. Suppose that a
user wants to purchase a plane ticket and reserve a hotel, and wants to
make sure that the operation is atomic - either both reservations
succeed or neither do. If the plane ticket and hotel booking
applications are on the same shard, this is easy: create a transaction
that attempts to make both reservations, and throws an exception and
reverts everything unless both reservations succeed. If the two are on
different shards, however, this is not so easy; even without
cryptoeconomic / decentralization concerns, this is essentially the
problem of [atomic database
transactions](https://en.wikipedia.org/wiki/Atomicity_(database_systems)).

With asynchronous messages only, the simplest solution is to first
reserve the plane, then reserve the hotel, then once both reservations
succeed confirm both; the reservation mechanism would prevent anyone
else from reserving (or at least would ensure that enough spots are open
to allow all reservations to be confirmed) for some period of time. With
cross-shard synchronous transactions, the problem is easier, but the
challenge of creating a sharding solution capable of making cross-shard
atomic synchronous transactions is itself decidedly nontrivial.

If an individual application has more than O(c) usage, then that
application would need to exist across multiple chains. The feasibility
of doing this depends on the specifics of the application itself; some
applications (eg. currencies) are easily parallelizable, whereas others
(eg. certain kinds of market designs) cannot be parallelized and must be processed serially.

There are properties of sharded blockchains that we know for a fact are
impossible to achieve. [Amdahl’s
law](https://en.wikipedia.org/wiki/Amdahl%27s_law) states
that in any scenario where applications have any non-parallelizable
component, once parallelization is easily available the
non-parallelizable component quickly becomes the bottleneck. In a
general computation platform like Ethereum, it is easy to come up with
examples of non-parallelizable computation: a contract that keeps track
of an internal value x and sets x = sha3(x, tx\_data) upon receiving a
transaction is a simple example. No sharding scheme can give individual
applications of this form more than O(c) performance. Hence, it is
likely that over time sharded blockchain protocols will get better and
better at being able to handle a more and more diverse set of
application types and application interactions, but a sharded
architecture will always necessarily fall behind a single-shard
architecture in at least some ways at scales exceeding O(c).

### What are the security models that we are operating under? What is the difference between traditional byzantine fault tolerance models and more cryptoeconomic approaches such as the Zamfir model?

There are several competing models under which the safety of blockchain
designs is evaluated. The first is an honest majority (or honest
supermajority) assumption, where we assume that there is some set of
validators and up to ½ (or ⅓ or ¼) of those validators are controlled by
an attacker; the remaining validators honestly follow the protocol. A
stronger model is the uncoordinated majority assumption, where we assume
that all validators are rational in a game-theoretic sense (except the
attacker, who is motivated to make the network fail in some way), but no
more than some fraction (often between ¼ and ½) are capable of
coordinating their actions. Bitcoin proof of work with [Eyal and Sirer’s
selfish mining fix](https://arxiv.org/abs/1311.0243) is
robust up to ½ under the honest majority assumption, and up to ¼ under
the uncoordinated majority assumption.

In any majority model, an important question is: when does “the
attacker” get to choose which nodes they corrupt? Many protocols
randomly select and assign the subset of validators that is responsible
for some consensus task ahead of time; if the attacker can choose nodes
after this assignment process takes place (eg. by hacking nodes, or via
the operators of the nodes connecting with each other and colluding),
ie. the attacker is an **adaptive adversary**, then they may be able to
wreak havoc with the protocol only corrupting a few nodes, but if the
attacker must choose before (ie. the attacker is a non-adaptive or
**oblivious adversary**) then the attacker must still corrupt a large
fraction of all nodes in order to stand a chance of corrupting any
subset.

An even stronger model is the Zamfir model. The Zamfir model has the
following properties:

-   **Majorities may be dishonest and may collude**. Collusion can happen
    either through cryptoeconomic bribery via reputation or smart
    contracts, socially organized cartels, or in more extreme versions
    of the model through 51% of the coins actually being controlled by
    one actor who physically has the private keys.
-   **The attacker has some “budget”** that they are theoretically willing
    to spend (either by losing the money through in-protocol costs or
    penalties or by bribing others). This budget is proportional to
    O(n), though as we will see below there are some limited attacks
    that may succeed with only an O(c) budget and which cannot be
    defended against.
-   The Zamfir model sometimes includes an **honest/uncoordinated minority
    assumption**, where for example at least ⅛ of any “economic set” (eg.
    miners, validators) is assumed to be honest (or at least
    uncoordinated) at any given time. The presence of altruists capable
    of making bribes or voluntarily accepting losses is also sometimes
    assumed, although it is always assumed that the capital available
    for altruists to burn is much lower than the attacker’s budget.

The goal is to try to ensure that any behavior even by a majority
coalition that infringes on the protocol’s guarantees or reduces
performance is costly, and to maximize this cost. Note that some kinds
of attacks (eg. [P + epsilon
attacks](https://blog.ethereum.org/2015/01/28/p-epsilon-attack/)
have a high budget requirement but low cost - the attacker must credibly
commit to spending a lot of money if necessary, but if they succeed
attacks are very cheap.

The honest majority model is arguably highly unrealistic and has already
been empirically disproven - see Bitcoin's [SPV mining
fork](https://www.reddit.com/r/Bitcoin/comments/3c305f/if_you_are_using_any_wallet_other_than_bitcoin/csrsrf9/) for
a practical example. It proves too much: for example, an honest
majority model would imply that honest miners are willing to voluntarily
burn their own money if doing so punishes attackers in some way. The
uncoordinated majority assumption may be realistic; there is also an
intermediate model where the majority of nodes is honest but has a
budget, so they shut down if they start to lose too much money.

The Zamfir model has in some cases been criticized as being
unrealistically adversarial, although its proponents argue that if a
protocol is designed with the Zamfir model in mind then it should be
able to massively reduce the cost of consensus, as 51% attacks become an
event that could be recovered from. We will evaluate sharding in the
context of both uncoordinated majority and Zamfir models.

### How can we break the trilemma in an honest or uncoordinated majority model?

In short, random sampling. Each shard is assigned a certain number of
validators (eg. 150), and the validator that makes a block on each shard
is taken from the sample for that shard. Samples can be reshuffled
either semi-frequently (eg. once every 12 hours) or maximally frequently
(ie. there is no real independent sampling process, validators are
randomly selected for each shard from a global pool every block).

The result is that even though only a few nodes are verifying and
creating blocks on each shard at any given time, the level of security
is in fact not much lower, in an honest/uncoordinated majority model,
than what it would be if every single node was verifying and creating
blocks. The reason is simple statistics: if you assume a ⅔ honest
supermajority on the global set, and if the size of the sample is 150,
then with 99.999% probability the honest majority condition will be
satisfied on the sample. If you assume a ¾ honest supermajority on the
global set, then that probability increases to 99.999999998% (see
[here](https://en.wikipedia.org/wiki/Binomial_distribution) for
calculation details).

Hence, at least in the honest / uncoordinated majority setting, we have:

-   **Decentralization** (each node stores only O(c) data, as it’s a light
    client in O(c) shards and so stores O(1) \* O(c) = O(c) data worth
    of block headers, as well as O(c) data corresponding to the full
    state and recent history of one or several shards that it is
    assigned to at the present time)
-   **Scalability** (with O(c) shards, each shard having O(c) capacity, the
    maximum capacity is n = O(c\^2))
-   **Security** (attackers need to control at least ⅓ of the entire
    O(n)-sized validator pool in order to stand a chance of taking over
    the network).

In the Zamfir
model (or alternatively, in the “very very adaptive adversary” model),
things are not so easy, but we will get to this later. Note that because of
the imperfections of sampling, the security threshold does decrease from ½
to ⅓, but this is still a surprisingly low loss of security for what may be
a 100-1000x gain in scalability with no loss of decentralization.

### How do you actually do this sampling in proof of work, and in proof of stake?

In proof of stake, it is easy. There already is an “active validator
set” that is kept track of in the state, and one can simply sample from
this set directly. Either an in-protocol algorithm runs and chooses 150
validators for each shard, or each validator independently runs an
algorithm that uses a common source of randomness to (provably)
determine which shard they are at any given time. Note that it is very
important that the sampling assignment is “compulsory”; validators do
not have a choice of what shard they go into. If validators could
choose, then attackers with small total stake could concentrate their
stake onto one shard and attack it, thereby eliminating the system’s
security.

In proof of work, it is more difficult, as with “direct” proof of work
schemes one cannot prevent miners from applying their work to a given
shard. It may be possible to use [proof-of-file-access forms](https://www.microsoft.com/en-us/research/publication/permacoin-repurposing-bitcoin-work-for-data-preservation/) of proof of
work to lock individual miners to individual shards, but it is hard to
ensure that miners cannot quickly download or generate data that can be
used for other shards and thus circumvent such a mechanism. The best
known approach is through a technique invented by Dominic Williams
called “puzzle towers”, where miners first perform proof of work on a
common chain, which then inducts them into a proof of stake-style
validator pool, and the validator pool is then sampled just as in the
proof-of-stake case.

One possible intermediate route might look as follows. Miners can spend
a large (O(c)-sized) amount of work to create a new “cryptographic
identity”. The precise value of the proof of work solution then chooses
which shard they have to make their next block on. They can then spend
an O(1)-sized amount of work to create a block on that shard, and the
value of that proof of work solution determines which shard they can
work on next, and so on<sup>[8](#ftnt_ref8)</sup>. Note that all of these approaches make proof of
work “stateful” in some way; the necessity of this is fundamental.

### What are the tradeoffs in making sampling more or less frequent?

Selection frequency affects just how adaptive adversaries can be for the
protocol to still be secure against them; for example, if you believe
that an adaptive attack (eg. dishonest validators who discover that they
are part of the same sample banding together and colluding) can happen
in 6 hours but not less, then you would be okay with a sampling time of
4 hours but not 12 hours. This is an argument in favor of making
sampling happen as quickly as possible.

The main challenge with sampling taking place every block is that
reshuffling carries a very high amount of overhead. Specifically,
verifying a block on a shard requires knowing the state of that shard,
and so every time validators are reshuffled, validators need to download
the entire state for the new shard(s) that they are in. This requires
both a strong state size control policy (ie. economically ensuring that the size of the state does not grow too large, whether by deleting old accounts, restricting the rate of creating new accounts or a combination of the two) and a fairly long reshuffling
time to work well.

Currently, the Parity client can download and verify
a full Ethereum state snapshot via “warp-sync” in \~10 minutes; if we
reduce by 5x to compensate for temporary issues related to past DoS
attacks and increase by 20x to compensate for increased usage (10 tx/sec
instead of the current 0.5 tx/sec) (we’ll assume future state size
control policies and “dust” accumulated from longer-term usage roughly
cancel out) , we get \~40 minute state sync time, suggesting that sync
periods of 12-24 hours but not less are safe.

There are two possible paths to overcoming this challenge.

### Can we force more of the state to be held user-side so that transactions can be validated without requiring validators to hold all state data?

The techniques here tend to involve requiring users to store state data
and provide Merkle proofs along with every transaction that they send. A
transaction would be sent along with a Merkle proof-of-correct-execution, and this proof
would allow a node that only has the state root to calculate the new
state root. This proof-of-correct-execution would consist of the subset of objects in the
trie that would need to be traversed to access and verify the state
information that the transaction must verify; because Merkle proofs are
O(log(n)) sized, the proof for a transaction that accesses a constant
number of objects would also be O(log(n)) sized.

<img src="https://github.com/vbuterin/diagrams/raw/master/scalability_faq/image03.png" width="450"></img><br>
<small><i>The subset of objects in a Merkle tree that would need to be provided in
a Merkle proof of a transaction that accesses several state objects</i></small>

Implementing this scheme in its pure form has two flaws. First, it
introduces O(log(n)) overhead, although one could argue that this
O(log(n)) overhead is not as bad as it seems because it ensures that the
validator can always simply keep state data in memory and thus it
never needs to deal with the overhead of accessing the hard
drive<sup>[9](#ftnt_ref9)</sup>. Second, it can easily be applied if the state
objects that are accessed by a transaction are static, but is more
difficult to apply if the objects in question are dynamic - that is, if
the transaction execution has code of the form `read(f(read(x)))` where
the address of some state read depends on the execution result of some
other state read. In this case, the address that the transaction sender
thinks the transaction will be reading at the time that they send the
transaction may well differ from the address that is actually read when
the transaction is included in a block, and so the Merkle proof may be
insufficient<sup>[10](#ftnt_ref10)</sup>.

A compromise approach is to allow transaction senders to send a proof
that incorporates the most likely possibilities for what data would be
accessed; if the proof is sufficient, then the transaction will be
accepted, and if the state unexpectedly changes and the proof is
insufficient then either the sender must resend or some helper node in
the network resends the transaction adding the correct proof. Developers
would then be free to make transactions that have dynamic
behavior, but the more
dynamic the behavior gets the less likely transactions would be to
actually get included into blocks.

Note that validators’ transaction inclusion strategies under this
approach would need to be complicated, as they may spend millions of gas
processing a transaction only to realize that the last step accesses
some state entry that they do not have. One possible compromise is for
validators to have a strategy that accepts only (i) transactions with
very low gas costs, eg. \<100k, and (ii) transactions that statically
specify a set of contracts that they are allowed to access, and contain
proofs for the entire state of those contracts. Note that this only
applies when transactions are initially broadcasted; once a transaction
is included into a block, the order of execution is fixed, and so only
the minimal Merkle proof corresponding to the state that actually needs
to be accessed can be provided.

If validators are not reshuffled immediately, there is one further
opportunity to increase efficiency. We can expect validators to store
data from proofs of transactions that have already been processed, so
that that data does not need to be sent again; if k transactions are
sent within one reshuffling period, then this decreases the average size
of a Merkle proof from log(n) to log(n) - log(k).

### I hear talk about separating data availability verification and state calculation, and how this might solve this problem. How does this work?

A blockchain can be viewed as a cryptoeconomic system that incentivizes
validators to make economic claims about certain facts, so as to achieve
consensus and allow users to efficiently determine information about the
state of the system. These claims can be broken down into several
categories:

-   **Data availability**: a block header containing a Merkle root of a
    transaction tree effectively claims “I believe that the data that
    this Merkle tree points to is readily accessible by any node through
    the network”
-   **Order**: a chain of block headers effectively claims “I believe that
    this data came in roughly this order”
-   **State calculation**: a block header containing a state root
    effectively claims “I believe that the transaction history
    referenced by this hash leads to a state whose root hash is X”

Current blockchains heavily conflate all three. Particularly, note that
even in systems that do not have a notion of Merkle state trees, state
calculation and data availability are heavily intertwined, as those
systems have a notion of a “valid transaction” where validity is
state-dependent - for example, a transaction might only be valid if, in
the current state, the sender account has the funds to pay for it. This
conflation is in some ways convenient, but it also greatly reduces the
scope of blockchain designs that are possible.

One could imagine a design that separates out these three components, or
at least separates out state calculation from the other two parts. This
could be accomplished by having a system where shards do NOT include
state roots by default, and where there are no rules on transaction
“validity” beyond basic formatting checks; a transaction that would be
previously considered “invalid” would now in many cases simply be
considered ineffective. Shard shuffling could happen every block without
concerns about state fetching, as validators would not need to make any
state calculations in order to include blocks. A separate process would
then calculate state for all shards; this process could be made much
more robust, as it can assume that the data that it operates on is all
available (more on why this is very important in later sections).

### How is the randomness for random sampling generated?

First of all, it is important to note that even if random number
generation is heavily exploitable, this is not a fatal flaw for the
protocol; rather, it simply means that there is a medium to high
centralization incentive. The reason is that because the randomness is
picking fairly large samples, it is difficult to bias the randomness by
more than a certain amount.

The simplest way to show this is through the [binomial
distribution](https://en.wikipedia.org/wiki/Binomial_distribution),
as described above; if one wishes to avoid a sample of size N being more
than 50% corrupted by an attacker, and an attacker has p% of the global
stake pool, the chance of the attacker being able to get such a majority
during one round is:

<img src="https://github.com/vbuterin/diagrams/raw/master/scalability_faq/image00.gif"></img>

Here’s a table for what this probability would look like in practice for
various values of N and p:

<table>
<tr><td>                </td><td> N = 50         </td><td> N = 100        </td><td> N = 150        </td><td> N = 250        </td>
</tr><tr>
<td> p = 0.4        </td><td> 0.0978         </td><td> 0.0271         </td><td> 0.0082         </td><td> 0.0009         </td>
</tr><tr>
<td> p = 0.33       </td><td> 0.0108         </td><td> 0.0004         </td><td> 1.83 * 10-5   </td><td> 3.98 * 10-8   </td>
</tr><tr>
<td> p = 0.25       </td><td> 0.0001         </td><td> 6.63 * 10<sup>-8</sup>   </td><td> 4.11 * 10<sup>-11</sup>  </td><td> 1.81 * 10-17  </td><
</tr><tr>
<td> p = 0.2        </td><td> 2.09 * 10<sup>-6</sup>   </td><td> 2.14 * 10<sup>-11</sup>  </td><td> 2.50 * 10<sup>-16</sup>  </td><td> 3.96 * 10<sup>-26</sup>  </td>
</tr></table>

Hence, for N >= 150, the chance that any given random seed will lead to
a sample favoring the attacker is very small
indeed<sup>[11](#ftnt_ref11),[12](#ftnt_ref12)</sup>. What this means from the
perspective of security of randomness is that the attacker needs to have
a very large amount of freedom in choosing the random values order to break the sampling process
outright. Most vulnerabilities in proof-of-stake randomness do not allow
the attacker to simply choose a seed; at worst, they give the attacker
many chances to select the most favorable seed out of many pseudorandomly generated options. If one is very worried about
this, one can simply set N to a greater value, and add a moderately hard
key-derivation function to the process of computing the randomness, so
that it takes more than 2<sup>100</sup> computational steps to find a way to bias
the randomness sufficiently.

Now, let’s look at the risk of attacks being made that try to influence
the randomness more marginally, for purposes of profit rather than
outright takeover.  For example, suppose that there is an algorithm
which pseudorandomly selects 1000 validators out of some very large set
(each validator getting a reward of $1), an attacker has 10% of the
stake so the attacker’s average “honest” revenue 100, and at a cost of
$1 the attacker can manipulate the randomness to “re-roll the dice”
(and the attacker can do this an unlimited number of times).

Due to the [central limit
theorem](https://en.wikipedia.org/wiki/Central_limit_theorem),
the standard deviation of the number of samples, and based [on other
known results in
math](http://math.stackexchange.com/questions/89030/expectation-of-the-maximum-of-gaussian-random-variables) the
expected maximum of N random samples is slightly under M + S \* sqrt(2
\* log(N)) where M is the mean and S is the standard deviation. Hence
the reward for manipulating the randomness and effectively re-rolling
the dice (ie. increasing N) drops off sharply, eg. with 0 re-trials your
expected reward is $100, with one re-trial it's $105.5, with two it's
$108.5, with three it's $110.3, with four it's $111.6, with five it's
$112.6 and with six it's $113.5. Hence, after five retrials it stops
being worth it. As a result, an economically motivated attacker with ten
percent of stake will (socially wastefully) spend $5 to get an additional
revenue of $13, for a net surplus of $8.

However, this kind of logic assumes that one single round of re-rolling
the dice is expensive. Many older proof of stake algorithms have a
“stake grinding” vulnerability where re-rolling the dice simply means
making a computation locally on one’s computer; algorithms with this
vulnerability are certainly unacceptable in a sharding context. Newer
algorithms (see the “validator selection” section in the [proof of stake
FAQ](https://github.com/ethereum/wiki/wiki/Proof-of-Stake-FAQ))
have the property that re-rolling the dice can only be done by
voluntarily giving up one’s spot in the block creation process, which
entails giving up rewards and fees. The best way to mitigate the impact
of marginal economically motivated attacks on sample selection is to
find ways to increase this cost. One method to increase the cost by a
factor of sqrt(N) from N rounds of voting is the [majority-bit method
devised by Iddo
Bentov](https://arxiv.org/pdf/1406.5694.pdf);
the Mauve Paper’s sharding algorithm expects to use this approach.

Another form of random number generation that is not exploitable by
minority coalitions is the deterministic threshold signature approach
most researched and advocated by Dominic Williams. The strategy here is
to use a [deterministic threshold
signature](https://eprint.iacr.org/2002/081.pdf) to
generate the random seed from which samples are selected. Deterministic
threshold signatures have the property that the value is guaranteed to
be the same regardless of which of a given set of participants provides
their data to the algorithm, provided that at least ⅔ of participants do
participate honestly. This approach is more obviously not economically
exploitable and fully resistant to all forms of stake-grinding, but it
has several weaknesses:

-   **It relies on more complex cryptography** (specifically, elliptic
    curves and pairings). Other approaches rely on nothing but the
    random-oracle assumption for common hash algorithms.
-   **It fails when many validators are offline**. A desired goal for public
    blockchains is to be able to survive very large portions of the
    network simultaneously disappearing, as long as a majority of the
    remaining nodes is honest; deterministic threshold signature schemes
    at this point cannot provide this property.
-   **It’s not secure in a Zamfir model** where more than ⅔ of validators
    are colluding. The other approaches described in the proof of stake
    FAQ above still make it expensive to manipulate the randomness, as
    data from all validators is mixed into the seed and making any
    manipulation requires either universal collusion or excluding other
    validators outright.

One might argue that the deterministic threshold signature approach
works better in consistency-favoring contexts and other approaches work
better in availability-favoring contexts.

### What are the concerns about sharding through random sampling in a Zamfir model?

In a Zamfir model, the fact that validators are randomly sampled doesn’t
matter: whatever the sample is, either the attacker can bribe the great
majority of the sample to do as the attacker pleases, or the attacker
controls a majority of the sample directly and can direct the sample to
perform arbitrary actions at low cost (O(c) cost, to be precise).

At that point, the attacker has the ability to conduct 51% attacks
against that sample. The threat is further magnified because there is a
risk of cross-shard contagion: if the attacker corrupts the state of a
shard, the attacker can then start to send unlimited quantities of funds
out to other shards and perform other cross-shard mischief. All in all,
security in the Zamfir model is not much better than that of simply
creating O(c) altcoins.

### How can we improve on this?

One mechanism that we can rely on is the existence of outside shards
that we can submit evidence to. If there is a 51% attack against a chain
and data on that chain is unavailable, then we can come up with
challenge-response mechanisms where users (sometimes called
“fishermen”<sup>[13](#ftnt_ref13)</sup> can issue challenges claiming that certain
data is unavailable, and until responses are published users would know
not to trust that chain. We can also try to detect situations where one
chain is “under attack”, and have a global “manager” mechanism crank up
the rewards and penalties on that chain so that continuing the attack
becomes more and more expensive. We can create mechanisms where such a
global mechanism “checkpoints” agreement on the state of a shard,
preventing that shard from regressing to a point before that checkpoint
(the Mauve Paper’s sharding mechanism does this through its finality
betting scheme).

Challenge-response mechanisms generally rely on a principle of
escalation: fact X is initially accepted as true if at least k
validators sign a claim (backed by a deposit) that it is. However, if
this happens, there is some challenge period during which 2k validators
can sign a claim stating that it is false. If this happens, 4k
validators can sign a claim stating that the claim is in fact true, and
so forth until one side either gives up or most validators have signed
claims, at which point every validator themselves checks whether or not
X is true. If X is ruled true, everyone who made a claim saying so is
rewarded and everyone who made a contradictory claim is penalized, and
vice versa. The malicious actors thus lose an amount of money
proportional to the number of actors that they forced to look up whether
or not X is true; this prevents the scheme from being used as a
denial-of-service vector. This scheme can be implemented in such a
structured way, or it can be implemented in a more free-form way via a
prediction market; the Mauve Paper’s finality scheme is intended to do
the latter.

This approach works very well if we are using it for state validation,
and data availability is assumed to be already solved so that
malfeasance can always be proven. However, it has a major weakness if we
use it to verify data availability itself: while data availability can
be proven, if necessary by simply providing the data, data
unavailability at time X can never be proven to validators who can only
perform checks after time X. If the above scheme is applied
directly, malicious actors can publish a block containing some
unavailable data, then allow the challenge-response game to escalate,
then suddenly publish the data, making everyone who had earlier
(correctly) claimed the data was unavailable look like a fool.
Challenging thus becomes a potentially altruistic activity, and so
attackers may well “wear down” challengers with false alarms to the
point where no one bothers challenging anymore, at which point they make
their actual attack.

### What if all validators check data availability for all block headers by randomly sampling only a few pieces of data?

Then attacks where only one piece out of a million is missing could
still go through, and pass nearly all validator checks with very high
probability.

### Require the data to be erasure-coded and have a ZK-SNARK proving that this was done?

Now we’re talking. However, this is a very cryptographically complex
approach, particularly since the fast Fourier transforms involved in
making efficient erasure-encodings are computationally heavy, and the
computation involved in making ZK-SNARKs even more so, so at the end of
the day instead of targeting n = O(c\^2) we’ll be targeting something
like n = O(c\^2 / log\^3(c)), and especially if we’re also targeting
fast block times it’s hard to tell if, after all the overhead, there
will be any real performance gains left.

### Let’s walk back a bit. Do we actually need any of this complexity if we have instant shuffling? Doesn’t instant shuffling basically mean that each shard directly pulls validators from the global validator pool so it operates just like a blockchain, and so sharding doesn’t actually introduce any new complexities?

Kind of. First of all, it’s worth noting that proof of work and simple
proof of stake, even without sharding, both have very low security in a
Zamfir model; a block is only truly “finalized” in the Zamfirian sense
after O(n) time (as if only a few blocks have passed, then the economic
cost of replacing the chain is simply the cost of starting a
double-spend from before the block in question). Casper solves this
problem by adding its finality mechanism, so that the economic security
margin increases exponentially rather than linearly. Validators are
willing to make exponentially large bets because they (i) see that other
validators are making bets, and (ii) have personally verified all state
transitions, so they can conclude that there is no chance that they are
signing on an invalid chain. In a sharded chain, if we want economic
finality then we need to come up with a chain of reasoning for why a
validator would be willing to make a very strong claim on a chain based
solely on a random sample, when the validator itself is convinced that
the Zamfir model is true and so the random sample could potentially be
corrupted.

### How can we use griefing factors to analyze this?

One approach we could take is to create a protocol and present a
strategy under that protocol that reaches finality, show how under
normal conditions this strategy is profit-maximizing, and then also show
that the strategy has a bounded griefing factor. A griefing factor can
be defined roughly as follows: actor A under protocol P with strategy S
has a griefing factor x if malicious actors willing to spend $k can
make the actor lose $k \* x. Note that griefing factors are often
situation dependent and may depend on x, as well as other factors like
the portion of the validator set that the attacker controls, but it
would be nice to show an upper bound on the griefing factor in a given
protocol with a given strategy under any situation.

In non-sharded Casper, one can create such a proof, at least in the case
of a double spend against a chain where the “exponential rush to
economic finality” has already started. A sketch is as follows.
According to the rules of the protocol, for a new chain to reach the
same degree of finality as an existing chain (so that non-colluding
nodes switch to it), there must be at least ⅔ of validators on the new
chain. There must also be at least ⅔ of validators on the old chain, and
so at least ⅓ of validators must be on both chains (they are thus by definition malicious);
we’ll call this fraction ⅓ + p. For the new
chain to get started, at least ⅔ of validators must be on it, and these
⅔ must all be malicious. Hence, when the old chain is discarded, we can
expect it to contain ⅓ + p of malicious validators, and at most ⅓ honest
validators (as the attack requires ⅔ malicious validators to pull off).
We’ve thus established that there are at least as many malicious validators as
honest validators on the old chain.

Because of the mechanics of how honest validators make their bets based
on the sizes of existing bets, we can bound honest validator losses at
1.5x attacker losses; hence, the
griefing factor is 1.5. If we increase the finalization threshold from ⅔
to ¾, then this value can be reduced to 0.67.

In the case of sharded data availability verification, it is not so easy
to do this. The reason is that it is intended that the majority of
validators get their opinions by trusting a small sample, and so the
attacker only needs a small budget to get that small sample to lie, and
thereby trick other validators into making bets that would cause them
great losses. The slow exponential ramping argument does not apply,
because the honest validators reason that past a certain point in the
curve, the other validators are not looking at the data directly -
rather, they are just looking at what other validators did.

Note that this argument applies both in the Zamfir model and the "very very adaptive adversary" model (eg. where an adversary can instantly hack computers but is bounded in the number of computers that it can hack).

### Can we use an honest minority assumption to get around this?

Most likely yes. We can have a protocol where validators only start the
exponential ramp-up process if “chain quality” (that is, the ratio of
blocks in the main chain divided by the total number of blocks being
produced) is greater than ⅞. If chain quality is less than ⅞ for an
extended period of time, then rewards and penalties on that chain are
increased (a balanced-budget condition is met, as the increased rewards
come from the increased penalties), until eventually each block on the
malicious chain drains the attacker’s entire balance, and the attacker
will then lose their entire deposit in O(n) time. Note that this is an
adaptation of [Sztorcian
consensus](http://www.truthcoin.info/papers/truthcoin-whitepaper.pdf),
which also works by “raising the stakes” only in the case of great
contention.

There are still many details to be ironed out in terms of the optimal
way to implement these kinds of protocols, and proving guarantees about
them, but this is one general kind of approach that seems very
promising.

### But doesn’t this still mean that an attacker can consume a small amount of capital to make a single shard work very poorly for a medium amount of time?

Yes.

### So we actually didn’t solve the trilemma, we weaseled out of it by pulling back a bit on the security model?

Kind of. Note that attackers can reduce the “chain quality” of a shard,
but they still can’t finalize any bad state with less than O(n) capital.

### Isn’t this terrible?

Not really. There is one trivial attack by which an attacker can always
burn O(c) capital to temporarily reduce the quality of a shard: spam it
by sending transactions with high transaction fees, forcing legitimate
users to outbid you to get in. This attack is unavoidable; you could
compensate with flexible gas limits, and you could even try “transparent
sharding” schemes that try to automatically re-allocate nodes to shards
based on usage, but if some particular application is non-parallelizable,
Amdahl’s law guarantees that there is nothing you can do. The attack that is
opened up here (reminder: it only works in the Zamfir model, not
honest/uncoordinated majority) is arguably not substantially worse than
the transaction spam attack. Hence, we've reached the known limit for 
the security of a single shard, and there is no value in trying to
go further.

### You mentioned transparent sharding. I’m 12 years old and what is this?

Basically, we do not expose the concept of “shards” directly to
developers, and do not permanently assign state objects to specific
shards. Instead, the protocol has an ongoing built-in load-balancing
process that shifts objects around between shards. If a shard gets too
big or consumes too much gas it can be split in half; if two shards get
too small and talk to each other very often they can be combined
together; if all shards get too small one shard can be deleted and its
contents moved to various other shards, etc.

Imagine if Donald Trump realized that people travel between New York and
London a lot, but there’s an ocean in the way, so he could just
take out his scissors, cut out the ocean, glue the US east coast and
Western Europe together and put the Atlantic beside the South Pole -
it’s kind of like that.

### What are some advantages and disadvantages of this?

-   Developers no longer need to think about shards
-   There’s the possibility for shards to adjust manually to changes in
    gas prices, rather than relying on market mechanics to increase gas
    prices in some shards more than others
-   There is no longer a notion of reliable co-placement: if two
    contracts are put into the same shard so that they can interact with
    each other, shard changes may well end up separating them
-   More protocol complexity

The co-placement problem can be mitigated by introducing a notion of
“sequential domains”, where contracts may specify that they exist in the
same sequential domain, in which case synchronous communication between
them will always be possible. In this model a shard can be viewed as a
set of sequential domains that are validated together, and where
sequential domains can be rebalanced between shards if the protocol
determines that it is efficient to do so.

### How would synchronous cross-shard messages work?

The process becomes much easier if you view the transaction history as
being already settled, and are simply trying to calculate the state
transition function. There are several approaches; one fairly simple approach can be described
as follows:

-   A transaction may specify a set of shards that it can operate in
-   In order for the transaction to be effective, it must be included at
    the same block height in all of these shards.
-   Transactions within a block must be put in order of their hash (this
    ensures a canonical order of execution)

A client on shard X, if it sees a transaction with shards (X, Y),
requests a Merkle proof from shard Y verifying (i) the presence of that
transaction on shard Y, and (ii) what the pre-state on shard Y is for
those bits of data that the transaction will need to access. If then
executes the transaction and commits to the execution result. Note that
this process may be highly inefficient if there are many transactions
with many different “block pairings” in each block; for this reason, it
may be optimal to simply require blocks to specify sister shards, and
then calculation can be done more efficiently at a per-block level. This
is the basis for how such a scheme could work; one could imagine more
complex designs. However, when making a new design, it’s always
important to make sure that low-cost denial of service attacks cannot
arbitrarily slow state calculation down.

### What about semi-asynchronous messages?

Vlad Zamfir created a scheme by which asynchronous messages could still
solve the “book a train and hotel” problem. This works as follows. The
state keeps track of all operations that have been recently made, as
well as the graph of which operations were triggered by any given
operation (including cross-shard operations). If an operation is
reverted, then a receipt is created which can then be used to revert any
effect of that operation on other shards; those reverts may then trigger
their own reverts and so forth. The argument is that if one biases the
system so that revert messages can propagate twice as fast as other
kinds of messages, then a complex cross-shard transaction that finishes
executing in K rounds can be fully reverted in another K rounds.

The overhead that this scheme would introduce has arguably not been
sufficiently studied; there may be worst-case scenarios that trigger
quadratic execution vulnerabilities. It is clear that if transactions
have effects that are more isolated from each other, the overhead of
this mechanism is lower; perhaps isolated executions can be incentivized
via favorable gas cost rules. All in all, this is one of the more
promising research directions for advanced sharding.

### What are guaranteed cross-shard calls?

One of the challenges in sharding is that when a call is made, there is
by default no hard protocol-provided guarantee that any asynchronous
operations created by that call will be made within any particular timeframe, or even made at all;
rather, it is up to some
party to send a transaction in the destination shard triggering the
receipt. This is okay for many applications, but in some cases it may be
problematic for several reasons:

-   There may be no single party that is clearly incentivized to trigger
    a given receipt. If the sending of a transaction benefits many
    parties, then there could be **tragedy-of-the-commons effects** where
    the parties try to wait longer until someone else sends the
    transaction (ie. play "chicken"), or simply decide that sending the transaction is not
    worth the transaction fees for them individually.
-   **Gas prices across shards may be volatile**, and in some cases
    performing the first half of an operation compels the user to
    “follow through” on it, but the user may have to end up following
    through at a much higher gas price. This may be exacerbated by DoS
    attacks and related forms of **griefing**.
-   Some applications rely on there being an upper bound on the
    “latency” of cross-shard messages (eg. the train-and-hotel example).
    Lacking hard guarantees, such applications would have to have
    **inefficiently large safety margins**.

One could try to come up with a system where asynchronous messages made
in some shard automatically trigger effects in their destination shard
after some number of blocks. However, this requires every client on each
shard to actively inspect all other shards in the process of calculating
the state transition function, which is arguably a source of
inefficiency. The best known compromise approach is this: when a receipt
from shard A at height `height_a` is included in shard B at height
`height_b`, if the difference in block heights exceeds `MAX_HEIGHT`, then
all validators in shard B that created blocks from `height_a + MAX_HEIGHT + 1` to
`height_b - 1` are penalized, and this penalty increases exponentially. A
portion of these penalties is given to the validator that finally
includes the block as a reward. This keeps the state transition function
simple, while still strongly incentivizing the correct behavior.

### Wait, but what if an attacker sends a cross-shard call from every shard into shard X at the same time? Wouldn’t it be mathematically impossible to include all of these calls in time?

Correct; this is a problem. Here is a proposed solution. In order to
make a cross-shard call from shard A to shard B, the caller must
pre-purchase “congealed shard B gas” (this
is done via a transaction in shard B, and recorded in shard B).
Congealed shard B gas has a fast demurrage rate: once ordered, it loses
1/k of its remaining potency every block. A transaction on shard A can
then send the congealed shard B gas along with the receipt that it
creates, and it can be used on shard B for free. Shard B blocks allocate
extra gas space specifically for these kinds of transactions. Note that
because of the demurrage rules, there can be at most GAS\_LIMIT \* k
worth of congealed gas for a given shard available at any time, which
can certainly be filled within k blocks (in fact, even faster due to
demurrage, but we may need this slack space due to malicious
validators). In case too many validators maliciously fail to include
receipts, we can make the penalties fairer by exempting validators who
fill up the “receipt space” of their blocks with as many receipts as
possible, starting with the oldest ones.

Under this pre-purchase mechanism, a user that wants to perform a
cross-shard operation would first pre-purchase gas for all shards that
the operation would go into, over-purchasing to take into account the
demurrage. If the operation would create a receipt that triggers an
operation that consumes 100000 gas in shard B, the user would pre-buy
100000 \* e (ie. 271818) shard-B congealed gas. If that operation would
in turn spend 100000 gas in shard C (ie. two levels of indirection), the
user would need to pre-buy 100000 \* e\^2 (ie. 738906) shard-C congealed
gas. Notice how once the purchases are confirmed, and the user starts
the main operation, the user can be confident that they will be
insulated from changes in the gas price market, unless validators
voluntarily lose large quantities of money from receipt non-inclusion
penalties.

### Congealed gas? This sounds interesting for not just cross-shard operations, but also reliable intra-shard scheduling

Indeed; you could buy congealed shard A gas inside of shard A, and send
a guaranteed cross-shard call from shard A to itself. Though note that
this scheme would only support scheduling at very short time intervals,
and the scheduling would not be exact to the block; it would only be
guaranteed to happen within some period of time.

### Does guaranteed scheduling, both intra-shard and cross-shard, help against majority collusions trying to censor transactions?

Yes. If a user fails to get a transaction in because colluding
validators are filtering the transaction and not accepting any blocks
that include it, then the user could send a series of messages which
trigger a chain of guaranteed scheduled messages, the last of which
reconstructs the transaction inside of the EVM and executes it.
Preventing such circumvention techniques is practically impossible
without shutting down the guaranteed scheduling feature outright and
greatly restricting the entire protocol, and so malicious validators
would not be able to do it easily.

### Could sharded blockchains do a better job of dealing with network partitions?

The schemes described in this document would offer no improvement over
non-sharded blockchains; realistically, every shard would end up with
some nodes on both sides of the partition. There have been calls (eg.
from [IPFS’s Juan
Benet](https://www.youtube.com/watch?v=cU-n_m-snxQ))
for building scalable networks with the specific goal that networks can
split up into shards as needed and thus continue operating as much as
possible under network partition conditions, but there are nontrivial
cryptoeconomic challenges in making this work well.

One major challenge is that if we want to have location-based sharding so that geographic network partitions minimally hinder intra-shard
cohesion (with the side effect of having very low intra-shard latencies
and hence very fast intra-shard block times), then we need to have a way
for validators to choose which shards they are participating in. This is
dangerous, because it allows for much larger classes of attacks in the
honest/uncoordinated majority model, and hence cheaper attacks with
higher griefing factors in the Zamfir model. Sharding for geographic
partition safety and sharding via random sampling for efficiency are two
fundamentally different things.

Second, more thinking would need to go into how applications are
organized. A likely model in a sharded blockchain as described above is
for each “app” to be on some shard (at least for small-scale apps);
however, if we want the apps themselves to be partition-resistant, then
it means that all apps would need to be cross-shard to some extent.

One possible route to solving this is to create a platform that offers
both kinds of shards - some shards would be higher-security “global”
shards that are randomly sampled, and other shards would be
lower-security “local” shards that could have properties such as
ultra-fast block times and cheaper transaction fees. Very low-security
shards could even be used for data-publishing and messaging.

### What are the unique challenges of pushing scaling past n = O(c\^2)?

There are several considerations. First, the algorithm would need to be
converted from a two-layer algorithm to a stackable n-layer algorithm;
this is possible, but is complex. Second, n / c (ie. the ratio between
the total computation load of the network and the capacity of one node)
is a value that happens to be close to two constants: first, if measured in blocks, a timespan
of several hours, which is an acceptable “maximum security confirmation
time”, and second, the ratio between rewards and deposits (an early
computation suggests a 32 ETH deposit size and a 0.05 ETH block reward
for Casper). The latter has the consequence that if rewards and
penalties on a shard are escalated to be on the scale of validator
deposits, the cost of continuing an attack on a shard will be O(n) in
size.

Going above c\^2 would likely entail further weakening the kinds of
security guarantees that a system can provide, and allowing attackers to
attack individual shards in certain ways for extended periods of time at
medium cost, although it should still be possible to prevent invalid
state from being finalized and to prevent finalized state from being
reverted unless attackers are willing to pay an O(n) cost. However, the
rewards are large - a super-quadratically sharded blockchain could be
used as a general-purpose tool for nearly all decentralized
applications, and could sustain transaction fees that makes its use
virtually free.

* * * * *
<a name="footnotes"></a>

1. <a name="ftnt_ref1"></a> Merklix tree == Merkle Patricia tree

2. <a name="ftnt_ref2"></a> Later proposals from the NUS group do manage to shard
state; they do this via the receipt and state-compacting techniques that
I describe in later sections in this document.

3. <a name="ftnt_ref3"></a> There are reasons to be conservative here.
Particularly, note that if an attacker comes up with worst-case
transactions whose ratio between processing time and block space
expenditure (bytes, gas, etc) is much higher than usual, then the system
will experience very low performance, and so a safety factor is
necessary to account for this possibility. In traditional blockchains,
the fact that block processing only takes \~1-5% of block time has the
primary role of protecting against centralization risk but serves double
duty of protecting against denial of service risk. In the specific case
of Bitcoin, its current worst-case [known quadratic execution
vulnerability](https://bitcoin.org/en/bitcoin-core/capacity-increases-faq#size-bump) arguably
limits any scaling at present to \~5-10x, and in the case of Ethereum,
while all known vulnerabilities are being or have been removed after the
denial-of-service attacks, there is still a risk of further
discrepancies particularly on a smaller scale. In Bitcoin NG, the need
for the former is removed, but the need for the latter is still there.

4. <a name="ftnt_ref4"></a> A further reason to be cautious is that increased
state size corresponds to reduced throughput, as nodes will find it
harder and harder to keep state data in RAM and so need more and more
disk accesses, and databases, which often have an O(log(n)) access time,
will take longer and longer to access. This was an important lesson from
the last Ethereum denial-of-service attack, which bloated the state by
\~10 GB by creating empty accounts and thereby indirectly slowed
processing down by forcing further state accesses to hit disk instead of
RAM.

5. <a name="ftnt_ref5"></a> In sharded blockchains, there may not necessarily be
in-lockstep consensus on a single global state, and so the protocol
never asks nodes to try to compute a global state root; in fact, in the
protocols presented in later sections, each shard has its own state, and
for each shard there is a mechanism for committing to the state root for
that shard, which represents that shard’s state

6. <a name="ftnt_ref6"></a> \#MEGA

7. <a name="ftnt_ref7"></a> If a non-scalable blockchain upgrades into a scalable
blockchain, the author’s recommended path is that the old chain’s state
should simply become a single shard in the new chain.

8. <a name="ftnt_ref8"></a> For this to be secure, some further conditions must be satisfied; particularly, the proof of work must be non-outsourceable in order to prevent the attacker from determining which <i>other miners' identities</i> are available for some given shard and mining on top of those.

9. <a name="ftnt_ref9"></a> Recent Ethereum denial-of-service attacks have proven
that hard drive access is a primary bottleneck to blockchain
scalability.

10. <a name="ftnt_ref10"></a> You could ask: well why don’t validators fetch Merkle
proofs just-in-time? Answer: because doing so is a \~100-1000ms
roundtrip, and executing an entire complex transaction within that time
could be prohibitive.

11. <a name="ftnt_ref11"></a>  One hybrid solution that combines the normal-case
efficiency of small samples with the greater robustness of larger
samples is a multi-layered sampling scheme: have a consensus between 50
nodes that requires 80% agreement to move forward, and then only if that
consensus fails to be reached then fall back to a 250-node sample. N =
50 with an 80% threshold has only a 8.92 \* 10-9 failure rate even
against attackers with p = 0.4, so this does not harm security at all
under an honest or uncoordinated majority model.

12. <a name="ftnt_ref12"></a> The probabilities given are for one single shard;
however, the random seed affects O(c) shards and the attacker could
potentially take over any one of them. If we want to look at O(c) shards
simultaneously, then there are two cases. First, if the grinding process
is computationally bounded, then this fact does not change the calculus
at all, as even though there are now O(c) chances of success per round,
checking success takes O(c) times as much work. Second, if the grinding
process is economically bounded, then this indeed calls for somewhat
higher safety factors (increasing N by 10-20 should be sufficient)
although it’s important to note that the goal of an attacker in a
profit-motivated manipulation attack is to increase their participation
across all shards in any case, and so that is the case that we are
already investigating.

13. <a name="ftnt_ref13"></a> See [Ethcore’s Polkadot
paper](https://github.com/polkadot-io/polkadotpaper/raw/master/PolkaDotPaper.pdf) for
further description of how their “fishermen” concept works.