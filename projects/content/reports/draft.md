# When the Agent Draws the Screen: The Three Settings of Generative UI

*The agent can now render the interface instead of describing it. There are three ways to let it, on a spectrum from total control to none, and the choice is really a question of how tightly you set the bar on what it is allowed to draw.*

![An isometric scene: an abstract teal agent arm drawing a UI directly onto a screen in real time, three small framed panels beside it ranging from a tightly gridded layout on the left to a loose freeform sketch on the right, a coral control dial underneath with three notches](hero.png)

I have been building interfaces that the agent draws itself for a few months now. A small plugin that pops up a real form to ask me a visual question and reads my click back as structured data. Reports that ship as navigable sites instead of walls of text. I was doing it by feel, one pattern at a time, without a map of the space I was working in.

Then Shubham Saboo published a clean framework for it, and the names did what good names do: they made the thing I had been fumbling toward obvious. His framing is that generative UI, letting the agent render the interface rather than write a paragraph about it, comes in exactly three patterns on a spectrum from control to freedom. Credit to him for the shape. This piece is my read on it from the building side, plus the lens that made the choice click for me, which is that the three patterns are three settings of how tightly you hold the bar on what the agent may draw.

Ask for a table, get a table, not a sentence describing one. That is the whole promise. The question is who decides what the table looks like.

**Three things to take away:**

- Generative UI is not one technique, it is a spectrum with three points: Controlled (you pre-build, the agent picks), Declarative (the agent fills a schema you defined), and Open-ended (the agent writes raw HTML). Each breaks differently at scale, and most teams run one without ever choosing it.
- The honest way to choose is to ask how much you are willing to let the agent decide. Each pattern is a different amount of bar set on the output. Tightest on the left, none on the right.
- Token economics decide more than aesthetics. Controlled grows your context window linearly with every component you add. Declarative stays flat no matter how many UIs your catalog can produce. That difference is the wall most teams hit around twenty-five components.

![A spectrum from total control on the left to total freedom on the right, with three points marked: Controlled (you pre-build, the agent picks, the tightest bar), Declarative (the agent fills a catalog you defined, the catalog is the bar), and Open-ended (the agent writes raw HTML, no bar), with the bar getting looser left to right](diagram-spectrum.png)

## The plumbing, in one paragraph

A few protocols sit under all of this, and it is worth knowing which does what so the rest reads cleanly. MCP connects an agent to tools. A2A connects agents to each other. AG-UI connects the agent to the user: it is the streaming, bidirectional wire that carries UI events, tool calls, and state in both directions over a single connection, so a user edit reaches the agent and an agent change reaches the screen. A2UI is Google's spec for an agent emitting UI as a schema, and it rides on AG-UI. You do not write a parser for any of it. A client library, CopilotKit is the one Saboo uses and the one with reference code for all three patterns, decodes the stream for you. That is the entire stack you need to hold in your head.

## Setting 1: Controlled, you hold the bar tightest

This is where most teams start, because most frameworks default here. You build a component by hand, bind it to a name, and the agent's only job is to decide when to render it and with what data. The agent never touches your markup. It picks from a menu you wrote.

```tsx
// You pre-build the component, then register it so the agent can choose it.
registerComponent({
  name: "expenseBreakdown",
  // Describe it by the user's intent, not the visual. This matters later.
  description: "Use when the user asks to compare spending across categories.",
  props: { title: "string", rows: "{label: string, amount: number}[]" },
  render: ({ title, rows }) => <ExpenseBreakdown title={title} rows={rows} />,
});
```

The bar here is as tight as it gets. The agent cannot draw anything you did not already build and approve. Your design system stays fully in charge, and the output is pixel-perfect every time because you drew every pixel.

That tightness has a price, and it is paid in tokens. Every component you register sits in the agent's context window on every turn, before the user has typed a word, because the agent has to know the menu to pick from it. A tool description with its schema runs a few hundred tokens. Twenty-five of them is several thousand tokens of overhead on every single request, whether or not any UI gets rendered. Your context cost grows linearly with your component count, and that line is the wall.

