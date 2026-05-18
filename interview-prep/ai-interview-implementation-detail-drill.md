---
title: AI 面試實作細節 Drill — 30 題
purpose: micro1 retake (2026-06-08) + Centific 3 rounds 共用實作細節練習
audience: self
language: ZH + EN
created: 2026-05-10
---

# AI 面試實作細節 Drill — 30 題

## 使用方式

每題 30min 流程：
1. 先白紙寫 code（不看下方答案）
2. 出聲講中文 60s
3. 出聲講英文 60s
4. 對照「Hand-wave 陷阱」自我檢查

**核心原則**：講出**精準 syntax / API 名 / 數值門檻**。AI 面試官扣分點不在你不會，而在你「描述模糊」。

---

# Section 1 — Advanced TypeScript (1-7)

## Q1. Discriminated Union

**Code**

```ts
type Action =
  | { kind: 'add'; n: number }
  | { kind: 'reset' }
  | { kind: 'set'; value: number };

function reduce(state: number, action: Action): number {
  switch (action.kind) {
    case 'add':   return state + action.n;
    case 'reset': return 0;
    case 'set':   return action.value;
    default:
      const _exhaustive: never = action;
      return state;
  }
}
```

**60s 講稿（中文）**

Discriminated union 是用一個共同的 literal 欄位（這裡叫 `kind`）當 discriminator，TypeScript 會在 switch 或 if 裡根據這個欄位自動 narrow 到對應的 variant。好處有兩個：第一，每個 case 裡 `action` 的型別會自動縮窄，例如 `kind === 'add'` 時編譯器知道有 `n: number`；第二，配合 `never` 型別宣告 `_exhaustive`，當未來新增 variant 但忘了處理時，編譯會直接報錯，達到 exhaustive check。比起 XOR props 的繞圈寫法，discriminated union 更直接、IDE 推導更穩。

**60s Script (English)**

A discriminated union uses a common literal field — here `kind` — as the discriminator. TypeScript narrows the union to the matching variant inside `switch` or `if`. Two payoffs: first, the variant fields become accessible without casts — when `kind === 'add'`, the compiler knows `n: number` exists. Second, assigning the unhandled value to a `never`-typed variable forces an exhaustive check — adding a new variant without updating the switch becomes a compile-time error. Compared to XOR-prop tricks, discriminated unions read cleaner and the IDE narrows them reliably.

**Hand-wave 陷阱**
- 講「這樣 TS 會知道」→ 改說「TS 在 switch case 內把 `Action` narrow 到對應 variant」
- 漏掉 `never` exhaustive check 的價值 — 必須講出「未來新 variant 編譯期報錯」
- 把 discriminated union 跟 type guard function 混為一談

---

## Q2. XOR Props (互斥 props)

**Code**

```ts
type Without<T, U> = { [K in Exclude<keyof T, keyof U>]?: never };
type XOR<T, U> = (Without<T, U> & U) | (Without<U, T> & T);

type ButtonAsLink   = { href: string; target?: string };
type ButtonAsAction = { onClick: () => void };

type ButtonProps = XOR<ButtonAsLink, ButtonAsAction>;

const a: ButtonProps = { href: '/x' };           // OK
const b: ButtonProps = { onClick: () => {} };    // OK
// const c: ButtonProps = { href: '/x', onClick: () => {} }; // ❌ 編譯錯
```

**60s 講稿（中文）**

XOR props 用在「兩組 props 必須互斥」的元件，例如 Button 不是 link 就是 action button，給 href 跟 onClick 同時存在語意上不對。實作核心是 `[K in Exclude<...>]?: never`，把對方的 keys 強制標成 `never`，這樣使用者只要嘗試同時傳就編譯失敗。比 runtime assert 早一個階段擋住錯用。Discriminated union 可以替代，但 XOR 的優勢是不需要使用者多寫 `kind` 欄位，API 比較乾淨。

**60s Script (English)**

XOR props enforce that two prop groups are mutually exclusive — for example, a Button is either a link or an action button; passing both `href` and `onClick` is semantically wrong. The trick is `[K in Exclude<...>]?: never`, which marks the opposite group's keys as `never`, so any attempt to pass both fails at compile time. It catches misuse one stage earlier than a runtime assertion. A discriminated union can do the same job, but XOR keeps the API cleaner because users do not need to add a `kind` field.

**Hand-wave 陷阱**
- 講「就是兩組擇一」→ 要講出 `[K in Exclude]?: never` 機制
- 不知道 XOR 跟 discriminated union 的取捨 — XOR 適合「使用者已經很習慣的 API 形狀」，DU 適合「reducer / state machine」

---

## Q3. `unknown` vs `any`

**Code**

```ts
function parseJSON(text: string): unknown {
  return JSON.parse(text);
}

const data = parseJSON('{"n":1}');
// data.n;            // ❌ 'data' is of type 'unknown'
if (typeof data === 'object' && data !== null && 'n' in data) {
  // data: object, narrowed
  console.log((data as { n: number }).n);
}

const dirty: any = JSON.parse('{}');
dirty.whatever.no.check; // ✅ 編譯不報錯，runtime 爆炸
```

**60s 講稿（中文）**

