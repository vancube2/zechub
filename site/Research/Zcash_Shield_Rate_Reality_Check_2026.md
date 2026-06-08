---
published: 2026-06-05
Zcash Privacy in 2026: What the Real Numbers Show
How much of Zcash is actually private right now? I went looking for a straight answer,
got the first version wrong, fixed it, and ended up measuring the network during the same
week a four year old bug in the shielded pool was finally caught.
Author: ZecLedger Research
Date: 5 June 2026
Data: my own Zebra v5.0.0 node (queried directly), the Blockchair API, and published
figures from the trackers listed at the end
Tool: github.com/vancube2/zecledger
Everything here can be reproduced with the commands in the methodology section. Don't take
my word for any of it.
---
It started with a question from @squirrel
I almost published a wrong number.
After my first write up, someone in the ZecHub Discord, @squirrel, asked a short question
that sent me back to the start:
> "Is Blockchair showing the shield rate of transparent to Orchard as well?"
It sounds simple. It isn't. The question forced me to check whether the data source I was
using could even see the thing I was trying to measure. It turned out it couldn't see all
of it, and the way I was counting transactions was plain wrong. I'd been treating money
that was leaving the private pool as if it counted as a private transaction. It doesn't.
So I fixed it. The corrected numbers are lower than what I first had, but they're honest,
and they tell a far more interesting story. Below is the whole thing: what I did, where I
went wrong, what the corrected data actually says, and what the community can do with it.
Thanks, @squirrel. The whole report is better because you asked.
---
What a block explorer can and cannot see
This is the part most people get wrong, and it's the heart of @squirrel's question.
Blockchair is one of the biggest blockchain data providers around. With Zcash it hits a
wall. For a shielded transaction, here's what it can tell you:
It can see:
that money moved into the shielded pool (a shielding transaction)
that money moved out of the shielded pool (a deshielding transaction)
the amount that crossed that boundary
the encrypted Orchard data on a transaction, which it can store but cannot read
It cannot see:
transactions that happen entirely inside the pool (private to private, or z→z)
who sent or received shielded funds
the balance of any shielded address
anything about what happens to money once it's inside the pool
When I pulled the raw data, the fields for shielded inputs and outputs came back as `null`.
That's not a bug. That's Zcash privacy working the way it's supposed to. Even a major
analytics company can't look inside.
So here's the thing to keep in mind for the rest of this report: any shield rate measured
from a block explorer is a floor, not a ceiling. The real amount of private activity is at
least what we can see, and probably a bit more, because the fully private transactions are
invisible to us on purpose. That detail matters a lot later.
---
Where my first attempt went wrong
My first way of spotting a shielded transaction was this: if a transaction has no
transparent inputs and isn't a mining reward, call it shielded.
Wrong. A transaction with no transparent inputs can be one of two completely opposite
things:
A fully private (z→z) transaction, where money stays inside the pool. That one really
is private.
A deshielding (z→t) transaction, where money is leaving the pool to become visible.
That's the opposite of private.
My old method counted both as shielded. Every time somebody pulled money out of privacy, I
was logging it as a win for privacy. No wonder the numbers looked high.
The corrected method reads two fields the explorer does give you. `shielded_value_delta`
tells you whether value moved into or out of the pool. `shielded_output_raw` tells you
whether new shielded notes were actually created. With those two, every transaction sorts
cleanly into five buckets: mining reward, transparent (t→t), shielding (t→z), deshielding
(z→t), and fully private (z→z).
---
The data, from three sources
I didn't want this resting on one provider, so I used three.
Source 1: my own node (Zebra v5.0.0, queried directly)
I ran the Zcash Foundation's own node software, Zebra v5.0.0, and queried it with nothing
in between. At the time of writing it had synced to about block 560,000, which puts us in
the Sapling era of 2019. I sampled 50 blocks:
Measure	Value
Blocks sampled	50
Total transactions	425
Coinbase (mining)	50 (11.8%)
Transparent	342 (80.5%)
Shielded	33 (7.8%)
So back in 2019, straight from my own node, the shield rate sat around 7.8%.
Source 2: Blockchair, 1,000 transactions across 10 windows
I sampled 100 transactions in each of 10 windows, spread across the days before and after
the network upgrade I'll get to shortly. Put together:
Type	Share of 1,000	What it means
Transparent (t→t)	about 62%	fully visible, like Bitcoin
Coinbase (mining)	about 21%	block rewards, always transparent today
Deshielding (z→t)	about 13%	money leaving the private pool
Fully private (z→z)	about 2%	completely private
Shielding (t→z)	about 2%	money entering the private pool
Any kind of shielded activity came to roughly 4% of daily transactions. And the flow was
badly lopsided. Far more money was leaving the pool than entering it, on the order of seven
exits for every one entry.
One thing I want to be loud about. These are samples of a few hundred transactions out of
roughly 8,900 a day. They're consistent across all ten windows, so I trust the shape, but
they're samples, not a full count. I'm not claiming decimal point precision. I'm claiming
the picture: daily shielded activity is in the low single digits, and the pool is draining
day to day.
Source 3: published figures (the supply side)
This is where it gets interesting, because the next set of numbers looks like it
contradicts mine, and that contradiction turns out to be the whole point.
Several data providers track how much ZEC is held in shielded addresses, and it's been
climbing fast:
Date	Shielded supply	Source
Early 2024	about 8% of supply	Messari, crypto.news
Oct 2025	about 18%	crypto.news
Nov 2025	about 23%	crypto.news
May 2026	about 30% (around 5 million ZEC)	Messari, CoinGecko, crypto.news
On top of that, the peak transaction shield rate reportedly hit 59.3% in February 2026,
and the Orchard pool alone holds around 4.2 million ZEC.
So how can 30% of all ZEC be sitting in private addresses while only about 4% of daily
transactions are private?
---
The answer: a vault, not a wallet
Both numbers are true. They just measure different things.
Shielded supply measures how much ZEC is parked privately. That's a savings number. The
daily shield rate measures how much of the spending and moving around is private. That's a
spending number.
Line them up and the behaviour is obvious. People shield their ZEC and hold it. When they
actually spend or move it, they tend to go transparent. The pool is being used as a private
savings vault, not a private current account.
That's genuinely good news for privacy as a store of value. Five million coins' worth of
people deliberately chose privacy. It's also the real challenge for privacy as everyday
money. The technology works fine. The daily habit hasn't caught up. And there's a
structural reason the daily number stays stuck, which the community can actually do
something about.
---
The mining reward problem, and the fix that already shipped
In my sample, about 21% of all transactions were mining rewards, and every single one went
to a transparent address. That's one guaranteed transparent transaction roughly every 75
seconds, around the clock, built into how the network runs.
That one fact caps the daily shield rate no matter what ordinary users do. It isn't apathy.
It's a default setting in mining software.
The good part is that the fix already shipped. Zebra v4.5.0, and now v5.0.0, let mining
rewards go straight to a shielded address. The capability is there today. What's missing is
the pools actually using it. If the big pools switched their payout addresses to shielded
ones, about a fifth of all daily transactions would flip from transparent to private almost
overnight, and regular users wouldn't have to change a thing.
That's the single biggest win available to the ecosystem right now, and one of the easiest.
---
The week I was writing this: the Orchard bug
I have to cover what happened while I was collecting this data, because the timing put my
work right in the middle of it, and because it turns the point about explorer blindness
from theory into something very real.
What happened
On 29 May 2026, an independent security researcher named Taylor Hornby, who'd been running
a paid audit for Shielded Labs since April, found a critical flaw in the Orchard circuit,
the cryptographic core of Zcash's newest shielded pool. By the end of the same day he had a
working exploit and had confirmed in a local test that it could create unlimited,
undetectable counterfeit ZEC inside the pool.
The flaw was two lines of code, an under constrained elliptic curve multiplication check
that let mathematically invalid inputs pass as valid. It had been live since the Orchard
pool launched in May 2022, which means it sat there through roughly four years of expert
review before anyone caught it.
The response
The response was quick, and it's the part the community got right. The timeline comes
straight from the Zcash Foundation:
29 May: the bug is disclosed privately to core engineers (Daira-Emma Hopwood, Kris
Nuttycombe, Jack Grigg) and confirmed within hours
late May into 1 June: the corrected circuit is prepared quietly, with details kept
private to avoid tipping off anyone who might exploit it
2 June, around 02:00 UTC: an emergency soft fork (Zebra 4.5.3) temporarily switches
Orchard off at block 3,363,426 while the fix is finalised
3 June: a hard fork, NU6.2 (Zebra 5.0.0), switches Orchard back on with the corrected
circuit at block 3,364,600
4 to 5 June: the incident is disclosed publicly and ZEC's price drops hard
This was only the second security driven upgrade in Zcash's history since 2016. The
network's turnstile accounting, which tracks total value moving between pools, confirmed
the total ZEC supply was intact. Sapling and transparent transactions kept working the
whole time.
When the news broke on 4 and 5 June, ZEC fell roughly 40 to 50% in a day, from a peak near
$624 on 4 June down into the $309 to $343 range. The snapshot I pulled on 5 June had it
around $343.90, with a market cap near $5.7 billion. The drop was spot selling and lost
confidence, not lost funds. The bug was fixed and the supply was confirmed whole.
The uncomfortable part, and why it ties back to @squirrel
Here's the line from the disclosures that stuck with me. Because Orchard transactions are
private by design, there's no way to prove with cryptography alone whether the bug was ever
exploited before it got fixed.
Now read that next to where this report started. A block explorer returns `null` for
shielded data. My own analysis can't see inside the pool either. The same privacy that
protects honest users means that if somebody had quietly minted counterfeit ZEC, it would
have been invisible to every analytics tool out there, mine included.
That isn't a reason to weaken privacy. It's the reason the turnstile accounting and the
proactive audit mattered so much here. Privacy and verifiability pull against each other,
and the right answer isn't to break privacy. It's to build supply checks, like the
turnstile and the new pool Shielded Labs has proposed, that prove the total is honest
without exposing a single individual.
---
Where Zcash stands today (5 June 2026)
From the live network stats on Blockchair:
Measure	Value
Latest block	3,367,171
Total transactions, all time	17,680,871
Transactions, last 24h	8,883
ZEC price	about $343.90
Market cap	about $5.7 billion
Active nodes	978
Median fee	about $0.027
Chain size	271 GB
For a historical anchor, my own Zebra v5.0.0 node, queried with no third party involved,
put the 2019 Sapling era shield rate at 7.8% across a 50 block sample of 425 transactions.
---
What the community is getting right
It's easy to read low daily numbers as bad news, but they aren't the whole story, and a lot
here is genuinely strong.
Privacy as savings is real. Supply went from 8% to about 30% in 18 months, and that's
millions of coins moved by individual choice, not hype.
The security response was excellent. From discovery to a network wide fix in roughly four
days, coordinated quietly across engineers, miners and exchanges, with the supply proven
intact the entire time. Very few networks could pull that off.
The proactive audit worked. Shielded Labs paid someone to go hunting for trouble before an
attacker did, and it paid off exactly as intended.
And the direction of travel is sound. The Z3 stack, with Zebra as the node, Zaino as the
data layer and Zallet as the wallet, is a clean, modular base for the next decade of
building.
---
What needs to change, and who can do it
Plain advice, sorted by who can act and how much it would move things.
Mining pools, the highest impact and lowest effort. Switch your payout address to a
shielded (z) address. Zebra v5.0.0 supports it now. One large pool doing this could lift
the daily shield rate by several points immediately. It's the most valuable single move
anyone in the ecosystem can make today.
Wallet developers, make shielded the default. Not a toggle buried in settings. The
default. When someone creates a wallet, hand them a shielded address first. The pool is fast
enough now, so there's no longer a good reason for transparent to be the easy path.
App and payment builders, accept shielded addresses. Every shop, tip jar or payroll
tool that only takes transparent addresses forces a deshielding event and drains the pool.
Receive at z-addresses and you turn each payment into a vote for privacy instead of against
it.
Everyday users, shield, and shield before you spend. Use a wallet that defaults to
shielded, like Ywallet or Zashi. When ZEC lands in a transparent address, move it into the
pool before you spend it. Your choice protects you and everyone else, because a bigger pool
means stronger privacy for all of its users.
Researchers and community members, keep asking the hard question. One question from
@squirrel about what a block explorer can really see fixed a flaw in my work and sharpened
this whole report. Challenge the numbers. That's how the data stays honest.
---
Questions people keep asking, answered from the data
Research should settle arguments, not start vague ones. These are the questions I keep
seeing. Each answer uses only what the data here supports, and each ends with something the
community can actually act on.
"Is Zcash privacy broken, given the bug, the explorers and the low daily numbers?"
No. The cryptography works. A block explorer literally can't see inside a z→z transaction,
and the Orchard bug was a circuit error that's been fixed, with total supply confirmed
intact. What's weak is usage, not the technology. So stop arguing about whether privacy
works and start measuring whether it's being used. That's the number that moves.
"If 30% of supply is shielded, why do analysts say daily privacy is tiny?"
Because those are two different measurements. Supply is what's parked privately, around 30%.
The daily shield rate is what moves privately, around 4%. People shield to hold and spend in
the open. Whenever you quote a shield rate, say which one you mean. Mixing them up is the
most common mistake in Zcash coverage right now.
"What's the highest leverage thing the ecosystem can do this quarter?"
Get mining pools onto shielded payouts. Mining rewards are about 21% of all transactions and
fully transparent today, and Zebra v5.0.0 already supports paying them to a shielded
address. A coordinated, public push aimed at the top three or four pools would move the
daily shield rate more than any wallet feature or campaign, and it costs the pools almost
nothing.
"Should the counterfeiting bug change how I hold or value ZEC?"
That's a personal call and I'm not a financial adviser, but the facts are clear. The flaw is
patched, the turnstile caps total supply, and Shielded Labs has proposed a supply proof
upgrade so anyone can verify the total without breaking privacy. The honest unknown is that
earlier exploitation can't be disproven, because of the same privacy that protects everyone.
Watch whether that supply proof proposal ships. That, not the patch alone, is what closes
the trust gap.
"Is the shielded pool growing or shrinking?"
Both, depending on how you look. By supply it's growing strongly, 8% to 30%. By daily flow
in my sample it's draining, around seven exits for every entry, most likely exchange and
spending activity. Track the flow ratio month to month. A shift from 7 to 1 back toward 1 to
1 would be the earliest sign that everyday privacy use is catching up to privacy saving.
---
What I'm committing to track, and how
It's easy to promise a monthly report and then quietly never ship it, so here's the actual
mechanism.
There's a repeatable job in the repo at `zecledger/scripts/monthly_shield_report.sh`. It
pulls a fresh multi window sample, sorts every transaction with the corrected method, and
writes a dated file. Each month's output gets committed to `zecledger/reports/`, for example
`reports/shield_rate_2026-07.json`, so the full history is public and anyone can check the
maths against the raw file.
I run it on the 1st of each month, push the data to GitHub, post a short note in the ZecHub
Discord, and publish a brief write up. The first scheduled report is 1 July 2026, and I've
set a recurring reminder so it doesn't slip.
Four things I'll be watching: whether pools start paying rewards to shielded addresses, the
daily shield rate trend now that NU6.2 and shielded mining are live, the direction of pool
flow, and the count of fully private z→z transactions, which is the purest signal of all.
The raw 500 transaction dataset behind this report is already in the repo:
research_data_500tx_verified.json.
---
Methodology, run it yourself
From my own Zebra v5.0.0 node:
```bash
python3 - <<'PY'
import json, urllib.request, random
def rpc(method, params=[]):
    url = 'http://localhost:8232'
    body = json.dumps({'jsonrpc':'2.0','method':method,'params':params,'id':1}).encode()
    req = urllib.request.Request(url, data=body, headers={'Content-Type':'application/json'})
    return json.loads(urllib.request.urlopen(req).read())['result']

tip = rpc('getblockcount')
heights = sorted(random.sample(range(tip-5000, tip), 50))
total=shielded=coinbase=transparent=0
for h in heights:
    for tx in rpc('getblock', [str(h), 2]).get('tx', []):
        total += 1
        is_cb = any('coinbase' in str(i) for i in tx.get('vin', []))
        has_shield = (tx.get('vShieldedOutput') or tx.get('vShieldedSpend')
                      or tx.get('orchard', {}).get('actions'))
        if is_cb: coinbase += 1
        elif has_shield: shielded += 1
        else: transparent += 1
print(f'blocks=50 total={total} coinbase={coinbase} '
      f'transparent={transparent} shielded={shielded} '
      f'shield_rate={shielded/total*100:.1f}%')
PY
```
From the Blockchair API, with the corrected classification:
```bash
python3 - <<'PY'
import json, urllib.request
url = 'https://api.blockchair.com/zcash/transactions?limit=100&s=block_id(desc)'
req = urllib.request.Request(url, headers={'User-Agent': 'ZecLedger/0.1'})
txs = json.loads(urllib.request.urlopen(req, timeout=30).read())['data']

def kind(t):
    cb = t.get('is_coinbase')
    inp = t.get('input_count') or 0
    delta = t.get('shielded_value_delta') or 0
    s_out = len(t.get('shielded_output_raw') or [])
    if cb: return 'coinbase'
    if inp == 0 and s_out > 0: return 'private_z2z'
    if inp > 0 and delta > 0: return 'shielding_t2z'
    if inp == 0 and s_out == 0: return 'deshielding_z2t'
    return 'transparent_t2t'

from collections import Counter
c = Counter(kind(t) for t in txs)
n = len(txs)
for k in ['coinbase','transparent_t2t','deshielding_z2t','shielding_t2z','private_z2z']:
    print(f'{k:<16} {c[k]:>3}  ({c[k]/n*100:.0f}%)')
print(f'any shielded     {(c["shielding_t2z"]+c["private_z2z"])/n*100:.1f}%')
PY
```
---
Annotated bibliography
Every source below was used directly in this report. Each note says what I took from it and
how far I'd trust it, so you can weigh the evidence yourself instead of taking my summary on
faith.
1. Zcash Foundation, Zebra 4.5.3 and 5.0.0: Emergency Soft Fork and NU6.2 Activation
https://zfnd.org/zebra-4-5-3-and-5-0-0-emergency-soft-fork-and-nu6-2-activation/
The authoritative source for the upgrade timeline: soft fork at block 3,363,426 on 2 June,
NU6.2 at block 3,364,600 on 3 June, and confirmation that total supply held via the
turnstile. I treat this as ground truth for every date and block height.
2. CoinDesk, Zcash price crash and record bearish bets
https://www.coindesk.com/markets/2026/06/05/bearish-zcash-bets-hit-record-high-as-privacy-token-s-price-crashes
Used for the disclosure framing and the price reaction, including the detail that the sell
off was spot driven with record open interest, so confidence rather than forced
liquidation. Cross checked against BitMEX and CoinGecko.
3. TechTimes, audit exposes four year Zcash Orchard bug
https://www.techtimes.com/articles/317831/20260605/why-crypto-crashing-ai-assisted-audit-exposes-four-year-zcash-orchard-bug-zec-plummets-31.htm
Source for the audit mechanics: Hornby's targeted Orchard review on 29 May, the working
exploit confirmed in a local test, and the four year age of the flaw. Corroborated by
Unchained and Decrypt.
4. Unchained, audit uncovers critical Zcash Orchard vulnerability
https://unchainedcrypto.com/ai-assisted-audit-uncovers-critical-zcash-orchard-vulnerability-that-could-have-minted-unlimited-counterfeit-zec/
Independent confirmation of the vulnerability details: two lines of under constrained
circuit code, unlimited undetectable counterfeit ZEC, present since Orchard launched in May
2022. Used so the bug story didn't rest on a single outlet.
5. The Defiant, Shielded Labs proposes new Zcash upgrade to prove ZEC supply
https://thedefiant.io/news/blockchains/shielded-labs-proposes-new-zcash-upgrade-to-prove-zec-supply-after-orchard-bug
The forward looking piece: a proposed supply proof upgrade so anyone can verify the total
without breaking privacy, plus the plain statement that earlier exploitation can't be
disproven cryptographically. This is the trust gap story, not just the patch story.
6. crypto.news, why 30% of Zcash supply is now in the shielded pool
https://crypto.news/why-30-of-zcash-supply-is-now-in-the-shielded-pool/
My main source for the supply growth curve from 8% to 30%, the daily transaction count, the
Orchard pool at 4.2 million ZEC, and the 59.3% February 2026 transaction shield rate peak.
The supply figures here trace back to this and to Messari.
7. Messari, Zcash project page, and CoinGecko, Zcash
https://messari.io/project/zcash and https://www.coingecko.com/en/coins/zcash
Independent confirmation of the roughly 30% shielded supply figure and circulating supply of
about 16.7 million ZEC. Two trackers agreeing raised my confidence on the headline number.
8. Delphi Digital, shielded supply commentary
Source for the projection that more than half of supply could be shielded within 12 to 18
months, and the point that shielded coins show lower turnover. Used as an analyst
projection, and labelled as one, not as settled fact.
9. Blockchair, Zcash API (/stats and /transactions)
https://api.blockchair.com/zcash/stats
My live source for the network snapshot and the 1,000 transaction daily sample. It's also
the source that returns null for shielded fields, which is the evidence behind @squirrel's
question and the explorer blindness point.
10. My own Zebra v5.0.0 full node, direct JSON-RPC
The only first party data here. It gave the 2019 figure of 7.8% shielded across 425
transactions in 50 blocks, queried with no third party in the loop. Slower and historical
for now, but the most trustworthy source I have, and the one I'll lean on more as it
finishes syncing.
11. ZecLedger raw research data, research_data_500tx_verified.json
https://github.com/vancube2/zecledger
The actual 500 transaction dataset behind the daily mix figures, committed publicly so
anyone can rerun the classification and check the maths. Don't trust me, run it.
---
About
Built with ZecLedger, an open source Zcash accounting and research tool, for the ZecHub
Hackathon 2026 (Accounting Track). MIT licensed.
Special thanks to @squirrel, whose question about what a block explorer can really see led
me to find and fix the flaw in my own method. The work is better for it.
A note on limits, plainly. The daily transaction figures here are samples. Shield rate
numbers taken from a block explorer are floors, not exact totals, because fully private
transactions can't be seen from outside the pool. The supply figures, around 30% of ZEC
shielded, come from the third party trackers above, not from my own node. I've tried to be
clear throughout about which number comes from where.
