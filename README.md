
drltc's BitShares Engineering Recommendations

2014-03-22

Author:  drltc (I prefer to remain pseudonymous)

If you like this paper, please send:

PTS  PZxpdC8RqWsdU3pVJeobZY7JFKVPfNpy5z

Introduction
------------

This whitepaper is in response to the controversy caused by bytemaster's mining proposal thread [1].  In a moment, I will provide
a list of design features which I believe BitShares X should have.  The purpose of this whitepaper is to establish the desirability
and technical feasibility of each of these features.  I hope that I have chosen a set of features whose desirability is obvious,
and will focus mainly on technical feasibility.  However, in this section I will provide a brief explanation and justification of
each feature's desirability.

Here are the features of my proposal.

F1.  *Blocks are minted, not mined.*  Nobody really wants to have mining in BitShares, but bytemaster felt compelled to introduce it because
(1) certain technical issues arose and/or (2) the mining algorithm encourages (F3).  See the next section for more about
my definitions of mining and minting.

F2.  *Coins are neither minted nor mined.*  A fundamental feature of the BitShares X social contract is a promise to PTS / AGS investors
that they will not be diluted by future issuance of XTS.  I would not propose a design which breaks that promise.

F3.  *Nodes are incentivized to stay online and regularly sign off on the chain.*  The more owners' approval the chain has, the harder it
becomes for the blockchain to be hijacked by a minority with different interests from the owners.

F4.  *If a node is online and minting, over the long run, its chance of creating a new block is proportional to its wallet balance.*
If the owners have responsibility for extending the blockchain, then it's only fair to distribute that responsibility to owners
proportionally to their ownership.

F5.  *If a node is online and minting, over the long run, its expected income is proportional to its wallet balance, ceteris paribus.*
If the owners have responsibility for extending the blockchain, then it's only fair to distribute the income produced by extending the
blockchain proportionally to their ownership.  "Ceteris paribus" in this context means that the percentage of other nodes
that are online and minting remains constant, and the average fee per block also remains constant.

F6.  *Block difficulty is adjusted so that blocks are produced at a constant rate.*  This is a necessary design feature of all
reasonable cryptocurrencies, an important mechanism to regulate transaction times, blockchain storage, and network bandwidth.

F7.  *Minting reward increases proportionally to fees.*  Since BitShares can't be created to pay minters (F2), all revenue for
minting must come from fees.

F8.  *Even if there are no transactions, there will be a nonzero minting reward.*  In times of light transaction load, the
network should continue to produce blocks.  These blocks should be incentivized.

F9.  *Small balances should be first-class citizens, including with respect to minting rewards.*  The BitShares X design
should protect small owners who are large enough for legitimate economic activity.  The best way to prevent mining
pools from forming is to make sure there is little economic incentive in doing so.

F10. *Objectives must be achieved within real-world engineering constraints.*  This whitepaper is intended to serve as
an implementation guide for bytemaster and the BitShares X team.  Theory is important, especially when it provides good
reasons to prefer one design choice over another.  But I will not allow any of the theory in this paper to make
unrealistic assumptions.

[1] TODO

Mining vs. minting (F1, F2)
---------------------------

"Mining" means the network produces blocks in such a way that buying more hardware increases the rate at which you produce blocks.  "Minting"
means the network produces blocks in such a way that buying hardware beyond a recent desktop PC with reasonable hardware specs will not increase
the rate at which you produce blocks.

"Minting" and "mining" are used to describe two distinct features of existing cryptocurrencies:  The decentralized selection of a
single node which is allowed to produce a block, and the creation of new coins that did not exist before.  By "mining/minting a block"
I'm referring only to the first feature (selecting a node to produce a block), and by "mining/minting coins" I'm referring only to the
second feature (creating new coins). The second feature exists in most current cryptocurrencies, so I may occasionally refer to it
in comparisons.  But creating new coins is not a feature that BitShares will have.

However, BitShares still has a minting reward.  By "mining/minting reward," I mean the coins given to a node by the network when
it mines/mints a block.  In most cryptocurrencies, this reward comes from a combination of transaction fees and newly minted
coins.  In BitShares X, the minting reward comes solely from transaction fees.  So even though I talk about "minting reward," I'm
not implying BitShares will ever create new coins!

Pools (F9)
----------