`unknown` 跟 `any` 都接受任何值，差別在「使用前是否被強制 narrow」。`any` 完全跳過型別檢查，可以直接 access 任何欄位；`unknown` 要求你先用 `typeof`、`in`、`instanceof` 或 type guard 收斂後才能用。實務上 API response、`JSON.parse`、postMessage payload 都應該標 `unknown` 而非 `any`，因為這些值是 trust boundary 外來的，強制 narrow 等於強迫你在 boundary 寫驗證。`any` 的合法用途幾乎只剩漸進式 migration 的暫時 escape hatch。

**60s Script (English)**

Both `unknown` and `any` accept any value; the difference is whether the compiler forces you to narrow before use. `any` skips type checking entirely — you can access any field. `unknown` blocks all access until you narrow with `typeof`, `in`, `instanceof`, or a type guard. In practice, API responses, `JSON.parse`, and postMessage payloads should be typed as `unknown`, never `any`, because they cross a trust boundary — forcing narrowing forces you to validate at that boundary. The legitimate use of `any` is basically just a temporary escape hatch during migration.

**Hand-wave 陷阱**
- 講「`unknown` 比較安全」→ 要講出「強制 narrow 才能 access」
- 漏掉 trust boundary 的概念
- 不知道 `any` 會污染整條 chain（`a.b.c.d` 全部 any）

---

## Q4. `null` vs `undefined`、`??` vs `||`

**Code**

```ts
const a = null ?? 'default';        // 'default'
const b = undefined ?? 'default';   // 'default'
const c = 0 ?? 'default';           // 0       ← 重點
const d = '' ?? 'default';          // ''      ← 重點

const e = 0 || 'default';           // 'default' ← 跟 ?? 不同
const f = '' || 'default';          // 'default'

interface User { name: string; nickname?: string }
function display(u: User): string {
  return u.nickname ?? u.name;  // nickname 是空字串時會用空字串，不是 name
}
```

**60s 講稿（中文）**

`??` (nullish coalescing) 只在左值是 `null` 或 `undefined` 時 fallback，`||` (logical OR) 對所有 falsy 值 fallback，包含 `0`、`''`、`false`、`NaN`。這個差別在處理「合法的 falsy 值」時是 bug 來源，例如數量輸入 0、空字串 placeholder、開關 false，用 `||` 會被錯誤覆蓋。語意上的選擇：「使用者沒提供」用 `??`，「使用者提供了但是空的」用 `||`。在 TypeScript `strictNullChecks` 開啟下，`null` 跟 `undefined` 必須顯式處理，optional chaining `?.` 是配套。

**60s Script (English)**

`??` (nullish coalescing) falls back only when the left side is `null` or `undefined`. `||` (logical OR) falls back on any falsy value — `0`, `''`, `false`, `NaN`. The gap matters when zero, empty string, or false is a legitimate value: a quantity input of 0, a placeholder empty string, or a toggle set to false will be silently overwritten by `||`. Rule of thumb: "user did not provide" use `??`; "user provided but it is empty" use `||`. With `strictNullChecks` on, `null` and `undefined` must be handled explicitly, and optional chaining `?.` is the partner operator.

**Hand-wave 陷阱**
- 只講「差別」不講具體 falsy 列表
- 不會舉「0 / 空字串 / false」三個典型踩雷情境
- 漏掉 `??` 跟 `?.` 的搭配關係

---

## Q5. `as const` Literal Narrowing

**Code**

```ts
const tabs1 = ['home', 'profile', 'settings'];
// tabs1: string[]

const tabs2 = ['home', 'profile', 'settings'] as const;
// tabs2: readonly ['home', 'profile', 'settings']

type Tab = typeof tabs2[number];
// type Tab = 'home' | 'profile' | 'settings'

const config = { mode: 'dark', size: 'md' } as const;
// config.mode: 'dark'  (literal, not string)
```

**60s 講稿（中文）**

`as const` 把 literal 值的型別從寬鬆的 primitive 收窄到具體 literal。`['a','b']` 預設推成 `string[]`，加 `as const` 變成 `readonly ['a', 'b']`。組合 `typeof tabs[number]` 可以從陣列直接萃取 union type，避免 array 跟 type 兩處重複定義。物件加 `as const` 會把所有欄位變 readonly literal，常用在設定檔或常數 map。實務情境：定義事件名、route 列表、API endpoint enum 替代品。

**60s Script (English)**

`as const` narrows a literal expression from its widened primitive type to its exact literal type. `['a', 'b']` defaults to `string[]`; with `as const` it becomes `readonly ['a', 'b']`. Combining `typeof arr[number]` extracts a union directly from the array, eliminating the duplicate-definition problem of declaring the array and the type separately. Applied to an object, `as const` makes every field readonly and literal-typed. Typical uses: event-name registries, route lists, and as a lighter alternative to enums for API endpoints.

**Hand-wave 陷阱**
- 不會寫 `typeof arr[number]` 萃取 union
- 不知道 `as const` 也使欄位 readonly
- 跟 `const` 變數宣告搞混（變數 `const` 不影響型別寬窄）

---

## Q6. `never` 與 Exhaustive Switch

**Code**

```ts
type Shape =
  | { kind: 'circle'; r: number }
  | { kind: 'square'; size: number };

function area(s: Shape): number {
  switch (s.kind) {
    case 'circle': return Math.PI * s.r ** 2;
    case 'square': return s.size ** 2;
    default:
      const _: never = s;  // 新增 variant 後沒處理 → 編譯期報錯
      throw new Error(`Unhandled: ${JSON.stringify(s)}`);
  }
}
```

**60s 講稿（中文）**