The other failure is subtler. Past fifteen or so components, two of them start to sound alike. A pie chart and a donut chart both "show proportions," so the agent guesses, and guesses wrong often enough to annoy. The fix is a discipline, not a feature: describe each component by the user intent that should trigger it, never by what it looks like. "Use when the user asks to compare proportions of a whole" beats "renders a pie chart," because the agent is matching the user's words, not yours.

Ship Controlled for your ten or fewer highest-value flows, the ones where precision is the point and you already know the exact UI. Do not ship it as the way you handle everything, because the token bill and the component sprawl both scale with you.

## Setting 2: Declarative, the catalog is the bar

This is the pattern most production apps end up needing, and the one I would default to. Instead of pre-building every screen, the agent emits a schema describing the UI it wants, and your app maps that schema to real components through a catalog. One tool on the agent side. Many possible UIs on the frontend.

```tsx
// The agent returns a UI tree. Your catalog maps node types to components.
const catalog = {
  FlightCard: ({ airline, price, route }) => (
    <article className="rounded-xl border p-4">
      <header className="flex justify-between"><span>{airline}</span><span>{price}</span></header>
      <div className="text-sm text-muted">{route}</div>
    </article>
  ),
  // ...the rest of the components the agent is allowed to emit
};

// agent output (conceptual): { type: "FlightCard", props: { airline: "KLM", price: "€212", route: "AMS → JFK" } }
```

The bar here is the catalog. The agent has freedom inside it, it composes and fills the components however the conversation calls for, but it cannot emit anything the catalog does not define. The props are typed, so a malformed UI becomes a build error instead of a blank screen. You set the boundary once, in the catalog, and the agent works inside it for every UI it will ever produce.

The token math is the reason this scales. Fifty card types or five hundred, the agent still sees one tool: "emit a UI from the catalog." Your component library can grow without growing the per-turn context cost at all. That flat line is the whole argument for Declarative over Controlled once you are past the prototype.

It is also extensible in a way Controlled is not. The schema is just JSON, so the same agent output can render in React, Svelte, or anything else with a matching catalog, and any agent that already speaks the wire can drive it without touching agent code.

The trade-off is real and you should name it out loud: the agent owns the layout within your catalog, so the exact arrangement varies run to run. If you are shipping legal disclosures, regulated content, or marketing surfaces where pixel placement is non-negotiable, this is the wrong bucket, go back to Controlled for those. The classic silent failure here is a catalog ID mismatch: you built a custom card, but the identifier the agent targets and the one your frontend registers do not match exactly, so the frontend quietly falls back to a generic component with no error in the console. Match the strings on both sides and it works.

Declarative is built for the long tail: dashboards, search results, forms, cards, the hundred small widgets you will never have time to hand-build. More use cases than hours, and you care about the token bill past the demo. That is the case for it.

## Setting 3: Open-ended, no bar at all

The far end of the spectrum removes the catalog entirely. The agent writes raw markup and your app renders it. Two flavors live here: an MCP App, where the agent drives a UI surface that a server exposes (an agent drawing diagrams on a real canvas, for instance), and sandboxed HTML, where the agent simply writes the HTML and you render it inside a locked-down iframe so it cannot touch your session.

```tsx
// The agent returns an HTML string. Render it sandboxed so it cannot escape.
<iframe
  srcDoc={agentHtml}
  // allow it to run and submit, nothing else. Never allow-same-origin.
  sandbox="allow-scripts allow-forms"
/>
```

There is no bar here. The agent draws whatever it wants, and that is exactly the point and exactly the problem. The freedom is total, and so is the variance.

I tried running open-ended as the primary interface for an agent, in spirit, by letting it freestyle the HTML. It was neo-brutalist on Tuesday and an iOS-4 tribute on Wednesday. Style rules in the system prompt nudge the model toward your brand, they do not bind it, so the look kept drifting and the product felt unserious. Without a bar held on the output, the output is whatever aesthetic was loudest in the model's training that week.