Small owners are important.  By "small owners," I'm not talking about "dust," economically useless amounts of a few satoshi.
"Dust" is annoying even to its owners and wasteful of limited resources.  I'm talking about owners bigger than "dust" but
still too small to mine effectively. E.g., if BitShares X achieves a market cap of $100 million and has ~100,000 blocks
per year, then small-time owners with less than $1000 worth of XTS will have a mining rate of less than one block a year
(with correspondingly high variance).

If small balances are treated as second-class citizens with respect to mining/minting rewards, then they will have an
incentive to form pools.  Bitcoin and other proof-of-work currencies are already struggling with the centralization created
by pools.  Pools can be even worse for proof-of-stake currencies:  If a proof-of-stake cryptocurrency's design requires a
mining pool to have the private keys which allow it to control its participants' assets, this creates a dangerous fiduciary
relationship.  In other words, there is no technical mechanism to prevent the pool operators from taking the participants'
funds and disappearing. (Of course, if a thieving pool's real-world presence can be found, they are still subject to
real-world checks and balances like lawsuits, reputation damage, or even physical threats from scammed participants.)

The potential damage a scamming pool operator can cause may be much greater for proof-of-stake than proof-of-work.
PoW only requires pool operators to have control of participants' *income*, not their *assets*.  Cautious PoW
pool participants can make frequent withdrawals so the pool never has control of a significant percentage of their
assets.  Most PoW pools can automate this process, and many require their users to do so; ethical
pool operators don't actually *want* fiduciary responsibility for large balances!  But if, for technical reasons,
a PoS coin's design requires a pool operator to have control over their participants' entire balances, none of the
above applies to a PoS coin.

Hash streams (F1, F4)
---------------------

Now let's get to the specifics of my proposed minting algorithm.  The most fundamental feature is this:
*Each transaction output produces a hash stream*.  The rate at which these hashes are produced is a global parameter
R_stream = 20 hashes / second.  A stream is only computed by the node that owns the output.  And it is possible to
compute the streams for many, many outputs on an ordinary CPU.

Practical hash streams (F1, F4)
-------------------------------

Now let's look at how a transaction output generates a hash stream in practice.  The hash produced at time t is equal to
H(txid, output_index, shiftreg, t) where H is a cryptographically sound hash function.

- txid : The transaction which produced the output
- output_index : The index of the output
- shiftreg : 128-bit string which contains the least significant bit of the block hash of 128 blocks, beginning SHIFT_DELAY = 6 blocks before the current block.
- t : Current time, measured in number of R_stream ticks since beginning of previous block's minting period.

Three of these four parameters are included in the hash for quite obvious reasons:  Including txid and output_index ensures that every output
produces a different hash stream.  Including t in the hash computation is how the stream advances.

A block's "minting period" is simply the time during which a successor block can be minted.  Since a 1-minute spacing between blocks is enforced,
t=0 means "60 seconds after the previous block", t=1 means "60.05 seconds after the previous block", etc.

Stream selection attack (F1, F4)
--------------------------------

The non-obvious item is shiftreg.  It's there to thwart an attack I call the "stream selection attack."  The value of a hash stream is
determined by its time-to-pass.  Creating different txid's, and thus different hash streams, is very cheap if you don't need to
broadcast them [1].  Thus, an attacker with lots of hash power might attempt to search for a valuable txid which produces
a stream with an unusually short time-to-pass.  A stream selection attack's "amplification factor" is the number of streams it can
consider; its "gain" is defined as log_2(amplification_factor).  Without shiftreg, the gain would be limited only by the hardware
hash rate an attacker has available.  With shiftreg, to get k bits of gain from the attack, an attacker must mine k+SHIFT_DELAY+1
consecutive blocks.  Very weak stream selection attacks are possible, but much capital will be required, and success will be modest and
infrequent.

For full protection against stream selection attacks, shiftreg should contain a full 128 bits.  Thus, an output's hash stream begins
128 blocks after the output is produced.  This restriction does not apply to genesis block outputs (otherwise it would be impossible
to mint blocks at the very beginning).  [2]

As I noted in a different thread [3], R = h*c*f(t) is the only way to go.  However, differently from my post in that thread, using a hash
stream means everyone's hash power h is limited to 20 hashes per second.

[1] For example, an attacker with one million satoshi could compute the hashes of one million potential
transactions to himself, T_1 ... T_1000000, where T_n sends n satoshi to himself, then publish whichever transaction creates a hash
stream that has a favorable value early on.  If none of these million txid's produce a hash stream to his liking, he could simply generate
a different recipient address and repeat the procedure.