`never` 是 bottom type，代表「絕對不可能存在的值」。用在 exhaustive switch 的 default 分支，把 `s` assign 給 `never` 變數：當所有 case 都列出時，`s` 在 default 已經被 narrow 成 `never`，賦值合法；如果未來新增 variant（例如 `triangle`）忘了加 case，`s` 在 default 會是 `{kind:'triangle', ...}`，無法 assign 給 `never`，編譯失敗。這把「忘記處理」從 runtime 推到 compile time。配合 discriminated union 用最自然。

**60s Script (English)**

`never` is the bottom type — a value that cannot exist. In an exhaustive switch, the default branch assigns `s` to a `never`-typed variable. When every case is handled, `s` has been narrowed to `never` by the time control reaches default, and the assignment compiles. If a new variant is added later — say `triangle` — and the switch is not updated, `s` in default is `{kind:'triangle', ...}`, which cannot be assigned to `never`, and the build fails. This converts a forgotten case from a runtime bug into a compile-time error. Pairs naturally with discriminated unions.

**Hand-wave 陷阱**
- 講「`never` 就是不會發生」→ 要講出「default 分支自動 narrow 到 never」
- 不會解釋為何加 variant 後會編譯失敗的具體機制
- 把 `never` 跟 `void` 搞混（`void` 是 return nothing，`never` 是不會 return）

---

## Q7. Generic Constraint

**Code**

```ts
function pick<T extends object, K extends keyof T>(obj: T, keys: K[]): Pick<T, K> {
  const out = {} as Pick<T, K>;
  for (const k of keys) out[k] = obj[k];
  return out;
}

const u = { id: 1, name: 'a', secret: 'x' };
const safe = pick(u, ['id', 'name']);  // { id: number; name: string }
// pick(u, ['nonexistent']);            // ❌ 編譯錯
```

**60s 講稿（中文）**

Generic constraint 用 `extends` 限制型別參數的形狀。`<T extends object>` 確保 `T` 是物件型；`<K extends keyof T>` 確保 `K` 必須是 `T` 的 key，這樣呼叫端傳入不存在的 key 會在編譯期被擋下。回傳型別 `Pick<T, K>` 則自動萃取對應子型別。比起接受 `Record<string, unknown>` 然後 runtime check，constraint 把錯誤前移、IDE 自動補 key 名。實務常見：表格欄位 selector、API field projector、form `register` 函式。

**60s Script (English)**

Generic constraints use `extends` to restrict the shape of a type parameter. `<T extends object>` requires `T` to be an object type; `<K extends keyof T>` requires `K` to be a key of `T`, so passing a non-existent key fails at compile time. The return type `Pick<T, K>` extracts the matching sub-shape automatically. Compared with accepting `Record<string, unknown>` and validating at runtime, constraints push the error left and give the IDE autocomplete on key names. Typical real-world uses: table column selectors, API field projectors, and form `register` functions.

**Hand-wave 陷阱**
- 講「就是限制型別」→ 要講出 `keyof T` 跟 `Pick<T, K>` 的搭配
- 不知道為何 IDE 可以自動補 key
- 漏講「錯誤前移到 compile time」的價值主張

---

# Section 2 — Cross-Browser (8-14)

## Q8. Feature Detection

**Code**

```ts
// ✅ Feature detection
if ('IntersectionObserver' in window) {
  const io = new IntersectionObserver(cb);
}

if (typeof window.fetch === 'function') {
  fetch(url);
}

if (CSS.supports('display', 'grid')) {
  el.style.display = 'grid';
}

// ❌ UA sniffing — fragile
if (navigator.userAgent.includes('Chrome')) { /* assume feature exists */ }
```

**60s 講稿（中文）**

Feature detection 是直接檢查 API 是否存在，而不是猜瀏覽器型號。三種寫法：`'X' in window` 檢查全域屬性、`typeof X === 'function'` 檢查呼叫得動、`CSS.supports(prop, value)` 檢查 CSS 特性。為什麼不用 UA sniff？因為 UA 字串可被改、新瀏覽器（Edge Chromium）會偽裝、版本爆炸難維護。實務組合：feature detect 通過走主路徑、不通過走 polyfill 或 graceful degradation；不應該根據瀏覽器名稱分支。

**60s Script (English)**

Feature detection checks whether an API actually exists, instead of guessing the browser. Three idioms: `'X' in window` for global properties, `typeof X === 'function'` for callable APIs, and `CSS.supports(prop, value)` for CSS features. Why not UA sniff? UA strings can be spoofed, new browsers like Chromium-based Edge masquerade, and version matrices explode. The pattern is: detect → take the happy path if supported, otherwise polyfill or degrade gracefully. Branching on browser name is a maintenance trap.

**Hand-wave 陷阱**
- 只講「檢查瀏覽器」→ 要講出三種具體 idiom
- 不會講為何 UA sniff 不可靠
- 不知道 `CSS.supports()` 存在

---

## Q9. Promise-based Script Loader

**Code**

```ts
function loadScript(src: string): Promise<void> {
  return new Promise((resolve, reject) => {
    const s = document.createElement('script');
    s.src = src;
    s.async = true;
    s.onload = () => resolve();
    s.onerror = () => reject(new Error(`Failed to load ${src}`));
    document.head.appendChild(s);
  });
}

// 用法
await loadScript('https://cdn.example.com/lib.js');
window.MyLib.init();
```