Open-ended is not useless, it is misapplied when teams reach for it as a default because it demos well. It is the right call for one thing: throwaway interactions the user will see once and never again. "Show me how electrons work." "Make a weird chart of my last ten queries." "Visualize this API response." Disposable, one-shot, nobody-cares-what-it-looks-like surfaces. For those, the freedom is a feature and the variance does not matter.

One safety note that is easy to get wrong: set the iframe sandbox to allow scripts and forms and nothing else. Never grant allow-same-origin to agent-written HTML, because that hands markup the model invented access to your origin. The whole reason this pattern is safe is the sandbox, so do not open it.

## How to pick, on purpose

The decision is short once you frame it as how much bar you want to hold.

![A decision flow for choosing a generative-UI pattern: if a designer has pixel-perfect mockups for the flow, choose Controlled; if you have dozens of widgets and a long tail to cover, choose Declarative; if it is a one-shot disposable visualization, choose Open-ended; if unsure, default to Declarative, use Controlled for the top three flows, and never default to Open-ended](diagram-decision.png)

A designer handed you exact mockups for this flow? Controlled, hold the bar tight. Dozens of widgets and results to cover with no time to hand-build them? Declarative, set the bar once in the catalog. A one-shot visualization the user will never see twice? Open-ended, drop the bar on purpose. Cannot decide? Default to Declarative, promote your top three flows to Controlled, and never let Open-ended be the default.

![A comparison of the three settings across control, token cost as you scale, output predictability, and best fit: Controlled is highest control, linear token cost, exact output, best for top flows; Declarative is shared control, flat token cost, varies within the catalog, best for the long tail; Open-ended is lowest control, low fixed cost, unpredictable output, best for disposable one-shots](diagram-comparison.png)

If you are already shipping and unsure where you landed, count your render tools. Past fifteen, you are in Controlled and the wall is near, start moving the long tail to Declarative this week.

## It was a bar decision the whole time

The reason this maps so cleanly is that drawing a screen is the same problem as any other agent output. You are deciding how much of the result you specify in advance and how much you let the agent invent. Controlled specifies everything, the agent only selects. Declarative specifies the vocabulary, the agent composes within it. Open-ended specifies nothing, and you get nothing you can count on, which is fine when the output is disposable and a problem when it ships twice.

The mistake is not picking the wrong setting. It is not knowing you picked one. Teams default to Controlled because the framework does, hit the component wall, then grab Open-ended because it looks great in a demo, and neither move was a decision. Both were drift. Choose the setting the way you would choose any other bar: match how tightly you constrain the agent to how much the output matters. Tight for the flows that have to be exact, the catalog for the long tail, and no bar at all only for the things nobody will remember seeing.

Saboo's reference implementations for all three patterns are worth cloning if you want to feel the differences in your hands rather than read about them. The framework is his. The lens, that every one of these is a bar you are choosing how tightly to hold, is what made me stop picking by accident.

---

*With thanks to Shubham Saboo, whose three-pattern framework for generative UI prompted this piece. His reference code for all three patterns lives in the awesome-llm-apps repository.*

*Marco Kotrotsos, specializing in practical AI implementation for organizations ready to close the gap between AI hype and AI value. With 30 years of IT experience now focused purely on AI deployment, he works hands-on with companies to turn AI potential into measurable business outcomes.*

*This article is published in [Autocomplete](https://medium.com/autocomplete-real-world-ai), a Medium publication about real-world AI for practitioners and decision-makers. We're always looking for writers. If you're building with AI and have something worth sharing, reach out.*

*My free Substack newsletter, also called Autocomplete, can be found here: https://acdigest.substack.com.*

*My books on Amazon: [Claude Code for Everyone Else](https://www.amazon.com/dp/B0H35YY851) and [From Vibe to Production](https://www.amazon.com/dp/B0H34GK9VW).*