[2] The genesis block is thus vulnerable to stream selection attacks.  This can be mitigated by
publishing the data and code used to produce the genesis block, to prove that the genesis block creators didn't intentionally
select txid's which favor particular account(s).

[3] TODO

Hash stream modulation
----------------------

Hash streams are "modulated."  By "modulated," I mean hashes produced by a hash stream are compared against a difficulty target;
only hashes smaller than the target are allowed through.  The specific modulation I use is the key insight which enables
my novel algorithm.

Specifically, the modulation difficulty has a negative dependency on the t value, and positive dependency on the block
height.  Since modulation difficulty is measured in bits, these are exponential dependencies in "rate space."
At the beginning of the genesis block, the difficulty target is initialized to a ridiculously high value, 2^256.
The difficulty target then decreases exponentially at a rate D_fall bits / minute.  When a new block is discovered,
the modulation difficulty instantly increases by D_rise bits / block.  The D_rise / D_fall ratio gives the
equilibrium block production time in minutes per block.
I suggest D_fall = 1/32 bits / minute and D_rise = 5/32 bits / block for the long term.

For early blocks, these should be bigger, e.g. D_fall_initial = 8 bits / minute and D_rise_initial = 40 bits / block
for the first block after the genesis block.  Then D_fall should decrease linearly by, say, 1/128 bits / minute every
block, and D_rise proportionally by 5/128 bits / block, so they will reach their long-term values at block 1020 --
about 3.5 days from the beginning.  This should help compensate for the fact that it's hard to guess, a priori,
how much ratepower will appear on the early network, and be more flexible dealing with fluctuations early on.

The modulation difficulty should not be allowed to fall too far.  Thus there is a maximum threshold D_maxfall which is
the greatest difficulty change permitted per block.  I propose D_maxfall = 1/4 bits / block.  Without this backstop,
the difficulty would happily adjust nearly instantaneously to any sudden rate drop.  While this sounds ideal in principle,
in practice it means a small network on the wrong side of a network split would go on producing its own blocks at
the same rate as the main network (and reconciling the fork afterward would be Fun in the Dwarf Fortress sense).

The backstop should also be increased in early blocks, perhaps to 4 bits or so, and then decrease linearly.

Minting using hash stream (F1, F4)
----------------------------------

Minting a new block happens when a transaction output's modulated hash stream produces a passing hash.  The transaction
output will be called the "mintstake."  The "mintstake's private key" will be needed; this key is simply the private
key that owns the mintstake.

First, the new block's body is produced (transactions).  The new block's header includes a hash of the new block's
body (e.g., the body hash might be a Merkle root hash of a tree with the transactions as leaves), and a hash of
the parent block.

To produce a proof-of-stake, the minter first creates a claim that the mintstake's modulated hash stream produced a
passing hash.  The claim includes t, txid and output_index.  The minter then computes H(claim, parent_block, body)
and signs it with the mintstake's private key, producing a mintstake signature.

Once a claim is published, each node which receives the claim must verify it.  (With the usual p2p protections against
malicious nodes, e.g. not broadcasting false or unverified claims, and banning nodes that produce false claims.)
The verification consists of the following steps:

- t must be within bounds (a certain minimum time past the previous block's t, and not later than the local system clock)
- The hash stream output H(txid, output_index, shiftreg, t) passes the modulation difficulty.
- The mintstake referred to by txid, output_index exists and is spendable.
- The mintstake is either a genesis block output, or is at least 128+SHIFT_DELAY blocks old.
- The mintstake is owned by the key used to produce the mintstake signature.
- The first transaction in the current block spends the mintstake.

Note that shiftreg and the modulation difficulty must be independently computed by each node.

Minting multiple branches (F1, F4)
----------------------------------

Due to SHIFT_DELAY, a minter who wins the race to produce a block on one branch of a fork may win the race on all branches.
Since minters are uncertain which branch will ultimately be adopted, they will have an incentive to mint on all the branches
they know of.  If enough minters adopt this strategy, the branches of a fork may grow equally, and the fork may remain
unresolvable.

Obviously this is an undesirable situation.  Fortunately, a simple solution is possible.  Any re-use of a mintstake produces
multiple H(claim, parent_body, block), signed by the same key, with identical claims and distinct values for
parent_body, block, or both.  This situation can be detected automatically.  Moreover, the values constitute irrefutable evidence
that the private key involved authorized the balance to be mis-used in a deliberate attempt to cause or prolong a fork.
Therefore, we can impose a consequence on this attacker!

What sort of consequence?  Well, the most serious would be to confiscate the offending mintstake, or even all funds controlled
by the offending private key!  However, an attacker could easily minimize the impact of these punishments by splitting up his
funds between multiple addresses ahead of time.  Worse yet, it might be possible for an oblivious user to unintentionally
trigger the consequences by minting with multiple instances of the same wallet.

Because of the potential for unintentional triggering, I think the consequences of this attack should be limited to a big fee
and locking the balance for several hours.  This should be more than enough to deter attackers without severe, disproportionate
punishment to users who have no malicious intent, merely somewhat lax configuration (having multiple clients minting with
the same private keys).  And this also suggests that minting should be restricted to balances that are big enough to at
least cover this potential fee; thus an attacker can't split up his holdings into multiple balances to reduce his
punishment.

As a safeguard against accidental claim re-use, minters should track their past minting actions in a local database and
refuse to create a mintstake that would re-use a claim.

Fork resolution (F1, F4)
------------------------

If a minter is aware of multiple branches, he should mint on the longer branch.  If both branches are of equal length,
the tie must be broken.  To break the tie, a minter should choose the branch with the smaller t value.  In the case
that two t values are tied, the tie should be broken in favor of the larger hash value.

Boost curve (F1, F5, F6)
------------------------

A given output's block production rate should be proportional to:

    R = c * f(age)

where c is the number of coins, and f(age) is a time-dependent boost function.
As protocol designers, we can design f(age) to have whatever characteristics we want.  The faster f(age)
increases, the more advantage ancient balances have against recent balances.  Also, f should be continuous
and not increase too quickly; otherwise the total block production rate might change too quickly for the
global boost factor to compensate.

This is implemented by simply multiplying the difficulty target by R.  Compensating for adjustments due
to rate fluctuation will be efficiently carried out by the minting algorithm.

Specific boost curve (F1, F5)
-----------------------------

I've sketched out qualitatively what I think would make a good boost curve.  (The drawing is not to scale.)
From t0 to t1, 134 blocks, shift_reg is not yet full, so minting cannot take place.  From t1 to t2, about
30 days, minting rate increases.  Starting from zero with zero slope means balances that have moved
recently are still heavily penalized, but the penalty gradually dissipates.  Once at t2, the slope starts
to decrease, although the boost is still modestly increasing.  This way, older balances will have an
advantage against younger balances -- minters who wait patiently should see some increase in their
probability.  Then at t3, near the end of the year (about 10 months), the minting rate begins to decrease.
At t4 (about 10.5 months), the boost rapidly decays, and at t5 (about 11 months), the boost is zero again.
This is to encourage minters to perform the required annual transaction before the last possible minute
(but remain continuous so there is no sudden dropoff).

Finding a good approximation of this curve is left as an exercise to the reader (hint:
try piecewise cubic polynomials).  The curve should have a simple implementation using purely integer
arithmetic.

Rewards (F2, F7, F8)
--------------------

So far, I've only talked about the rate of producing blocks.  How do block rewards happen?

Since coins are never created (F2), the only possible source for block rewards are transaction
fees.  I think that exactly what fees exist, and how much they will be, is something that is
easily understood and adequately addressed by current BitShares development efforts.

So I will focus on the question of what happens to the fees after the fee money is collected:
How the fees are distributed.

The BitShares designs I've seen so far all suggest paying all of a block's fees immediately
to its miner / minter.  However, we desire blocks to continue to be produced even when there
happen to be no fee-paying transactions.  But this produces a new problem:  Where do the
minting rewards for no-fee blocks come from?

The answer is to avoid paying out all the fees immediately.  Rather, a percentage of the fees
goes to the minter as an immediate reward to incentivize the inclusion of fee-paying transactions,
and a complementary percentage goes to a reserve.  Then a percentage of the reserve goes to the
current minter.  That is:

    imm_reward = IMM_REWARD_FACTOR * block[i].total_fees
    res_income = block[i].total_fees - imm_reward
    res_reward = RES_REWARD_FACTOR * (reserve[i-1] + res_income)
    reserve[i] = reserve[i-1] + res_income - res_reward
    reward = imm_reward + res_reward

Let's check that no coins are created or destroyed by this scheme.  Beforehand the assets
available to pay the minter reward are:

    available_assets = reserve[i-1] + block[i].total_fees

Afterward these assets are re-distributed as reserve[i] + reward; simplifying this shows
that no new coins were created:

    reserve[i] + reward = reserve[i-1] + res_income - res_reward + imm_reward + res_reward
                        = reserve[i-1] + block[i].total_fees - imm_reward - res_reward + imm_reward + res_reward
                        = reserve[i-1] + block[i].total_fees

The IMM_REWARD_FACTOR should be large enough that minters have an immediate incentive to
include fee-paying transactions.  The RES_REWARD_FACTOR determines how long coins that
go to the reserve are taken out of circulation.  I suggest thinking about RES_REWARD_FACTOR
in terms of the reserve half-life, i.e. how long it will take the reserve to lose half
its value if no fee-paying transactions appear.  I think RES_REWARD_FACTOR should be chosen
to create a half-life of approximately one month.

Other reserve activities
------------------------

It is possible to use some of the money from the reserve for tasks other than paying minting rewards.
For example, dividends can be implemented by destroying some percentage of the reserve every block.

In the next sections, I will recommend protocols for an insurance pool for short sales,
and a signoff lottery to encourage idle balances to participate in securing the blockchain.

Insured shorts
--------------

Much forum controversy has been devoted to the problem of "underwater" or "uncollateralized" shorts.
Suppose Alice sells BitUSD short using 100 XTS as collateral.  Then suppose the price of BitUSD
rises so buying back the BitUSD will cost more than the available 100 XTS (and the market was
volatile enough that Alice's position couldn't be automatically liquidated during the rise).

I think the BitUSD should be repurchased by joining the 100 XTS collateral with
reserve funds to purchase the required amount of XTS.  Alice won't get a single satoshi of
her 100 XTS collateral, but the negative BitUSD balance will disappear from her books when
the liquidation occurs.  I think it would be best to keep an insurance reserve for this
purpose, separate from the mining reserve above.

A desired capitalization level for the insurance reserve can be calculated by establishing
a black swan price multiplier, k_swan.  The reserve must be able to completely collateralize
shorts in the event the price is instantaneously multiplied by k_swan.  If the reserve's
capital is insufficient, dividends should be reduced or eliminated and used to increase the
reserve instead.  If the insurance reserve is overcapitalized, its income should instead go
to dividends.  If the insurance reserve is ridiculously overcapitalized, it should decay
exponentially (i.e. produce dividends from its assets).  I propose k_swan = 2, with
capital that would be unused even in the k_swan = 3 case being converted to dividends
with a half-life of one month.

The reserve's market activities should be capped at a fraction of the available
market depth and a fraction of the reserve balance (to try to avoid paying
extortionate prices or exhausting the reserve during a short-lived fluctuation).

The above limits mean that sometimes the reserve won't be able to immediately liquidate
all undercapitalized shorts.  In that instance, a choice must be made about which
shorts should be liquidated.  I propose that the shorts should be put in a sorted queue,
with the most badly undercapitalized shorts liquidated first.  The reason?
Liquidating the entire queue will take potentially many blocks.  The price will move in
the meantime.  If the price moves downward, the reserve will get lucky and the shorts
that are only marginally undercapitalized may become fully capitalized and not need
any insurance funds at all.

Since insurance funds are removed from circulation, reserve income is
equivalent to a dividend as long as the insurance is not needed.  Indeed, my proposal
can be interpreted as "insurance funds are really dividends, and insurance payouts
are really newly printed money."  But this interpretation doesn't quite capture the
fact that the insurance fund can be exhausted, while the printing press cannot.

Suppose the reserve is wiped out during an extreme market event.  If the event ends
and the exchange rate stabilizes, the reserve will still get income from fees,
and continue to repurchase undercapitalized positions, though the rate at which
repurchases occur may be slow enough that it will take months or years before all
the undercapitalized positions created in the event can be repurchased.  In that
case, BitAsset holders may experience some inflation in the short to medium term,
but they can be assured that in the long term, the backing will eventually be
restored.

The insurance fund approach strikes a balance between the interests of BitAsset
holders ("We want fully collateralized assets"), XTS holders ("Why should we
bail out the owners of some random BitAsset we don't hold long, short, or sideways
and TBH couldn't care less about?"), and AGS/PTS investors ("We were promised that
our genesis block BitShares will never be diluted").  The reserve's limited funds
serve as a natural check against the system robbing XTS holders blind in favor
of BitAsset holders in extreme conditions, while still providing BitAsset holders
assurances that every reasonable effort will be made to protect their BitAssets'
backing.  And genesis block investors will be happy because the no-printing promise
will be kept.

Signoffs (F3, F9, F10)
----------------------

Nodes should be incentivized to stay online and regularly sign off on the chain (F3).
This was a major motivation for bytemaster's proposal to introduce mining to BitShares.
It suggests a particularly interesting problem:  How do n private keys indicate they
approve the state of the blockchain, without requiring O(n) blockchain storage space
for signatures?

I'm not sure I can see a colorable solution to this problem in bytemaster's TaPoS
proposal.  The solution I suggested in that thread was a proxy voting system which
would allow miners / minters to sign over the ability to approve blocks, allowing
a single key to control them.  This achieves the objective, but has some drawbacks.

I thought for quite a while about an AMT (alternative minting transaction), by
which a balance too small to mint reliably could request their "fair share" of
the minting income once every 6 months or so.  Unfortunately, this idea is
quite vulnerable to sybils.  An XTS holder rich enough to mint, say, ten times
a year reliably, could simply split up his balance equally into, say, 1000
different addresses.  Then he could accept the minting rewards for the 10
addresses that minted successfully (recall his total minting rate is independent
of how his funds are split up), and claim the AMT for the 990 that didn't
-- getting nearly twice his fair share of minting profits!

With the minting algorithm, proof that large blocks of coins approved the current
blockchain is inherent in the minting difficulty.  So the problem isn't really to
prove that a certain number of balances have approved the blockchain state; minting
already accomplishes that.  Rather, the problem is that minting has a high variance
and long expected completion time for small balances; thus they may rationally
decide not to participate.  (By "small balances," I mean balances that are big
enough to be reasonable and economically meaningful, not "dust.")  Even "dust"
should be encouraged to participate in the transaction, which I'll explain shortly.

In this section I will propose a different solution.  My solution is to allow
"signoffs" -- yea-or-nay approval of blocks -- at a faster rate than block
production.  This idea was originally explored in bytemaster's UNL proposal
(which was fatally flawed for unrelated reasons).

A signoff is a transaction together with a signstake.  Signstakes are produced
by the same algorithm which produces mintstakes, but the rate at which signstakes
are produced is equal to AVG_SIGNOFFS_PER_BLOCK times the block rate.  The
AVG_SIGNOFFS_PER_BLOCK parameter is a trade-off; larger values will require more
blockchain storage, but reduce variance in the return of balances that participate
in securing the network.  I propose AVG_SIGNOFFS_PER_BLOCK = 16.

The reward paid to a signoff is minting_reward / AVG_SIGNOFFS_PER_BLOCK.  The value
minting_reward itself should increase with each signoff (to provide the minter
a stronger incentive to include the signoff than the transaction fee for the
signstake transaction).

As with minting, in order to be valid, a signstake requires a transaction
which spends the output used to produce the signstake to be included in the
block.  This means that, if you operate a node that regularly produces a
signstake, and you want to do a transaction, but are willing to hold off on
submitting it, you can do the transaction you wanted to do anyway as the
signstake transaction the next time you produce a signstake.  It would
also be nice if the client had an option to automatically use signstake
transactions to simplify your blockchain position by e.g. combining the
signstake output with balances near their anniversary.

In order to secure the blockchain, a minimum number of signoff transactions
should be required in each block.  I propose
MIN_SIGNOFFS_PER_BLOCK = AVG_SIGNOFFS_PER_BLOCK / 2.

TODO:  Minting and BitAssets
----------------------------

- Who controls the minting potential of collateral used to back BitUSD?
- BitUSD long holders?  BitUSD short holders?  Nobody?
- Analyze advantages and disadvantages of each approach (implementation difficulty, economics)

TODO:  Minting authorization
----------------------------

- Allow minters to sign over minting power of their BitShares to another address
- Does this change necessity of signoffs?
- Does this change answers in "Minting and BitAssets"?
- Is it possible to limit the size of a minting pool at the network level?
- Disadvantage:  More complicated implementation
- Disadvantage:  More centralization
- Advantage:  Easier to obtain consensus regarding protocol extensions
- Advantage:  Casual minters can outsource responsibility to keep minting 24/7
- Advantage:  Reduces variance
- Advantage:  Reduces need for dangerous fiduciary relationships

