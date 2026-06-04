# Apple's On-Device Model Replaced a Cloud API in My Desktop App

*There is a capable language model already sitting inside macOS, free, offline, and private. For classification and structured-output work, it does the job you would normally rent a remote API for. Here is how I used it to read bank statements without any of them leaving the laptop, with the code.*

![An isometric scene: a closed Mac-shaped device on a desk sorting a stream of small coral transaction cards into labeled teal bins entirely inside its own glass enclosure, a cut network cable lying beside it to show nothing leaves the machine](hero.png)

I build a desktop app called Subscription Radar. It reads your bank and credit-card exports, finds the recurring charges, and reconciles them into one list of every subscription you are paying for. The input is about as sensitive as personal data gets: a year of someone's bank statement, every merchant, every amount, every date.

So when the app needs a language model to make a judgement call, "is this messy run of charges actually a subscription, and at what cadence," shipping that statement to a cloud API was never going to happen. The promise of the app is that nothing leaves your machine. The classification had to run locally or not at all.

It runs locally. The work is done by the roughly three-billion-parameter model that ships inside macOS, the same one behind Apple Intelligence, exposed to developers as the Foundation Models framework. It is free, it runs on the device with no network call, and for the kind of job I needed, deciding what a thing is and returning a structured answer, it is good enough to ship. This piece is how that works, where the model earns its place, where it does not, and the actual Swift to drive it.

**Three things to take away:**

- For classification, tagging, extraction, and structured output, the on-device model does work you would normally pay a remote API for, at zero marginal cost and with the data never leaving the machine. That covers a large slice of the "ask an LLM" tasks in a real app.
- It is not a frontier chatbot. The context window is 4096 tokens for input and output combined, and the model is weak on broad knowledge and hard reasoning. Knowing which side of that line your task sits on is the entire skill.
- Guided generation is the feature that makes it usable. You define a Swift struct, annotate it, and the model returns a guaranteed instance of that type. No prompt-and-pray, no parsing free text, no schema drift.

## What the model is, and is not

The Foundation Models framework landed with macOS 26, iOS 26, and iPadOS 26. One line, `import FoundationModels`, gives you direct access to the on-device model from Swift. No download, no key, no account, no per-token bill.

What it is built for is language understanding, structured output, and tool calling. It classifies, tags, extracts, rewrites, and summarizes the text you hand it. What it is not built for is being a general-knowledge assistant. It does not know much about the world, it is not a strong reasoner, and it will happily be confident and wrong if you ask it a trivia question. Apple is direct about this: it is an engine for building features, not a replacement for a frontier model.

Two limits shape everything you do with it. The combined input-plus-output context window is 4096 tokens, which is small, so you render your data compactly and you do not paste whole documents. And it requires recent Apple Silicon with Apple Intelligence enabled, so you write a real availability check and a fallback for every other machine. Treat both as design constraints from the first line, not as surprises later.

## The job: classify a charge history, as a typed struct

The task in Subscription Radar is narrow and well-shaped, which is exactly what this model is good at. Given a merchant name and a list of dated charges, decide whether it is a recurring subscription, at what cadence, in what category, and how confident the model is.

The feature that makes this clean is guided generation. Instead of asking for text and parsing it, you describe the answer as a Swift type and the model is constrained to produce a valid instance of it. Here is the actual output type from the app:

```swift
@Generable(description: "A judgement about whether a series of charges is a recurring subscription.")
struct SubscriptionJudgement: Codable, Sendable {

    @Guide(description: "True if these charges represent a recurring paid subscription.")
    var isSubscription: Bool

    @Guide(description: "How often the charge recurs.",
           .anyOf(["weekly", "monthly", "quarterly", "annual", "unknown"]))
    var cadence: String

    @Guide(description: "A short, lowercase category, e.g. 'streaming', 'software', 'news'.")
    var category: String

    @Guide(description: "Confidence from 0.0 to 1.0.", .range(0.0 ... 1.0))
    var confidence: Double

    @Guide(description: "One or two sentences explaining the judgement.")
    var reasoning: String
}
```

The `@Generable` macro makes the struct something the model can emit. Each `@Guide` describes a field, and the guides do real work. The `.anyOf` on `cadence` pins the string to a closed vocabulary, so downstream code never has to handle "Monthly" or "every month" or "mo." The `.range` on `confidence` keeps it a real probability. You get a typed value back, not a blob of JSON you hope parses.

## The call

Driving it is three steps: check the model is available, build a session, ask for your type. The availability gate is not optional, because most machines in the world cannot run this, and you want a clean signal when that is the case.