**60s 講稿（中文）**

Promise-based script loader 用三步：建 `<script>` element、塞 `src` 跟 `async`、註冊 `onload`/`onerror` 後 append 到 DOM。回傳 Promise 讓呼叫端可以 `await`。陷阱有三個：第一，沒設 `async` 可能 block render；第二，`onerror` 只會在網路 / 4xx 觸發，script 內部 throw 不會觸發；第三，重複 load 同 URL 應該 cache（可以維護一個 `Map<src, Promise>`）。實務用途：lazy load 第三方 SDK、按需載入 chart 函式庫、polyfill 動態注入。

**60s Script (English)**

A Promise-based script loader has three steps: create a `<script>` element, set `src` and `async`, attach `onload` and `onerror` handlers, then append to the DOM. The returned Promise lets the caller `await` it. Three traps: first, omitting `async` can block rendering; second, `onerror` only fires for network or 4xx failures — exceptions thrown inside the script do not trigger it; third, repeated loads of the same URL should be cached, typically with a `Map<src, Promise>`. Real uses: lazy-loading third-party SDKs, on-demand chart libraries, dynamic polyfill injection.

**Hand-wave 陷阱**
- 漏掉 `async = true`
- 不講 `onerror` 的限制（內部 throw 抓不到）
- 不會提 dedup cache pattern

---

## Q10. CSS `@supports` 跟 `@media`

**Code**

```css
/* Feature query */
@supports (display: grid) {
  .layout { display: grid; grid-template-columns: 1fr 1fr; }
}
@supports not (display: grid) {
  .layout { display: flex; }
}

/* 組合 */
@supports (display: grid) and (gap: 1rem) {
  .layout { gap: 1rem; }
}

/* Media query */
@media (prefers-color-scheme: dark) { :root { --bg: #000; } }
@media (prefers-reduced-motion: reduce) { * { animation: none !important; } }
```

**60s 講稿（中文）**

`@supports` 是 CSS 版的 feature detection，瀏覽器解析後判斷 property + value 的組合是否合法。可以用 `not`、`and`、`or` 組合條件。跟 JS 的 `CSS.supports()` 對應，差在 `@supports` 寫在 stylesheet 裡、不需要 JS 介入。`@media` 不只看 viewport，還可以查 `prefers-color-scheme`、`prefers-reduced-motion`、`hover` 能力。實務：grid 的 progressive enhancement、暗色模式、無障礙降低動畫。

**60s Script (English)**

`@supports` is CSS-side feature detection — the browser parses and checks whether a property/value pair is valid. It composes with `not`, `and`, `or`. The JS equivalent is `CSS.supports()`; the difference is `@supports` lives in the stylesheet without JS involvement. `@media` covers more than viewport — it also queries `prefers-color-scheme`, `prefers-reduced-motion`, and `hover` capability. Real uses: progressive enhancement for grid, dark mode, accessibility-driven motion reduction.

**Hand-wave 陷阱**
- 不知道 `@supports not (...)` 寫法
- 漏講 `@media (prefers-*)` 系列
- 跟 `@media` 混為一談

---

## Q11. UA-Client Hints vs UA String

**Code**

```ts
// 新做法：UA-Client Hints
if ('userAgentData' in navigator) {
  const data = await (navigator as any).userAgentData.getHighEntropyValues([
    'platform', 'platformVersion', 'model'
  ]);
  console.log(data.platform);  // 'macOS'
}

// 舊做法：UA string parsing（fragile）
const isIOS = /iPad|iPhone|iPod/.test(navigator.userAgent);
```

**60s 講稿（中文）**

`navigator.userAgentData` 是新版 Client Hints API，回傳結構化資料而不是字串，且需要明確列出 high-entropy fields 才能拿到精確資訊（隱私保護）。優點：結構化、瀏覽器主動提供、未來可被權限控制。缺點：Safari / Firefox 還沒完整支援，需要 fallback 回 UA string。實務策略：優先用 `userAgentData`，fallback 用 UA string + library（例如 `bowser`）；最終目的是「**用 feature detection 取代 UA detection**」，UA 只剩在 logging、analytics、無 feature 等同物時用。

**60s Script (English)**

`navigator.userAgentData` is the newer Client Hints API — it returns structured data instead of a string, and high-entropy fields require an explicit request for privacy reasons. Pros: structured, browser-provided, can be permission-gated. Cons: Safari and Firefox lag in support, so a UA-string fallback is still needed. Strategy: prefer `userAgentData`, fall back to UA string plus a parser like `bowser`; the long-term goal is to **replace UA detection with feature detection**, leaving UA only for logging, analytics, or cases with no feature equivalent.

**Hand-wave 陷阱**
- 不知道 high-entropy 概念
- 不講 fallback 策略
- 沒講「UA detection 應該被 feature detection 取代」這個方向性結論

---

## Q12. Polyfill 判斷

**Code**

```ts
// ✅ 條件 polyfill
if (!Array.prototype.at) {
  Array.prototype.at = function(n: number) {
    return n < 0 ? this[this.length + n] : this[n];
  };
}

// ✅ 動態 import polyfill
if (!('IntersectionObserver' in window)) {
  await import('intersection-observer');  // side-effect polyfill
}

// ❌ 無條件 polyfill — 浪費 bytes
import 'core-js/stable';
```

**60s 講稿（中文）**

