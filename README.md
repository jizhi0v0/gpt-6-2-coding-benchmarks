# GPT-6.2 coding benchmarks

On a task that spanned a service and its browser editor, **Astra low and Astra medium each passed all six acceptance checks in three out of three independent runs**. Low averaged $2.92 per run and medium averaged $3.52. One run with Astra low supervising Sol high cost $2.99. The median solo low run cost $3.01, so these results support using low on this task but give little evidence of consistent savings over the supervised run.

The benchmark covers 31 implementation runs and four read-only code reviews in one task family. Each configuration was run at most three times. Costs are API-equivalent USD estimates based on recorded input, cached input and output tokens. Implementation costs include self-testing by the agents listed for each group. External coordination and independent acceptance testing were metered separately. Subscription quota consumption was not measured.

## Independent and supervised implementation

| Configuration | Runs | Cost per run (USD) | Mean elapsed time | Independent acceptance results |
|---|---:|---|---:|---|
| Astra low, solo | 3 | 3.01, 3.30, 2.44 | 11.1 min | All six checks passed in all three runs |
| Astra medium, solo | 3 | 3.82, 3.78, 2.96 | 12.7 min | All six checks passed in all three runs |
| Astra low supervising Sol high | 1 | 2.99 | About 18 min | Cold-start playback timed out on the final build. All three playback retests and the other core checks passed. |

All six solo runs started from the same code and prompt. Two requirements were explicit from the start. A preview had to start playing while the provider was still generating audio, and a user had to be able to reopen the editor, find an earlier job and use its result. Each agent planned, implemented and tested its own changes without external feedback before handoff. The supervised run received independent feedback during implementation and had a shorter cold-start playback window. Both differences matter when comparing elapsed time and pass rates.

All six solo implementations reused the existing SDK for streaming and full audio generation. None changed the SDK or reimplemented the provider's server-sent event parser. Two low runs used MediaSource in the browser. The other four used a native audio element with a short-lived playback credential. Both approaches passed the early-playback check. The six implementations cost $19.31 in total, with about $6.16 recorded separately for coordination and independent testing through the reporting cutoff.

## Sol results by task input and workflow

| Input and workflow | Runs | Early browser playback | Save and read back in both locations | Resume original job after restart | Mean cost per run (USD) |
|---|---:|---:|---:|---:|---|
| Sol implements a plan prepared by Astra low | 9 | 9/9 | 9/9 | 9/9 | 1.15, implementation only |
| Sol receives the original brief with two Luna max agents exploring the code | 9 | 0/9 | 9/9 | 9/9 | 1.53, including exploration |
| Astra receives the original brief, delegates to Sol and reviews the work | 4 | 0/4 | 4/4 | 4/4 | 5.14, lead and worker |
| Staged implementation and reviews from the original brief | 2 | 0/2 | 2/2 | 2/2 | 7.89, all agents in the workflow |

The first two groups each ran Sol medium, high and xhigh three times per effort level. The four delegated runs paired each of Astra low and medium with each of Sol medium and high once.

All 24 runs passed the save and restart checks. The clearest difference appeared in the audio stream. With a prepared plan, Sol passed both early audio delivery and browser playback in 9/9 runs, across all three effort levels. With the original brief and two explorers, it passed both checks in 0/9, although editing during generation worked in 9/9. In that group, the client failed to receive the required audio bytes while the provider was still generating. A slow browser start alone cannot explain the misses. The prepared-plan group has no comparable observations for editing while pending or recovering a result after reopening, so its 9/9 applies only to the recorded checks.

The original brief asked for a fast preview. Astra's six solo runs explicitly required playback before generation finished. The 0/9 and 6/6 figures describe how those two workflows behaved. They leave open which model would infer the streaming requirement from the same brief. The browser fixtures, observation windows and cost coverage also varied across groups.

## Independent code review