```swift
import FoundationModels

let model = SystemLanguageModel.default

switch model.availability {
case .available:
    break
case .unavailable(let reason):
    // .deviceNotEligible, .appleIntelligenceNotEnabled, or .modelNotReady
    // Fall back to your non-LLM path and tell the user why.
    return
}
```

Then a session with system instructions, and the structured request. Temperature zero plus greedy sampling makes the output deterministic, which matters when you are classifying and want the same input to give the same label every run:

```swift
let session = LanguageModelSession(
    model: model,
    instructions: Instructions("""
        You are a precise financial classifier. Decide whether a charge history
        is a recurring subscription and fill in every field. Be conservative:
        if the evidence is weak, use a low confidence and "unknown" cadence.
        """)
)

let options = GenerationOptions(sampling: .greedy, temperature: 0)

let response = try await session.respond(
    to: Prompt(prompt),
    generating: SubscriptionJudgement.self,
    includeSchemaInPrompt: true,
    options: options
)

let judgement = response.content   // a real SubscriptionJudgement
```

The prompt itself is plain text. The trick, given the tiny context window, is to render the data compactly rather than dumping rows:

```text
Analyze the following merchant charge history and decide whether it is a
recurring subscription.

Merchant name: SPOTIFY P3A4F
Charges (date: amount):
  - 2025-01-03: 10.99
  - 2025-02-03: 10.99
  - 2025-03-03: 11.99

Consider the spacing between dates to infer cadence (about 30 days = monthly).
Consider whether amounts are steady, trend over time, or look like overlapping
plans. If there is too little signal, use "unknown" and a low confidence.
```

And the model returns, on device, in well under a second after the first warm-up:

```json
{
  "isSubscription": true,
  "cadence": "monthly",
  "category": "streaming",
  "confidence": 0.95,
  "reasoning": "Three charges about 30 days apart at a steady ~11 EUR, a small
                price rise in March. Consistent with a monthly streaming plan."
}
```

No network. No key. No cost. The bank statement that produced those charges is still only on the laptop.

## The non-obvious use: let the model read any bank's CSV

The second job I gave the model is the one I did not expect to need, and it is the better story. Every bank exports CSVs differently. Some put debits as negative numbers, some use a separate "Bij/Af" direction column in Dutch, some split money-out and money-in into two columns. Writing a parser per bank does not scale.

So I hand the model the headers and two sample rows and ask it to map the columns to roles, again as a typed struct:

```swift
@Generable(description: "A mapping from a bank CSV's columns to transaction roles.")
struct ColumnMapping: Codable, Sendable {
    @Guide(description: "Exact header name of the DATE column.")
    var dateColumn: String

    @Guide(description: "Exact header of the AMOUNT column.")
    var amountColumn: String

    @Guide(description: "How debit vs credit direction is encoded.",
           .anyOf(["amount-sign", "sign-column", "separate-columns", "all-outflow"]))
    var signMode: String

    @Guide(description: "Exact header naming the MERCHANT. Empty string if none.")
    var merchantColumn: String

    @Guide(description: "Confidence from 0.0 to 1.0.", .range(0.0 ... 1.0))
    var confidence: Double
}
```

This is schema inference, on device, for free, against a format the model has never seen. The deterministic parser still does the actual row extraction, the model only decides which column means what, but that one judgement is what used to require either a hand-written adapter or a cloud call with the user's financial headers in the payload. Now it is a local function that returns a Swift value.

That pattern generalizes well past banking. Any time you have messy, varied, real-world structure and you need to map it to your clean internal shape, the local model is a good first pass, and the structured output keeps it honest.

## How a JavaScript app reaches a Swift-only framework

Subscription Radar is an Electron and TypeScript app, and Foundation Models is Swift only. The bridge is boring on purpose, which is the point.

I compiled a tiny Swift command-line helper that does nothing but read requests as NDJSON on standard input, run them through the on-device model, and write one JSON line of structured output per request to standard output. The Node side spawns that helper once, keeps it warm, and routes responses back by id. Everything degrades to null: wrong platform, missing binary, a timeout, a per-import budget cap, anything at all, and the caller silently falls back to the deterministic verdict. No error from the model path is ever allowed to break an import.

![An architecture diagram: an Electron and TypeScript app on the left sends NDJSON requests over stdio to a small warm Swift helper process, which calls the on-device 3B Foundation Models, which returns structured JSON, all inside a box labeled on-device with a crossed-out network arrow leaving it](diagram-architecture.png)