Polyfill 三原則：第一**檢查再 patch**，用 `if (!X)` 包住，避免覆蓋原生實作；第二**動態載入**，只有不支援的瀏覽器才下載 polyfill bundle，現代瀏覽器零負擔；第三**側效 import**，polyfill 本身改全域，不需要拿 export。實務工具：`core-js` 提供 spec-compliant polyfill、`@babel/preset-env` 配 `useBuiltIns: 'usage'` 自動依使用情況注入、Vite 的 `@vitejs/plugin-legacy` 可生雙 bundle（modern + legacy）。

**60s Script (English)**

Three rules for polyfills: first, **detect before patching** — wrap in `if (!X)` to avoid clobbering native implementations; second, **load dynamically** — only browsers that need the polyfill pay the bytes, modern ones get zero overhead; third, **side-effect import** — polyfills mutate globals, no exports to consume. Tooling: `core-js` provides spec-compliant polyfills, `@babel/preset-env` with `useBuiltIns: 'usage'` injects based on actual usage, Vite's `@vitejs/plugin-legacy` produces dual bundles (modern + legacy).

**Hand-wave 陷阱**
- 只講「就 import polyfill」→ 要講出條件檢查 + 動態載入
- 不知道 `core-js` 跟 `@babel/preset-env` 的關係
- 不會講 dual-bundle 策略

---

## Q13. Module Loading: `type="module"` vs `nomodule`

**Code**

```html
<!-- 現代瀏覽器執行 -->
<script type="module" src="/app.modern.js"></script>

<!-- 舊瀏覽器 fallback -->
<script nomodule src="/app.legacy.js"></script>
```

**60s 講稿（中文）**

`<script type="module">` 在支援 ES module 的瀏覽器執行（自動 deferred、嚴格模式、有作用域），同一個瀏覽器會忽略 `nomodule` 標記的 script。舊瀏覽器不認識 `type="module"`（直接 ignore）但會執行 `nomodule`。組合起來就是「modern/legacy dual bundle」：modern bundle 較小、用 ES2020 語法、tree-shaking 完整；legacy bundle 經 transpile + polyfill，只有 IE / 老 Safari 載。Vite 的 `@vitejs/plugin-legacy` 自動產出兩個 bundle 跟正確的 script tag。

**60s Script (English)**

`<script type="module">` runs only in browsers that support ES modules — automatically deferred, strict mode, scoped. Those browsers also ignore any `<script nomodule>`. Older browsers do not recognise `type="module"` (silently ignored) and execute the `nomodule` script. Combined, this is the "modern/legacy dual-bundle" pattern: the modern bundle is smaller, uses ES2020 syntax, and tree-shakes cleanly; the legacy bundle is transpiled and polyfilled, downloaded only by IE and older Safari. Vite's `@vitejs/plugin-legacy` emits both bundles and the correct script tags.

**Hand-wave 陷阱**
- 不知道兩個 tag 的「互斥執行」機制
- 不會講 module 自帶 deferred + strict
- 漏掉 dual-bundle 是建構工具自動產出

---

## Q14. `requestIdleCallback` Fallback

**Code**

```ts
type IdleCB = (deadline: { didTimeout: boolean; timeRemaining: () => number }) => void;

const ric: (cb: IdleCB, opts?: { timeout?: number }) => number =
  (window as any).requestIdleCallback ??
  ((cb: IdleCB) => setTimeout(() => cb({ didTimeout: false, timeRemaining: () => 50 }), 1));

ric(deadline => {
  while (deadline.timeRemaining() > 0 && tasks.length) {
    runOne(tasks.shift()!);
  }
}, { timeout: 2000 });
```

**60s 講稿（中文）**

`requestIdleCallback` 在瀏覽器空閒時跑 callback，避免擋住主線程。Safari 至今不支援，需要 fallback。Polyfill 用 `setTimeout` 模擬，假裝 deadline 永遠剩 50ms — 不完美但可用。`timeout` 參數確保最壞情況也會在指定 ms 內被呼叫（`didTimeout: true`）。實務情境：log 上報、analytics flush、低優先 prefetch、將大批 work 切片在多個 idle 期執行。**不要用** 在跟使用者互動相關的工作（按下按鈕後的 response），那應該走 `requestAnimationFrame` 或同步路徑。

**60s Script (English)**

`requestIdleCallback` runs callbacks during browser idle periods to avoid blocking the main thread. Safari still does not support it, so a fallback is needed. The polyfill simulates with `setTimeout`, pretending 50ms always remains — imperfect but workable. The `timeout` option guarantees the callback fires within the given ms in the worst case (`didTimeout: true`). Real uses: telemetry uploads, analytics flushing, low-priority prefetch, slicing large batches across multiple idle periods. **Do not use** for user-interactive work like button-click response — that belongs in `requestAnimationFrame` or the synchronous path.

**Hand-wave 陷阱**
- 不知道 Safari 不支援
- 不會解釋 `timeout` 參數的意義
- 把 idle callback 用在互動 critical path

---

# Section 3 — DevTools (15-22)

## Q15. Performance Panel — Long Task

**怎麼做**

1. 開 DevTools → Performance → record
2. 重現操作 → stop
3. 看 **Main thread** flame chart 找紅色三角（long task >50ms）
4. 點該 task → bottom-up view → 看哪個 function 佔最多 self time
5. 對照 source map 找到實際 code

**60s 講稿（中文）**