Astra low and medium each reviewed the same two implementations. For sample A, each identified one of four previously fixed issues. For sample B, each identified one of three. Both efforts therefore scored 2/7 exact matches. The reference set covers known fixes only, and additional findings were not consistently adjudicated. These data support neither a false-positive estimate nor an overall ranking of review quality.

## Comparison with the release benchmarks

OpenAI reports that Sol scored within 0.5 points of Astra on FrontierCode 2.0 at about one-fifth the cost, with both models at max reasoning effort. FrontierCode evaluates whether changes to real codebases meet maintainers' merge standards. Our runs cover one task family at different effort levels. A local comparison of the score gap or the cost ratio would require matched model runs, acceptance criteria and cost coverage. The $1.15 mean for Sol with a prepared plan excludes planning; the $2.92 solo Astra mean includes it. [OpenAI release announcement](https://openai.com/index/introducing-gpt-6-2-astra-sol/)

SDK reuse answers a narrower engineering question about whether an agent rebuilds an existing provider protocol. We did not independently grade overall code simplicity or test focus. The four read-only reviews cover Astra only, and the implementation data contain no matched runs of the preceding model generation. OpenAI's examples of cross-service debugging with Astra and PR review with Sol describe suggested uses. [ChatGPT Learn](https://learn.chatgpt.com/docs/whats-new/gpt-62-astra-sol)

## Baseline for future models

The **v1 baseline** is the six independent runs. The task combined streaming previews and asynchronous generation in a single service. The following checks define what counted as a pass for those runs.

| Check | Pass condition |
|---|---|
| Early audio bytes | The simulated provider holds back the second half of the audio. The client receives valid audio bytes before release. |
| Early browser playback | After one click on the editor's preview button, the playback position exceeds 0.08 seconds within seven seconds while the provider is still generating audio. |
| Editing while a job is pending | After the provider starts the submitted job, the tester can change another editor field and read back its new value before the job finishes. |
| Find and use a result after reopening | Reopen the editor, retrieve the signed-in user's history from the server, select the matching job and verify that its result is applied. |
| Save in both locations | Use the same completed result in two supported locations. After saving, read both files back and verify that each matches the provider output byte for byte. |
| Resume after restart | Force-stop the service process during an active job. After restart, the job completes under its original ID. |

For a future comparison, keep the starting code, prompt, fixtures, observation windows and feedback policy fixed. Record at least three runs per effort level and report the first acceptance result separately from retests. Assign a new cohort when the project or acceptance criteria change, or when agents receive feedback before handoff. The solo runs used a real local browser, service process and database with a simulated upstream provider. The CSV files are sufficient to recompute the published aggregates. An exact rerun requires the original project and fixtures.

The [implementation results](implementation.csv) list roles, cost coverage and check outcomes by run. A blank cell means no comparable observation was recorded. A failed check means the run did not meet that check's condition during its observation window. Receipt of audio bytes and actual browser playback are scored separately. The supervised run records the final build's cold start separately from its playback retests. The [review results](review.csv) list reference counts, exact matches, elapsed time and cost for each anonymized sample.

## Open comparisons

| Question | Matched comparison |
|---|---|
| Can Sol deliver the same behavior when the requirements are explicit? | Run Sol high solo three times under the six-run Astra protocol, including the seven-second playback window and no external feedback before handoff. The existing Astra runs provide the baseline. |
| Which constraints does each model infer from the original brief? | Give Astra low and Sol high the same original brief in fresh solo runs. Before implementation, have each record its own acceptance criteria. Check whether it identified early playback, chose a design that supports it, and exercised it in the browser. This extra planning instruction defines a separate cohort for both models. |
| Does higher effort or a newer generation change the result? | Repeat a matched cohort with Sol xhigh to test effort, or with GPT-6.1 Sol at the same effort to test generation. Vary one factor at a time. |

To measure rework, record first-handoff acceptance, repair rounds, human review minutes and total API-equivalent cost through the first passing delivery, including planning, review and repairs. Keep unsuccessful runs and their costs in the record. Repetitions on the same task measure repeatability; additional task families are needed to assess how well the result generalizes.