That shape, a long-lived helper speaking line-delimited JSON over a pipe, is reusable for any non-Swift app that wants the on-device model: a Node CLI, a Python tool, a Tauri app. The model is Swift's, but a forty-line helper makes it everyone's.

## Pros and cons, from shipping it

The case for reaching for the on-device model first, on a classification-shaped task, is strong.

![A comparison panel: on-device Foundation Models versus a remote API across cost (free vs per-token), privacy (nothing leaves the device vs data sent to a vendor), offline (works vs needs a connection), latency (sub-second local vs round-trip), and capability (good at classification and structure vs broad knowledge and hard reasoning), and context window (4096 tokens vs large)](diagram-comparison.png)

The wins are real. It is free, with no per-token bill, so a feature that classifies every transaction in a year of statements costs nothing to run. The data never leaves the device, which for financial or health or personal data is not a nice-to-have, it is the whole product. It works offline. It is fast after a one-time cold load, well under a second per classification. Temperature zero gives you deterministic labels. And guided generation hands you a typed value instead of a parsing problem.

The limits are just as real, and you design around them.

- **The context window is 4096 tokens, input and output combined.** You cannot paste a document. You summarize and render data compactly, the way the charge history above is three short lines, not raw CSV rows.
- **The model is small and not a knowledge base.** It is weak on world facts, math, and multi-step reasoning. Keep the task to judgement over the text you provide, not questions about the world.
- **It only runs on recent Apple Silicon with Apple Intelligence enabled.** You write the availability switch and a real fallback. On every other machine your feature has to work without it.
- **Instructions work best in English**, even when user content is in another language. My instructions are English, the merchant names are Dutch, and that split is fine.
- **Guided generation produces every field you declare**, used or not, so do not bloat the struct with fields you will not read.
- **One non-obvious trap: do not block the main thread.** Foundation Models delivers results over XPC on the main queue. If you block the main thread waiting on a semaphore, `respond` deadlocks at zero percent CPU forever. Drive it with async/await and keep the main actor free.

## Other things to point it at

Subscriptions are one use. The same local-classification shape fits a long list of features that teams currently send to a cloud model:

- **Content tagging and categorization.** Apple ships a `contentTagging` use-case adapter tuned for exactly this, auto-tagging notes, emails, photos by description, or to-do items, on device.
- **Triage and routing.** Classify an incoming support message, email, or form by intent and urgency locally, then send only the genuinely hard cases to a cloud model. Most of the volume never leaves the machine.
- **Extraction from pasted text.** Pull a structured order, address, or event out of a blob the user pasted, into a typed struct, without a network call.
- **Redaction before the cloud.** Use the local model to find and strip personal data from text before anything goes to a remote service. The sensitive pass is the one that should stay local.
- **Smart replies and rewrites.** Tone adjustment, short reply suggestions, and summarization of on-device notes or messages, where the content is private by default.
- **Local search understanding.** Turn a fuzzy natural-language query into a structured filter over the user's own data, on device.

The throughline is that all of these are judgements over text the user already has, returned as structure. That is the model's home turf.

## When to still call the cloud

The honest boundary: reach for a remote model when you need broad world knowledge, genuinely hard reasoning, a large context, or you are not on an Apple platform. The strongest pattern is not local-or-cloud, it is local-then-cloud. Run the cheap, private, free classification on device for the ninety percent of cases it handles, and escalate only the residue to a paid model. Your bill and your privacy surface both shrink to the hard tail.

For a desktop app, especially one handling data a user would not want uploaded, the model already on the Mac is the right first reach. It is free, it is private, it is offline, and for deciding what something is and handing you back a clean typed answer, it is enough. The classification I used to imagine paying an API for runs on the laptop now, and the bank statements never go anywhere.

---

*Marco Kotrotsos, specializing in practical AI implementation for organizations ready to close the gap between AI hype and AI value. With 30 years of IT experience now focused purely on AI deployment, he works hands-on with companies to turn AI potential into measurable business outcomes.*

*This article is published in [Autocomplete](https://medium.com/autocomplete-real-world-ai), a Medium publication about real-world AI for practitioners and decision-makers. We're always looking for writers. If you're building with AI and have something worth sharing, reach out.*

*My free Substack newsletter, also called Autocomplete, can be found here: https://acdigest.substack.com.*

*My books on Amazon: [Claude Code for Everyone Else](https://www.amazon.com/dp/B0H35YY851) and [From Vibe to Production](https://www.amazon.com/dp/B0H34GK9VW).*
