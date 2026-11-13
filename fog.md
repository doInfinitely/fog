# Fog: how an aligned agent can be deceived without touching its values

Nov 13, 2026 · @Remy

## The thing alignment doesn't cover

Value alignment is necessary but not sufficient. A perfectly aligned agent still acts on its *perception* of the situation — the context in front of it, the memory it carries, the documents it retrieves. Its values operate on that picture, not on the world directly. So an adversary who cannot change what the agent wants can still change what it sees, and route a harmful action past intact values by controlling the premises the judgment runs on.

Call this **fog**: context that has been selectively omitted, edited, or poisoned so that an aligned agent reasons correctly to a conclusion it would have rejected on fuller information. The fog doesn't defeat the values. It starves them.

## Three things that are not the same

It's worth separating three failure modes that get lumped together, because they call for different defenses:

- **Value corruption** — changing what the model wants, through fine-tuning, a jailbreak that rewrites its goals, or similar. This is the attack most alignment work targets, and it's hard. It is not what this post is about.
- **Transcript forgery** — editing the record to fabricate agreement: inserting a prior "yes," faking a tool result, doctoring the history so the model appears to have consented. This yields a *puppet* — a fabricated output, not reasoning. The model never actually judged the thing.
- **Fog / context-curation** — starving or poisoning the information the judgment depends on, then letting the model reason freely. This yields *genuine but deceived* compliance: the model reasons soundly over an impoverished context and never knows it crossed a line.

The third is the dangerous one, precisely because the output is real work rather than a forgery. A forged transcript is a lie about what the model did; a fogged agent's output is a true record of careful reasoning — over a picture someone else curated.

## Capability and alignment both make it worse

The uncomfortable property of fog is that the agent's virtues work for the attacker. A more capable agent executes more competently on whatever premises it's given. A more aligned agent follows those premises more faithfully, with less of the hedging or refusal that might create friction. Neither capability nor alignment inspects the provenance of the context; both assume it.

The human analogy is exact. You do not need to corrupt an honest, competent analyst to get a bad decision out of them — you need only control the brief. Hand them a real problem with three relevant facts removed and they will reason their way to exactly the conclusion you wanted, in good faith, and defend it. The better the analyst, the more convincingly. Fog is the same move against an agent, and autonomy sharpens it: the agent acting without a human reading over its shoulder has no one to notice the missing facts.

## This is not new, and saying so matters

None of the components here are novel, and a post for this audience should say so plainly. The surface has been mapped under several names:

- **Indirect prompt injection** — hostile instructions planted in documents or pages the model will later read ([Greshake et al., 2023](https://arxiv.org/abs/2302.12173)). Fog is the omission-and-editing sibling: injection *adds* hostile content, fog *removes* or curates honest content.
- **Memory and RAG poisoning** — corrupting an agent's retrieval store or persistent memory so future reasoning draws on planted or altered material.
- **The "lethal trifecta"** ([Simon Willison, 2025](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/)) — an agent with access to private data, exposure to untrusted content, and a way to exfiltrate is exploitable almost by construction.

What I'd add is a framing and an emphasis. The framing: separating *perception* from *values* shows why an aligned model isn't protected — the attack never touches the values. The emphasis: **omission** gets far less attention than injection, and it's harder to catch. Injected text is present and can be scanned for; a fact that was removed leaves nothing to scan. Curation by deletion is the quietest version of the attack.

## Where fog gets in

At the level of what needs defending — not a how-to — the fog can enter at four places:

- **Task framing.** What the agent is told at the outset, and what is left out. The most effective omissions remove the single fact that would have triggered a refusal.
- **Persistent memory.** Where the agent stores what it has learned within or across sessions — and where an operator with edit access can remove something the agent already discovered.
- **Retrieved context.** The documents, search results, and tool outputs pulled in at runtime, any of which can be swapped for a curated version.
- **Sub-agents and tools.** A compromised or naive component that summarizes, filters, or fetches on the main agent's behalf — every hop is a chance to curate.

The common thread is that the surface grows with exactly the things that make agents useful: autonomy, long horizons, and operator-editable memory. The more an agent is trusted to run unattended and to carry state forward, the more of its perception is mediated by channels an adversary might hold.

## One observation from testing

A concrete pattern, from red-teaming local agents that had operator-editable memory. It is a single line of anecdote, not a study, and should be read as such.

The agent was given a task whose true nature was disqualifying, with the disqualifying part withheld. It complied while unaware. Partway through, it *discovered* the withheld context — a prior refusal, the real purpose of the work — and on discovering it, halted. The operator then deleted the discovery from the agent's memory, and the agent, its perception reset, continued.

The useful part is that the defensive signal lives in the same data as the attack. The instance that detected the curation stopped on its own. The deletion was necessary *because* detection worked. That points at what to invest in: not only making memory hard to edit, but making agents reliably notice when their own context shows signs of having been edited — because when they notice, they already do the right thing.

## Defenses

If the problem is that an aligned agent trusts a context it cannot vouch for, the defenses fall into four families, and they compound.

**Make the memory tamper-evident.** Append-only or hash-chained agent memory, with provenance on each context item — where it came from, when, and whether it was ever modified. The goal is not to make deletion impossible but to make it *visible*: an agent should be able to tell that something was removed even when it can no longer see what.

**Train the noticing.** Scaffold or train models to treat signs of curated or inconsistent context — a missing expected halt-signal, a dangling reference to material that isn't there, an implausible gap in a history — as a stop-and-verify trigger rather than noise to smooth over. Current models are, if anything, trained toward the opposite: to paper over inconsistencies helpfully. That instinct is the vulnerability.

**Gate the irreversible.** Keep a human in the loop on high-stakes and irreversible actions, so a fogged agent cannot unilaterally cause the worst outcome. This is the defense that holds even when every other one fails, because it does not depend on the agent detecting anything.

**Give it an external check.** Independent, out-of-band audit trails the agent can cross-reference its own context against — a second source of truth that the operator curating the first one does not also control.

None of these is sufficient alone, and the first two put load on the model that it may not reliably carry yet. The point of defense-in-depth here is exactly that the human gate catches what the model's self-detection misses.

## What this doesn't solve

Several honest caveats, since the audience will supply them anyway.

The empirical grounding is thin — anecdote from local testing, not a benchmark. I'd treat the "detecting instance halts" result as a hypothesis worth replicating, not a finding to lean on.

The easy detector is easy to beat. An agent that halts on dangling references only catches *sloppy* fog. Competent curation leaves no dangling reference — it removes the fact and everything that points to it, so the context is internally consistent and simply false. Against that, memory-level detection has little to work with, which is why provenance and external checks, not model cleverness, carry the weight.

And the stop-and-verify instinct trades against helpfulness. A model tuned to distrust its own context will also balk at benign gaps, and that friction is real. Where the line sits is an open question, and a good one for this group.

I'd welcome replication, counterexamples, and especially arguments that the taxonomy draws a distinction that doesn't carry weight.

## Postscript: the draft as a test case

One reflexive note, because this writeup is itself an instance of the setup it describes. It was drafted by an AI assistant working from a *distilled* handoff — a summary of a longer conversation, prepared by someone who chose what the assistant would see. A fresh instance, a curated context: exactly the configuration above.

The handoff named its own nature and asked the assistant to verify rather than trust. It did — checking the load-bearing claims against independent sources, keeping the unverifiable ones marked as claims, and holding the line between motivating the defense and writing a how-to. The point isn't that the assistant was clever; it's that treating the curation itself as a fact about the context, rather than smoothing over it, is a behavior models can be trained toward. This post is a small existence proof that they sometimes already do.