Performance panel 用來找 main thread blocking。錄製後在 main thread 軌道找紅色三角標記，那些是 long task（>50ms），會延遲 user input response，影響 INP 指標。點下去後切到 bottom-up view 按 self time 排序，找出真正耗時的 function（不是 caller）。常見元兇：大型 sync loop、未節流的 scroll handler、每幀重新計算的 layout。修法：把 work 切片用 `requestIdleCallback`、用 web worker 移出 main thread、virtualize 長 list。Frame 軌道顯示掉 frame，配對找 jank。

**60s Script (English)**

The Performance panel finds main-thread blocking. After recording, look for red triangles on the Main thread track — those are long tasks (>50ms) that delay user input response and hurt the INP metric. Click one, switch to bottom-up view sorted by self time, and find the actual hot function (not the caller). Common culprits: large synchronous loops, unthrottled scroll handlers, layout recalculated every frame. Fixes: slice work with `requestIdleCallback`, move to a Web Worker, virtualise long lists. The Frames track shows dropped frames — pair with the main track to localise jank.

**Hand-wave 陷阱**
- 不知道 long task 門檻是 50ms
- 講 top-down 而不是 bottom-up
- 漏講 INP 跟 long task 的關係

---

## Q16. Memory Panel — Heap Snapshot

**怎麼做**

1. DevTools → Memory → Heap snapshot → take snapshot (baseline)
2. 操作可疑流程（例如開關 modal 100 次）
3. 再 take snapshot
4. 切到 **Comparison** 視圖比對兩個 snapshot
5. 按 **Retained Size** 排序
6. 找新增的 detached DOM 節點 → 追 retainers chain 到 root

**60s 講稿（中文）**

Heap snapshot 找記憶體洩漏。流程是 baseline snapshot、做可疑操作、再 snapshot、用 comparison view 找 delta。關鍵欄位是 **retained size**（這個物件被釋放後可回收的總記憶體），比 shallow size 更能指出問題。看 **detached DOM** 類別找 unmount 後沒釋放的節點，沿著 retainers chain 往上追到根因（通常是某個 closure / event listener / setInterval 抓住）。實務常見洩漏：未 `removeEventListener`、未 `clearInterval`、Vue/React effect 沒 cleanup、global cache 無限長大。

**60s Script (English)**

Heap snapshots find memory leaks. The flow: baseline snapshot, perform the suspect action, snapshot again, then use the Comparison view to find the delta. The key column is **retained size** — total memory reclaimable when this object is freed — more diagnostic than shallow size. Inspect the **detached DOM** category to find unmount-survived nodes, then walk the retainers chain to the root cause (usually a closure, event listener, or `setInterval` holding the reference). Common leaks: missing `removeEventListener`, missing `clearInterval`, React/Vue effects without cleanup, unbounded global caches.

**Hand-wave 陷阱**
- 講 shallow size 不講 retained size
- 不知道 detached DOM 類別
- 不會說「沿 retainers chain 往上追」

---

## Q17. Memory — Allocation Timeline

**怎麼做**

1. DevTools → Memory → **Allocations on timeline** → record
2. 操作 → stop
3. 看時間軸上的藍色長條（仍存活的 allocation）
4. 點某個藍條 → 看 allocation 的 stack trace → 找到觸發的 code location
5. 對照 closure / listener / interval

**60s 講稿（中文）**

Allocation timeline 跟 heap snapshot 互補：snapshot 看終態、timeline 看過程。錄製後時間軸有藍色長條代表「在錄製結束時還活著」的 allocation，灰色是已被 GC 回收的。點藍條展開可以看 allocation 那一刻的 JS stack trace，直接定位 source code 位置。比 snapshot 快定位「誰建的」、適合追「為什麼這東西沒被 GC」。實務：找 closure 抓住的大物件、event handler 不斷新增舊的沒移除、subscribe 後忘了 unsubscribe。

**60s Script (English)**

The allocation timeline complements heap snapshots: snapshots show the end state, the timeline shows the process. After recording, blue bars on the timeline represent allocations still alive at recording end; grey bars were GC'd. Clicking a blue bar reveals the JS stack trace at allocation time, jumping directly to source. Faster than snapshots for "who allocated this" and ideal for "why isn't this GC'd". Real uses: closures retaining large objects, event handlers being added without removing old ones, subscriptions never unsubscribed.

**Hand-wave 陷阱**
- 跟 heap snapshot 講不出差異
- 不知道藍 vs 灰的意義
- 漏講可以直接拿到 allocation stack trace

---

## Q18. Network Panel — Waterfall

**看什麼**

- **Queueing**: 瀏覽器排隊（連線數限制 6 per host）
- **Stalled**: 等 connection slot
- **DNS / Connect / SSL**: 建連階段
- **TTFB** (Time To First Byte): 後端反應時間
- **Content Download**: 實際傳輸

```bash
# Export HAR
DevTools → Network → 右鍵 → "Save all as HAR with content"
```

**60s 講稿（中文）**

Network panel waterfall 把每個 request 拆成幾個階段：queueing 等瀏覽器分配（HTTP/1.1 同 host 限 6 條）、stalled 等 connection slot、DNS/Connect/SSL 建連、TTFB 是 server 反應時間（後端問題的最大線索）、content download 實際傳輸。看 waterfall 找瓶頸方式：高 TTFB 找後端、高 queueing 考慮 HTTP/2 / domain sharding、長 download 看資源大小。匯出 HAR 可以離線分析或寄給後端。CDP 還能看 priority、protocol（h2/h3）、cache hit。

