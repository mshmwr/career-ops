---
company: micro1
role: Frontend Engineer
application: #31
interview_date: 2026-05-08
result: Rejected
retake_eligible: 2026-06-08
saved: 2026-05-10
---

# micro1 — Frontend Engineer (Rejection Feedback)

## Verdict

Rejected. Re-apply allowed:
- **Different skill set** — immediate
- **Same skill set** — retake on **2026-06-08**

## Strengths (per micro1)

1. **TypeScript foundation** — interface for extension, union types for flexible props. On right path, needs sharper articulation.
2. **Web perf + cross-browser** — fallbacks, conditional loading, polyfill judgement. Mindset solid.
3. **Frontend debugging** — Network/Sources panels, React DevTools, perf+stability triangulation. Problem-solving demonstrated.

## Weak spots (cite verbatim from feedback)

1. **Advanced TS patterns** — discriminated unions, XOR props, `unknown` vs `any` vs `null`/`undefined` distinctions explained imprecisely.
2. **Cross-browser specifics** — feature detection checks, Promise-based script loaders missing step-by-step implementation.
3. **DevTools depth** — which panel for which bottleneck (perf vs memory leak), specific signals to look for.

## Root-cause read

Pattern: **conceptual fluent, implementation-detail thin**. Knows the *what*, less crisp on *exact API/check/signal*. AI interview format penalises hand-wave answers — wants concrete code/snippet/syntax.

## Drill list (before any retake)

- [ ] Discriminated union vs XOR props — write 3 examples each, explain when to use which
- [ ] `unknown` vs `any` vs `null` vs `undefined` — type narrowing examples, runtime vs compile-time
- [ ] Feature detection: `'IntersectionObserver' in window` style checks vs UA sniff — when each
- [ ] Promise-based script loader — write one from scratch (`new Promise`, `script.onload/onerror`)
- [ ] Chrome DevTools panel mapping:
  - Performance panel → main thread blocking, layout thrash
  - Memory panel → heap snapshot, retained size, detached DOM
  - Network panel → waterfall, TTFB, blocking resources
  - Sources → breakpoint types (DOM/XHR/event listener)
  - React DevTools Profiler → wasted renders, why-did-you-render

## Source feedback (raw)

Hi YICHEN,

Thank you for requesting feedback on your interview for the Frontend Engineer position. Below is a detailed breakdown of your performance, highlighting your strengths and areas for improvement.

Strengths we noticed

You demonstrated a strong foundation in TypeScript, especially in recognizing when to use interfaces for extension and union types for flexible prop handling. Your practical application of these concepts shows you are on the right path and with continued practice, you can explain these design patterns with even greater clarity.
Your answers on web performance and cross-browser compatibility reflected a good understanding of modern development strategies, such as leveraging fallbacks, conditional loading, and thoughtfully evaluating when a polyfill is necessary. Your performance-conscious mindset is a valuable asset and will serve you well as you refine your approach with more concrete implementation details.
You showcased a solid troubleshooting mindset in frontend debugging by utilizing tools like the Network and Sources panels, React DevTools, and considering both performance and stability concerns. Your approach to triangulating issues demonstrates effective problem-solving and highlights your readiness to tackle complex debugging scenarios.
Areas for Improvement
You could have explained discriminated unions, XOR props, and the distinctions between 'unknown', 'any', and null/undefined with more precision. Brushing up on these advanced TypeScript patterns and providing concise examples in future answers will make your explanations even more compelling.
Your responses regarding cross-browser strategies could have included more specific details on feature detection checks and the exact usage of Promise-based script loaders. Enhancing your explanations with step-by-step implementation approaches will showcase your depth of knowledge.
You could have described the specific use cases for different DevTools panels in diagnosing performance bottlenecks and memory leaks with greater accuracy. Offering more targeted examples of which signals to check for in each tool will help illustrate your technical depth during discussions.
What's next?

The good news is that you can re-apply! You have two options:
Immediately with a different set of skills that better represent your strengths
If you prefer to stick with the same skill set, you can retake the interview after 30 days

Retake with different skills

Retake interview with same skills on 08 Jun, 2026


We deeply appreciate your time and very much looking forward to your next application. If you need any guidance, feel free to reach us anytime, we're here to help! :)
