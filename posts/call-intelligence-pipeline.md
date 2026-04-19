# Building a Call Intelligence Pipeline: Why Transcription Is the Easy Part

## Thesis

Every consumer company with a phone channel sits on thousands of hours of unanalyzed calls. The tooling to transcribe them is now nearly free. The actual product problem is the analysis taxonomy — what you extract, how you classify it, and how it routes to the people who should act on it. Most teams nail transcription and then drown in unstructured text.

## The frame

If your company makes outbound or inbound calls — consultations, inside sales, customer support, scheduling — those calls contain signal that never makes it into your CRM, your analytics, or your product decisions. The standard approach: record everything, transcribe with Whisper or a cloud ASR service, dump it into a folder, and call it "call intelligence."

That's a transcription pipeline, not an intelligence pipeline. Intelligence means structured extraction — who said what, what was decided, what product was discussed, what the customer's sentiment was, and where that data goes next.

## Why it's harder than it looks

**1. Call types need different analysis prompts.**
A consultation call and a sales call have completely different structures. A consultation meanders — the caller explores, the agent advises, there's no single "conversion moment." A sales call has a pitch, objections, and a close. If you run the same extraction prompt on both, you get mush. The first design decision: classify the call type BEFORE analysis, then route to a type-specific prompt.

**2. Speaker separation is a product decision, not just an audio problem.**
Stereo recording (agent on left channel, customer on right) solves diarization trivially — but only if your telephony provider supports it. If you're on mono, you need speaker diarization models, and they're still unreliable on overlapping speech. The PM decision: is it worth renegotiating your telephony contract for stereo, or do you accept noisy diarization? In my experience, stereo is worth the fight — it halves your analysis error rate.

**3. Long calls break naive chunking.**
A 45-minute consultation doesn't fit in a single LLM context window at raw transcript length. You need a chunking strategy. Naive time-based chunks (every 5 minutes) split mid-thought. Better: chunk on silence gaps or topic shifts. Best: a two-pass approach — first pass summarizes chunks, second pass analyzes the summaries. The trap is building the chunking layer after you've already committed to a prompt design that assumes full-call context.

**4. The "transcribe everything, analyze nothing" trap.**
It's psychologically satisfying to watch transcription volume climb. Thousands of hours processed! But if nobody reads the analysis, or if the analysis isn't structured enough to route to action, you've built an expensive archive. The discipline: define the output schema (decisions, action items, topics, sentiment, next steps) before you build the pipeline. If you can't name the consumer of each output field, don't extract it.

## A playbook for PMs building this

1. **Start with the output, not the input.** Write the analysis output schema first. Who reads it? What do they do with it? A sales manager wants conversion signals and objection patterns. An ops lead wants scheduling failures and SLA misses. A product manager wants feature requests and pain points. These are different schemas.

2. **Classify before you analyze.** Build a lightweight call-type classifier (even rule-based: department tag, queue ID, or a 10-second audio snippet prompt). Route each call to its type-specific analysis prompt.

3. **Invest in stereo recording early.** If your telephony supports it, record in stereo. The downstream quality improvement is massive — speaker attribution, sentiment per speaker, interruption patterns.

4. **Design the chunking strategy for your longest calls.** If 80% of calls are under 10 minutes, you might not need chunking. If you have 30–60 minute consultations, build chunking into the architecture from day one. Two-pass (chunk-summarize-analyze) is more robust than trying to fit everything in one pass.

5. **Build the routing layer, not just the extraction layer.** Extraction without routing is an archive. Each extracted field should have a destination: CRM event, dashboard metric, Slack alert, or agent coaching queue. If a field has no destination, don't extract it yet.

6. **Measure analysis quality, not just transcription accuracy.** Transcription WER (word error rate) is well-understood. Analysis quality is harder — sample 50 calls, have a human label the expected outputs, compare. Do this monthly. Prompt drift is real.

## How to know you got it right

- **Routing coverage:** >80% of extracted insights have a defined consumer (dashboard, CRM, alert).
- **Analysis adoption:** the people who should read the analysis actually open it weekly.
- **Time-to-insight:** a call recorded today produces actionable output within the same business day.
- **Prompt accuracy on sampled calls:** >85% agreement between LLM extraction and human labels on a monthly sample of 50 calls.
- **Call-type classification accuracy:** >90% on a held-out set. If it's lower, your type-specific prompts are analyzing the wrong call types.

## Closing

The transcription problem is solved. The product problem — what to extract, how to classify it, where to route it — is where PMs earn their keep. If you're building a call intelligence system, start from the action, not the audio.