**60s Script (English)**

The Network panel waterfall breaks each request into stages: queueing waits for browser slot allocation (HTTP/1.1 caps at 6 per host); stalled waits for a connection slot; DNS/Connect/SSL is connection setup; TTFB is server response time — the strongest signal for backend issues; content download is actual bytes. Diagnosing a slow request: high TTFB → backend, high queueing → consider HTTP/2 or domain sharding, long download → resource size. HAR export enables offline analysis or sharing with backend. The panel also shows priority, protocol (h2/h3), and cache hits.

**Hand-wave 陷阱**
- 不知道 HTTP/1.1 6 per host 限制
- 不知道 TTFB 指後端
- 不會講 HAR 怎麼匯出

---

## Q19. Sources — Breakpoint 三類

**Code / Steps**

```
1. Line breakpoint        → 點行號
2. Conditional breakpoint → 右鍵行號 → "Add conditional breakpoint" → 輸入表達式
3. DOM breakpoint         → Elements panel → 右鍵節點 → Break on:
                             - subtree modifications
                             - attribute modifications
                             - node removal
4. XHR breakpoint         → Sources → XHR/fetch Breakpoints → "+" → 填 URL substring
5. Event listener breakpoint → Sources → Event Listener Breakpoints → 勾 "click"/"keydown"/...
```

**60s 講稿（中文）**

Sources panel 有四類進階 breakpoint：line（最基本）、conditional（只在 expression true 時 break，找 race condition 很好用）、DOM breakpoint（節點被改、屬性變、被移除時 break，追「誰把 DOM 改了」）、XHR/fetch breakpoint（指定 URL substring，知道哪段 code 發 request）、event listener breakpoint（按事件類別 break，找出「哪個 handler 跑了」）。實務情境：DOM 莫名被清空 → DOM breakpoint；「為何發了重複 request」→ XHR breakpoint；「點按鈕沒反應」→ click event breakpoint。

**60s Script (English)**

The Sources panel offers four advanced breakpoint types beyond line breakpoints: conditional (breaks only when an expression is true — useful for race conditions); DOM breakpoint (breaks on subtree modification, attribute change, or node removal — tracks down "who mutated the DOM"); XHR/fetch breakpoint (matches URL substring — locates the code firing a request); event listener breakpoint (breaks by event class — finds "which handler actually ran"). Real uses: DOM mysteriously cleared → DOM breakpoint; duplicate requests → XHR breakpoint; click does nothing → click event breakpoint.

**Hand-wave 陷阱**
- 只講 line breakpoint
- 不會講 DOM breakpoint 三種子類
- 不知道 event listener breakpoint 可以按類別勾選

---

## Q20. Coverage Tab — 找未用 JS/CSS

**怎麼做**

```
DevTools → Cmd+Shift+P → "Show Coverage" → reload + record
→ 看 unused bytes 比例
→ 點檔案進到 Sources，紅色行號 = 未執行
```

**60s 講稿（中文）**

Coverage tab 顯示哪些 JS / CSS 被執行了、哪些沒有。錄製載入後，每個檔案會有 used/unused bytes 比例。點進去 source 用紅綠槽顯示哪行跑過、哪行沒跑。實務用途：第一找「永遠沒用到的 module」可以 lazy load 或刪掉；第二抓「初始載入但首屏不需要的 component」改 dynamic import；第三找 dead CSS（特別是 framework 全載例如 Bootstrap 用到不到 10%）。注意：unused 不等於可刪 — 有些 code 是條件路徑，需要操作完整流程才會被計入。

**60s Script (English)**

The Coverage tab shows which JS and CSS executed and which did not. After recording a load, each file gets a used/unused-bytes ratio. Drilling into the source highlights run vs unrun lines in green/red. Real uses: first, find modules never executed and lazy-load or delete them; second, identify components loaded eagerly but not needed for first paint and convert to dynamic imports; third, locate dead CSS — especially with full-framework imports like Bootstrap where under 10% may actually be used. Caveat: unused does not equal deletable — conditional paths only register after exercising every flow.

**Hand-wave 陷阱**
- 不知道 Coverage tab 怎麼開（Cmd+Shift+P）
- 講 unused 直接刪 → 要講「unused 不等於 dead，要跑完所有流程」
- 不會講「lazy load 比刪除更實用」

---

## Q21. Lighthouse — Core Web Vitals 三指標

**Code / Reference**

```
LCP (Largest Contentful Paint) — 最大內容元素渲染時間
  目標: <= 2.5s
  常見問題: 大圖未壓縮、render-blocking CSS、TTFB 高

CLS (Cumulative Layout Shift) — 視覺偏移總量
  目標: <= 0.1
  常見問題: 圖片無 width/height、字體 swap、廣告插入推開內容

INP (Interaction to Next Paint) — 互動到下次繪製延遲
  目標: <= 200ms
  常見問題: long task 阻擋 main thread、handler 太重
```

**60s 講稿（中文）**

Core Web Vitals 三大指標：**LCP** 是最大可見內容（通常 hero image / H1）的渲染時間，目標 ≤2.5s，瓶頸常在大圖、render-blocking CSS、後端 TTFB；**CLS** 是 viewport 內視覺偏移總量，目標 ≤0.1，主因是 img/iframe 沒設尺寸、字體 swap、動態插入內容；**INP** 取代 FID，量測「互動到下次 paint」延遲，目標 ≤200ms，瓶頸是 long task 跟過重 handler。Lighthouse 跑出分數後，每個指標下會列具體 opportunity（例如「Eliminate render-blocking resources, save 1.2s」），是優化路線圖。

