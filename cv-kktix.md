# Yi-Chen Lee

**Frontend Engineer**

yichen.lee.20@gmail.com | +886-955-072-502 | [linkedin.com/in/yichenlee-career](http://www.linkedin.com/in/yichenlee-career) | [github.com/mshmwr](https://github.com/mshmwr) | Taiwan (UTC+8)

---

## Professional Summary

Frontend Engineer with 6+ years of software development experience, including 4+ years building high-traffic B2C web applications at Binance, the world's largest cryptocurrency exchange. Owned a configurable React/TypeScript verification platform that lifted 7-day KYC conversion from 12.19% to 20.03% and unblocked 5-country market expansion through a single shared frontend config layer. Adopted AI-assisted coding workflows in production at Binance (~50% time saved on schema-config delivery) and extended into directing a six-role agent pipeline from spec to production in [personal projects](https://k-line-prediction-app.web.app/).

---

## Core Competencies

React & TypeScript · Frontend Architecture · Design Systems & Component Libraries · A/B Testing & Experimentation · AI-assisted Development · Performance & Scalability · TDD (Jest, RTL, Playwright) · Code Review Culture

---

## Work Experience

### Binance — Frontend Engineer
**Nov 2021 - Present**

#### KYC Team (2022 - Present)

- Boosted 7-day KYC conversion rate from 12.19% to 20.03% by refactoring verification flows into a fully configurable React/TypeScript platform, replacing hardcoded logic with frontend-maintained configurations to dynamically render UI across all verification scenarios.
- Enabled new market expansion with a 14.5% lifetime KYC pass rate by centralizing scattered country-specific configurations across a 5-country verification flow, eliminating logic sprawl that made new country integrations error-prone.
- Led development of a compliance-driven migration flow, integrating 4 modules from the main flow into an undocumented scope and becoming the primary frontend owner for ongoing maintenance and feature enhancements.
- Drove data-informed product decisions through A/B testing, designing and implementing feature experiments to optimize user conversion funnels and validate UX improvements before full rollout.
- Developed and maintained high-traffic B2C web applications serving millions of users using React, TypeScript, Redux Toolkit, and monorepo architecture on large-scale, globally distributed production systems, supporting both mobile and desktop platforms.
- Maintained production stability and resolved critical incidents within 1 hour using log analysis and error monitoring tools.
- Reduced schema config development time by approximately 50% by building an AI-assisted coding workflow (Cursor, Claude Code) — feeding existing config files and Figma screenshots as context to generate new configs and component updates.
- Consistently delivered stable feature releases by collaborating with 20+ cross-functional partners (Backend, iOS, Android, QA, PM, UX) across multiple time zones, maintaining quality through Jest / React Testing Library coverage, rigorous self-testing, and peer code review.

#### Electron Team (Nov 2021 - 2022)

- Improved Webview app loading speed by 30% by designing and implementing a Webview Pool solution, enabling pre-loading and caching strategies for frequently accessed micro-apps.
- Reduced user memory consumption by developing single/multi-process functionality, allowing users to choose between performance and resource efficiency based on device capabilities.
- Expanded user accessibility to Arabic-speaking regions by implementing RTL (Right-to-Left) layout support, enhancing internationalization for the desktop application.
- Increased accessibility for visually impaired users by implementing CVD (Color Vision Deficiency) color scheme options, expanding the product's user demographic.
- Developed and maintained desktop trading applications using Electron with JavaScript and TypeScript, including feature development, architecture design, and debugging.

---

### International Games System Co., Ltd. (IGS) — Software Engineer
**Feb 2020 - Oct 2021**

- Accelerated feature development time by 95% by refactoring the event framework architecture, enabling faster iteration cycles for game event implementations.
- Reduced application release time by 80% by optimizing the CI/CD pipeline with Jenkins, automating build and deployment processes and shifting deploying tasks to non-developers.
- Improved app loading time by approximately 10% by reducing platform resource dependencies and optimizing the .apk/.ipa file size.
- Reduced rework and maintenance costs by developing reusable modules and establishing standardized component libraries for cross-project usage.

---

### WeHelp — Web Trainee
**Feb 2021 - Aug 2021**

- Acquired web development knowledge and skills in a 26-week frontend engineer bootcamp.
- Developed an e-commerce tourism website (Taipei Day Trip) with minimal guidance in 5 weeks.
- Implemented a reservation system with CMS for merchants, featuring multi-language support and calendar functionality.

---

## Projects

### K-Line Prediction | 2026

[Website](https://k-line-prediction-app.web.app/) — Solo-operator AI-directed delivery: 40+ tickets, 7 days, six-agent pipeline (PM · Architect · Engineer · Reviewer · QA · Designer), zero-to-production React/TS + FastAPI app.

- Designed the agent harness itself — wrote 6 role personas mapped to SDLC handoffs (a pre-commit script blocks ticket merges unless every required handoff document is present).
- Codified 10 governance rules, each traced to the bug ticket that triggered it — e.g., Pre-Design Dry-Run Proof (Architect verifies "API unchanged" claims against the base-branch source before delivery), Cross-Page Shared-Component Consistency (QA asserts identical DOM output across all consuming routes), Content-Alignment Gate (PM pauses pipeline until operator approves verbatim user-voice copy).
- Proved the harness on its own author — when drafting the project README, the Content-Alignment Gate paused the pipeline and forced operator approval on the user-voice copy before merge, demonstrating governance rules trigger on real work, not just retrospectives.
- Enforced design-as-source-of-truth — Designer owns the canonical design file; Engineer implements against exported specs; Reviewer runs line-by-line parity between spec and rendered UI.
- Delivered end-to-end full-stack with React + TypeScript + Vite on Firebase Hosting, FastAPI + Python on Cloud Run, plus Vitest + Playwright + pytest coverage and a pre-commit SSOT gate blocking role-table drift across roles.json → README → protocol doc.

---

## Education

**National Yang Ming Chiao Tung University** | Master of Science
*Institute of Multimedia Engineering, College of Computer Science* | 2017 - 2019

**National Sun Yat-sen University** | Bachelor of Engineering
*Department of Mechanical and Electromechanical Engineering* | 2013 - 2017

---

## Skills

- **Languages:** TypeScript, JavaScript (ES6+), HTML5, CSS3
- **Frameworks & Libraries:** React (Hooks, Router), Next.js, Redux, Redux Toolkit, Styled-Components, Tailwind CSS, Atomic CSS, Electron, Formik
- **Testing & TDD:** Jest, React Testing Library, Playwright, Cypress, Unit Testing, A/B Testing (Themis), Test-Driven Development
- **Dev Tools:** Git, NPM, Webpack, Babel, Chrome DevTools, Charles, Postman, Storybook
- **AI-assisted Coding:** Cursor, Claude Code, Codex — applied to daily development cycles for velocity optimization and code quality; experienced in agentic workflows and autonomous testing pipelines
- **Backend:** Node.js, Express, Fastify, Python, Flask, FastAPI, MySQL, RESTful APIs
- **DevOps & CI/CD:** Jenkins, AWS EC2, Firebase Hosting, Cloud Run, Shell Script, Kubernetes
- **Spoken Languages:** Mandarin (Native), English (Professional - TOEIC High Intermediate), Japanese (JLPT N2)