**60s Script (English)**

Core Web Vitals: **LCP** measures the largest visible content (typically hero image or H1) render time — target ≤2.5s — bottlenecks are heavy images, render-blocking CSS, high backend TTFB. **CLS** is cumulative visual shift inside the viewport — target ≤0.1 — main causes: images/iframes without dimensions, font swap, content injected after layout. **INP** replaced FID, measuring "interaction to next paint" latency — target ≤200ms — capped by long tasks and heavy handlers. Lighthouse runs surface a score plus specific opportunities per metric (e.g., "Eliminate render-blocking resources, save 1.2s"), forming the optimisation roadmap.

**Hand-wave 陷阱**
- 講不出 2.5s / 0.1 / 200ms 三個門檻
- 不知道 INP 已取代 FID
- 不會把指標對應到具體優化動作

---

## Q22. React DevTools Profiler

**怎麼做**

```
React DevTools → Profiler → record → 操作 → stop
→ 看 Flamegraph: 每個 commit 一個 bar，bar 寬 = 渲染時長
→ 點 component → "Why did this render?"（需要 setting 開啟）
→ 看 Ranked: 按渲染時長排序找熱點
```

**60s 講稿（中文）**

React DevTools Profiler 量測每次 commit 的渲染成本。Flamegraph view 顯示每個 commit 是一個 bar，寬度代表 component 樹渲染總時長；點 component 看 "Why did this render?"（要在 settings 開啟），會顯示 props/state/hook/parent 哪個觸發。Ranked view 按單個 component render 時長排序找熱點。常見優化目標：parent 重 render 導致 children 全重 render → `React.memo`；inline object/function prop 每次都新 ref → `useMemo` / `useCallback`；context value 每次新物件 → 拆 context 或 memoise value。**注意**：不是每個 re-render 都壞，重點看「是否導致 jank」。

**60s Script (English)**

The React DevTools Profiler measures per-commit render cost. The Flamegraph view shows each commit as a bar, width equal to the component tree's render time. Clicking a component reveals "Why did this render?" (must be enabled in settings), naming whether props, state, hooks, or parent triggered it. The Ranked view sorts components by individual render time to find hotspots. Common optimisations: parent re-render cascading to all children → `React.memo`; inline object/function props creating new refs every render → `useMemo` / `useCallback`; context value as a fresh object each time → split context or memoise the value. **Caveat**: not every re-render is bad — only fix the ones that cause jank.

**Hand-wave 陷阱**
- 不知道要先在 settings 啟用 "Why did this render?"
- 講「memo 一切就好」→ 要講「不是每個 re-render 都壞」
- 不會分辨 Flamegraph vs Ranked 用途

---

# Section 4 — Mock + Integration (23-30)

## Q23-26. Mock Interview 4 場

每場 30min，AI 當 interviewer，從 Q1-Q22 隨機抽 5-7 題。

**Mock prompt（餵給 Cursor / Claude）**

```
你扮演 micro1 / Centific 的 AI 面試官。從以下 22 題隨機選 6 題，
每題等我口頭回答（我會打字），完成後對我評分：
- 是否講出精準 syntax / API 名 / 數值門檻
- 哪裡 hand-wave，要怎麼改
- 給我「如果這是真實面試，會不會通過」的判斷

題目清單：[貼 Q1-Q22 的標題]
```

**自我評分標準**

- ✅ 講得出 syntax + 數值門檻 + 取捨
- ⚠️ 知道概念但講不出精準 API 名
- ❌ 完全 hand-wave

目標：4 場下來 ≥80% 題目達 ✅。

---

## Q27-30. 重做最弱 4 題

從 mock 的 ⚠️ + ❌ 題目挑出 4 題，重新走完整 30min drill 流程：白紙寫 code → 中文 60s → 英文 60s → AI 批 → 記錄 hand-wave 點。

**追蹤表（建在 drill log）**

| Date | Topic | First-pass | Re-do | Status |
|------|-------|------------|-------|--------|
| 2026-05-11 | Q1 Discriminated union | ⚠️ | ✅ | done |
| ... | | | | |

---

# 收尾

## 預期成果

- 4 週後實作細節描述能力 ≥80% 題目達標
- micro1 retake 2026-06-08 通過率顯著提升
- Centific 3 rounds 技術問答有具體 syntax 可以引用
- 同步累積 STAR 故事的 Result 段佐證材料（「我能在 60s 內精準解釋 X」本身就是 demo）

## 不做什麼

- 不背 spec 條文 — 重點是描述能力，不是記憶
- 不在白紙寫 code 階段查 doc — 失去意義
- 不只用中文或只用英文 — AI 面試多半英文，Centific 可能英文，micro1 一定英文

## 配套

- Drill log 檔（每日打卡）— 用 Cursor 開個 sticky scratch
- 60s 錄音 — 手機 voice memo 即可，週末回聽找 filler word
- 每週 retro — 哪些題反覆卡，調整下週順序

---

**最後備忘**：實作細節描述能力 = 把「我大概知道」變成「我能在 60 秒內講出 syntax + 數值 + 取捨」。每天 30 分鐘，4 週可達標。
