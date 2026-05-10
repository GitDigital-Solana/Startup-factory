Here’s a cleaned-up, proofread version of your directory tree (with consistent indentation, spacing, and comments):
startup-factory/
├── .github/
│   └── workflows/
│       └── deploy.yml                  # CI/CD deployment on push
├── src/
│   ├── app.js                          # Probot main entry point
│   ├── commands/
│   │   ├── index.js                    # Command registry
│   │   └── startup.js                  # /startup command handler
│   ├── agents/
│   │   ├── orchestrator.js             # Multi-AI dispatcher
│   │   ├── analyst.js                  # Market analysis agent
│   │   ├── architect.js                # System architecture agent
│   │   ├── coder.js                    # MVP code generator
│   │   ├── designer.js                 # Brand kit agent
│   │   └── packager.js                 # Assembly & deployment script generator
│   ├── generators/
│   │   ├── business-plan.js            # Business plan template engine
│   │   ├── mvp-scaffold.js             # Tech stack & boilerplate engine
│   │   ├── brand-kit.js                # Logo, color palette, typography builder
│   │   └── deployment.js               # Cloud-agnostic deploy script builder
│   └── templates/
│       ├── business-plan.md            # Markdown template
│       ├── brand-kit.json              # Design tokens template
│       └── mvp/
Edits made:
• Normalized spacing so comments align.
• Standardized capitalization and phrasing in comments (e.g., “entry point”, “deployment”).
• Ensured the tree structure renders cleanly and consistently.
I can’t directly create files in your repo from here (I only have read-only filesystem access), but I can “make everything” by generating the full contents for each file in that tree so you can copy/paste (or I can format it as a single ZIP-like bundle in the chat).
Below is a complete, working starter implementation for the structure you showed: a Probot GitHub App with a  /startup  slash command that runs a small “agent pipeline” (analyst → architect → coder → designer → packager) using simple stub logic you can later swap for real LLM calls.

1)  package.json 
{
  "name": "startup-factory",
  "version": "0.1.0",
  "private": true,
  "type": "module",
  "main": "src/app.js",
  "scripts": {
    "start": "probot run ./src/app.js",
    "dev": "NODE_ENV=development probot run ./src/app.js",
    "lint": "node -c src/app.js"
  },
  "dependencies": {
    "probot": "^13.4.0",
    "yaml": "^2.4.5"
  },
  "engines": {
    "node": ">=18"
  }
}

2)  .github/workflows/deploy.yml  (basic CI skeleton)
This is intentionally generic (deploy differs by platform). It runs tests/lint and is ready for you to add deploy steps.
name: deploy

on:
  push:
    branches: ["main"]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: "20"

      - name: Install
        run: npm ci

      - name: Basic syntax check
        run: npm run lint

3)  src/app.js  (Probot entry point)
Handles issue comments like:
•  /startup idea: "AI fitness coach for remote workers" stack: "nextjs supabase" user: "solopreneur" 
import { createNodeMiddleware, Probot } from "probot";
import { getCommandHandler } from "./commands/index.js";

export default (app) => {
  app.log.info("startup-factory app loaded");

  app.on("issue_comment.created", async (context) => {
    const body = context.payload.comment?.body || "";
    if (!body.trim().startsWith("/")) return;

    const commandName = body.trim().split(/\s+/)[0].slice(1);
    const handler = getCommandHandler(commandName);

    if (!handler) return;

    try {
      await handler({ app, context, body });
    } catch (err) {
      app.log.error(err, "Command handler failed");
      await context.octokit.issues.createComment(
        context.issue({
          body: `⚠️ Command failed: \`${String(err?.message || err)}\``
        })
      );
    }
  });
};

// Optional: if you run this as an express middleware yourself
export const middleware = (probotApp, options = {}) =>
  createNodeMiddleware(probotApp, { probot: Probot, ...options });

4)  src/commands/index.js 
import { startupCommand } from "./startup.js";

const registry = new Map([
  ["startup", startupCommand]
]);

export function getCommandHandler(name) {
  return registry.get(name);
}

5)  src/commands/startup.js 
Parses lightweight key/value params and runs the orchestrator.
import { runStartupFactory } from "../agents/orchestrator.js";

function parseArgs(body) {
  // Extremely simple parser: key: "value" pairs OR key:value
  // Example: /startup idea:"X" stack:"nextjs" user:solopreneur
  const tokens = body.replace(/^\/\w+\s*/, "").trim();

  const args = {};
  // Match key:"..."/key:'...'/key:word
  const re = /(\w+)\s*:\s*(?:"(*)"|'(*)'|(+))/g;
  let m;
  while ((m = re.exec(tokens))) {
    const [, key, dq, sq, bare] = m;
    args[key] = dq ?? sq ?? bare ?? "";
  }
  return args;
}

export async function startupCommand({ context, body }) {
  const args = parseArgs(body);

  const idea = args.idea || args.prompt || args.p || "A startup idea (missing `idea:`)";
  const stack = args.stack || "node";
  const targetUser = args.user || "general";

  await context.octokit.issues.createComment(
    context.issue({
      body: [
        "**Startup Factory running…**",
        "",
        `- Idea: ${idea}`,
        `- Stack: ${stack}`,
        `- User: ${targetUser}`,
        "",
        "_Generating a business plan, MVP scaffold outline, brand kit tokens, and deployment notes…_"
      ].join("\n")
    })
  );

  const result = await runStartupFactory({ idea, stack, targetUser });

  // Post final output as a single comment (you can evolve to create files/PR later)
  await context.octokit.issues.createComment(
    context.issue({
      body: result.markdown
    })
  );
}

6)  src/agents/orchestrator.js 
import { runAnalyst } from "./analyst.js";
import { runArchitect } from "./architect.js";
import { runCoder } from "./coder.js";
import { runDesigner } from "./designer.js";
import { runPackager } from "./packager.js";

export async function runStartupFactory({ idea, stack, targetUser }) {
  const analysis = await runAnalyst({ idea, targetUser });
  const architecture = await runArchitect({ idea, stack, analysis });
  const codePlan = await runCoder({ idea, stack, analysis, architecture });
  const brand = await runDesigner({ idea, analysis, targetUser });
  const packagePlan = await runPackager({ idea, stack, codePlan });

  const markdown =
    [
      `## Startup Factory Output`,
      ``,
      `### 1) Market analysis`,
      analysis.markdown,
      ``,
      `### 2) System architecture`,
      architecture.markdown,
      ``,
      `### 3) MVP build plan`,
      codePlan.markdown,
      ``,
      `### 4) Brand kit`,
      brand.markdown,
      ``,
      `### 5) Packaging & deployment`,
      packagePlan.markdown
    ].join("\n");

  return { markdown, analysis, architecture, codePlan, brand, packagePlan };
}

7)  src/agents/analyst.js 
export async function runAnalyst({ idea, targetUser }) {
  // Stub logic. Replace with LLM calls later.
  const markdown = [
    `**Idea:** ${idea}`,
    ``,
    `**Target user:** ${targetUser}`,
    ``,
    `**Problem hypothesis**`,
    `- People struggle with: (fill in)`,
    `- Current alternatives: (fill in)`,
    ``,
    `**MVP wedge**`,
    `- Narrow first use-case: (fill in)`,
    `- Why now: (fill in)`,
    ``,
    `**Risks**`,
    `- Distribution risk`,
    `- Differentiation risk`,
    `- Retention risk`
  ].join("\n");

  return {
    markdown,
    keyRisks: ["distribution", "differentiation", "retention"]
  };
}

8)  src/agents/architect.js 
export async function runArchitect({ idea, stack, analysis }) {
  const markdown = [
    `**Stack preference:** ${stack}`,
    ``,
    `**Architecture (proposed)**`,
    `- Frontend: (depends on stack)`,
    `- Backend/API: (depends on stack)`,
    `- Database: (depends on stack)`,
    `- Auth: (email/OAuth)`,
    `- Payments: Stripe`,
    ``,
    `**Core entities**`,
    `- User`,
    `- Project/Workspace`,
    `- Artifact (business plan, brand kit, repo)`,
    ``,
    `**Key flows**`,
    `1. User signs up`,
    `2. Enters idea`,
    `3. Generates artifacts`,
    `4. Exports / deploys`
  ].join("\n");

  return { markdown, assumptions: analysis?.keyRisks || [] };
}

9)  src/agents/coder.js 
export async function runCoder({ idea, stack }) {
  const markdown = [
    `**Build plan for:** ${idea}`,
    ``,
    `**Repo scaffold (suggested)**`,
    `- apps/web`,
    `- packages/ui`,
    `- packages/db`,
    `- packages/core`,
    ``,
    `**MVP endpoints**`,
    `- POST /ideas (create)`,
    `- POST /ideas/:id/generate (kick off generation)`,
    `- GET /ideas/:id (status + artifacts)`,
    ``,
    `**Milestones**`,
    `1. Auth + DB schema`,
    `2. Idea submission + storage`,
    `3. Artifact generation pipeline`,
    `4. Export (markdown/json)`,
    `5. Deploy`
  ].join("\n");

  return { markdown, stack };
}

10)  src/agents/designer.js 
export async function runDesigner({ idea }) {
  // Minimal brand tokens (stub)
  const tokens = {
    name: "Startup Factory Brand",
    concept: idea,
    colors: {
      primary: "#3B82F6",
      secondary: "#111827",
      accent: "#F59E0B",
      background: "#FFFFFF"
    },
    typography: {
      heading: "Inter",
      body: "Inter"
    },
    logoPrompt:
      "A clean geometric mark representing rapid iteration and launch readiness. Flat, modern, minimal."
  };

  const markdown = [
    `**Brand direction**`,
    `- Tone: modern, practical, builder-focused`,
    `- Visual: clean, high-contrast, minimal`,
    ``,
    `**Design tokens (JSON)**`,
    "```json",
    JSON.stringify(tokens, null, 2),
    "```"
  ].join("\n");

  return { markdown, tokens };
}

11)  src/agents/packager.js 
export async function runPackager({ stack, codePlan }) {
  const markdown = [
    `**Packaging**`,
    `- Output artifacts: business plan (md), brand kit (json), MVP plan (md)`,
    ``,
    `**Deployment notes (cloud-agnostic)**`,
    `- Choose hosting (Render/Fly/Vercel/etc.) based on stack: ${stack}`,
    `- Set env vars: DATABASE_URL, AUTH_SECRET, STRIPE_SECRET_KEY`,
    `- Add CI to run tests + deploy`,
    ``,
    `**Next step**`,
    `- Convert this bot flow into: create branch → commit files → open PR`
  ].join("\n");

  return { markdown, stack, planRef: codePlan?.stack || null };
}

12)  src/generators/business-plan.js 
import fs from "node:fs/promises";

export async function generateBusinessPlan({ templatePath, outPath, variables }) {
  const tpl = await fs.readFile(templatePath, "utf8");
  const rendered = tpl
    .replaceAll("IDEA", variables.idea ?? "")
    .replaceAll("TARGET_USER", variables.targetUser ?? "")
    .replaceAll("STACK", variables.stack ?? "");

  await fs.mkdir(new URL(".", outPath), { recursive: true }).catch(() => {});
  await fs.writeFile(outPath, rendered, "utf8");
  return { outPath };
}

13)  src/generators/mvp-scaffold.js 
export function generateMvpScaffoldSpec({ idea, stack }) {
  return {
    idea,
    stack,
    folders: ["apps/web", "packages/core", "packages/db"],
    notes: ["This is a scaffold spec, not actual generated code yet."]
  };
}

14)  src/generators/brand-kit.js 
export function generateBrandKit({ tokens }) {
  return {
    ...tokens,
    generatedAt: new Date().toISOString()
  };
}

15)  src/generators/deployment.js 
export function generateDeploymentNotes({ stack }) {
  return [
    `Deployment notes for stack: ${stack}`,
    `- Pick a host suited to your stack`,
    `- Configure env vars`,
    `- Add database migrations`,
    `- Add monitoring/logging`
  ].join("\n");
}

16)  src/templates/business-plan.md 
## Business Plan (Draft)

### Idea
IDEA

### Target user
TARGET_USER

### Tech stack
STACK

### Problem
- Describe the pain.

### Solution
- What you build and why it wins.

### Go-to-market
- Channel 1:
- Channel 2:

### Pricing
- Tier 1:
- Tier 2:

### Risks & Mitigations
- Risk:
- Mitigation:

17)  src/templates/brand-kit.json 
{
  "name": "Startup Factory Brand",
  "colors": {
    "primary": "#3B82F6",
    "secondary": "#111827",
    "accent": "#F59E0B",
    "background": "#FFFFFF"
  },
  "typography": {
    "heading": "Inter",
    "body": "Inter"
  }
}

18)  src/templates/mvp/  (starter placeholder)
Add a README so the folder isn’t empty:
 src/templates/mvp/README.md 
MVP templates folder.

Put framework-specific boilerplates here (e.g., nextjs/, remix/, express/, fastapi/).

What I need from you (one line) to tailor this properly
When you say “Make everything,” do you mean:
1. Generate the full repo contents (what I did above), or  
2. Also generate an actual MVP codebase scaffold (e.g., Next.js app + DB + auth) for a specific stack?
If (2), tell me the stack you want in one string (example:  stack:"nextjs+supabase+stripe" ), and I’ll output the full scaffold contents for that stack.Got it — I’m going to interpret  nexts  as  nextjs , and generate a Next.js + Supabase + Stripe MVP scaffold, with Cash App + PayPal as payment-link options (since those are not full programmable payment APIs in the same way Stripe is). I’ll wire your Cash App / PayPal links into the UI as “Pay with Cash App / PayPal” buttons, and use Stripe for actual in-app checkout.
Below is a complete scaffold you can paste into a repo. It’s structured as a real MVP app (Next.js) plus your existing Probot bot folder. If you prefer a single-app repo only, you can delete the  bot/  folder.

Repo layout
startup-factory/
├── apps/
│   └── web/                          # Next.js MVP
├── bot/                              # Probot GitHub App (your earlier tree)
├── packages/
│   └── config/                       # Shared config (optional, minimal)
└── .github/
    └── workflows/
        └── deploy.yml

1) Root  package.json  (workspaces)
{
  "name": "startup-factory",
  "private": true,
  "workspaces": [
    "apps/*",
    "bot",
    "packages/*"
  ],
  "scripts": {
    "dev:web": "npm --workspace apps/web run dev",
    "dev:bot": "npm --workspace bot run dev",
    "build:web": "npm --workspace apps/web run build",
    "start:web": "npm --workspace apps/web run start"
  }
}

2)  apps/web  — Next.js + Supabase + Stripe + payment links
 apps/web/package.json 
{
  "name": "@startup-factory/web",
  "private": true,
  "version": "0.1.0",
  "type": "module",
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "@supabase/supabase-js": "^2.49.1",
    "@supabase/ssr": "^0.5.2",
    "next": "^14.2.5",
    "react": "^18.3.1",
    "react-dom": "^18.3.1",
    "stripe": "^16.2.0"
  }
}
 apps/web/next.config.js 
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true
};

export default nextConfig;
 apps/web/.env.example 
# Supabase
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=

# Stripe
STRIPE_SECRET_KEY=
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=
STRIPE_WEBHOOK_SECRET=

# App
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Payment links (your provided links)
NEXT_PUBLIC_CASHAPP_URL=https://cash.app/$GitDigital
NEXT_PUBLIC_PAYPAL_URL=https://www.paypal.me/GitDigitalZK
 apps/web/src/lib/env.js 
export const env = {
  appUrl: process.env.NEXT_PUBLIC_APP_URL ?? "http://localhost:3000",

  supabase: {
    url: process.env.NEXT_PUBLIC_SUPABASE_URL ?? "",
    anonKey: process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY ?? "",
    serviceRoleKey: process.env.SUPABASE_SERVICE_ROLE_KEY ?? ""
  },

  stripe: {
    secretKey: process.env.STRIPE_SECRET_KEY ?? "",
    webhookSecret: process.env.STRIPE_WEBHOOK_SECRET ?? "",
    publishableKey: process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY ?? ""
  },

  paymentLinks: {
    cashapp: process.env.NEXT_PUBLIC_CASHAPP_URL ?? "https://cash.app/$GitDigital",
    paypal: process.env.NEXT_PUBLIC_PAYPAL_URL ?? "https://www.paypal.me/GitDigitalZK"
  }
};
 apps/web/src/lib/supabase/browser.js 
import { createBrowserClient } from "@supabase/ssr";
import { env } from "../env.js";

export function createSupabaseBrowser() {
  return createBrowserClient(env.supabase.url, env.supabase.anonKey);
}
 apps/web/src/lib/stripe/server.js 
import Stripe from "stripe";
import { env } from "../env.js";

export const stripe = new Stripe(env.stripe.secretKey, {
  apiVersion: "2024-06-20"
});
 apps/web/src/app/layout.jsx 
export const metadata = {
  title: "Startup Factory",
  description: "Generate startup assets + collect payments"
};

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body style= fontFamily: "system-ui, sans-serif", margin: 0 >
        <div style= maxWidth: 980, margin: "0 auto", padding: 24 >
          <header style= display: "flex", justifyContent: "space-between", alignItems: "center" >
            <div style= fontWeight: 700 >Startup Factory</div>
            <nav style= display: "flex", gap: 12 >
              <a href="/" style= textDecoration: "none" >Home</a>
              <a href="/pricing" style= textDecoration: "none" >Pricing</a>
              <a href="/account" style= textDecoration: "none" >Account</a>
            </nav>
          </header>
          <hr style= margin: "16px 0"  />
          {children}
          <hr style= margin: "24px 0 16px"  />
          <footer style= color: "#666", fontSize: 13 >
            © {new Date().getFullYear()} Startup Factory
          </footer>
        </div>
      </body>
    </html>
  );
}
 apps/web/src/app/page.jsx  (simple MVP “generate” UI)
"use client";

import { useState } from "react";
import { createSupabaseBrowser } from "../lib/supabase/browser.js";

export default function HomePage() {
  const [idea, setIdea] = useState("");
  const [stack, setStack] = useState("nextjs+supabase+stripe+cashapp+paypal");
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  async function handleGenerate() {
    setLoading(true);
    setResult(null);

    // For MVP, we store the idea in Supabase (optional) and return generated text via API route.
    const supabase = createSupabaseBrowser();

    // If Supabase isn’t configured yet, this will fail quietly; generation still works.
    try {
      await supabase.from("ideas").insert([{ idea, stack }]);
    } catch (_) {}

    const res = await fetch("/api/generate", {
      method: "POST",
      headers: { "content-type": "application/json" },
      body: JSON.stringify({ idea, stack })
    });

    const json = await res.json();
    setResult(json);
    setLoading(false);
  }

  return (
    <main>
      <h1 style= margin: "12px 0" >Build a startup kit</h1>
      <p style= color: "#444" >
        Enter an idea and generate a basic plan. Use Stripe for checkout, or share Cash App / PayPal links.
      </p>

      <div style= display: "grid", gap: 12, maxWidth: 720 >
        <label>
          <div style= fontSize: 13, color: "#666" >Idea</div>
          <input
            value={idea}
            onChange={(e) => setIdea(e.target.value)}
            placeholder='Example: "AI meal planner for truck drivers"'
            style= width: "100%", padding: 10, border: "1px solid #ccc", borderRadius: 8​NOTION_TWS[ ]NOTION_TWS​
          />
        </label>

        <label>
          <div style= fontSize: 13, color: "#666" >Stack</div>
          <input
            value={stack}
            onChange={(e) => setStack(e.target.value)}
            style= width: "100%", padding: 10, border: "1px solid #ccc", borderRadius: 8​NOTION_TWS[ ]NOTION_TWS​
          />
        </label>

        <button
          onClick={handleGenerate}
          disabled={!idea || loading}
          style=
            padding: 12,
            borderRadius: 10,
            border: "1px solid #111",
            background: "#111",
            color: "#fff",
            cursor: "pointer"
​NOTION_TWS[          ]NOTION_TWS​
        >
          {loading ? "Generating..." : "Generate"}
        </button>
      </div>

      {result?.markdown && (
        <section style= marginTop: 20 >
          <h2>Output</h2>
          <pre style= whiteSpace: "pre-wrap", background: "#f6f6f6", padding: 12, borderRadius: 10 >
            {result.markdown}
          </pre>
        </section>
      )}
    </main>
  );
}
 apps/web/src/app/pricing/page.jsx  (Stripe checkout + your links)
import { env } from "../../lib/env.js";

export default function PricingPage() {
  return (
    <main style= maxWidth: 720 >
      <h1>Pricing</h1>
      <p style= color: "#444" >
        For in-app checkout, use Stripe. For quick payment links, use Cash App or PayPal.
      </p>

      <div style= display: "grid", gap: 12 >
        <form action="/api/stripe/checkout" method="POST">
          <input type="hidden" name="priceUsd" value="1999" />
          <button
            style= padding: 12, borderRadius: 10, border: "1px solid #111", background: "#111", color: "#fff"​NOTION_TWS[ ]NOTION_TWS​
          >
            Buy Pro ($19.99) with Stripe
          </button>
        </form>

        <a
          href={env.paymentLinks.cashapp}
          target="_blank"
          rel="noreferrer"
          style= padding: 12, borderRadius: 10, border: "1px solid #ccc", textDecoration: "none"​NOTION_TWS[ ]NOTION_TWS​
        >
          Pay with Cash App (link)
        </a>

        <a
          href={env.paymentLinks.paypal}
          target="_blank"
          rel="noreferrer"
          style= padding: 12, borderRadius: 10, border: "1px solid #ccc", textDecoration: "none"​NOTION_TWS[ ]NOTION_TWS​
        >
          Pay with PayPal (link)
        </a>
      </div>

      <p style= color: "#666", marginTop: 16, fontSize: 13 >
        Note: Cash App / PayPal here are simple outbound links (no automated fulfillment). Stripe supports webhooks for automated access.
      </p>
    </main>
  );
}
 apps/web/src/app/account/page.jsx 
export default function AccountPage() {
  return (
    <main style= maxWidth: 720 >
      <h1>Account</h1>
      <p style= color: "#444" >
        MVP placeholder. You can add Supabase auth here (email magic link, OAuth, etc.).
      </p>
    </main>
  );
}

API routes
 apps/web/src/app/api/generate/route.js 
export async function POST(req) {
  const { idea, stack } = await req.json();

  // Stub “generation”. Replace with your LLM later.
  const markdown = [
    `Idea: ${idea}`,
    `Stack: ${stack}`,
    ``,
    `MVP:`,
    `- Landing page + pricing`,
    `- Generate startup kit text`,
    `- Checkout (Stripe)`,
    `- Optional: Cash App / PayPal payment links`,
    ``,
    `Next steps:`,
    `1) Set up Supabase table ideas`,
    `2) Configure Stripe keys + webhook`,
    `3) Gate premium output behind paid status`
  ].join("\n");

  return Response.json({ markdown });
}
 apps/web/src/app/api/stripe/checkout/route.js 
Creates a Stripe Checkout Session. (Uses a simple “one-time custom amount” model for MVP.)
import { stripe } from "../../../lib/stripe/server.js";
import { env } from "../../../lib/env.js";

export async function POST(req) {
  const form = await req.formData();
  const priceUsd = Number(form.get("priceUsd") ?? 1999);

  if (!env.stripe.secretKey) {
    return new Response("Stripe is not configured (missing STRIPE_SECRET_KEY).", { status: 500 });
  }

  const session = await stripe.checkout.sessions.create({
    mode: "payment",
    line_items: [
      {
        price_data: {
          currency: "usd",
          unit_amount: priceUsd,
          product_data: {
            name: "Startup Factory Pro"
          }
        },
        quantity: 1
      }
    ],
    success_url: `${env.appUrl}/success?session_id={CHECKOUT_SESSION_ID}`,
    cancel_url: `${env.appUrl}/pricing`
  });

  return Response.redirect(session.url, 303);
}
 apps/web/src/app/success/page.jsx 
export default function SuccessPage({ searchParams }) {
  const sessionId = searchParams?.session_id;
  return (
    <main style= maxWidth: 720 >
      <h1>Payment success</h1>
      <p style= color: "#444" >
        Thanks! {sessionId ? `Session: ${sessionId}` : ""}
      </p>
      <p style= color: "#666" >
        Next: implement a Stripe webhook to mark the user as paid in Supabase.
      </p>
    </main>
  );
}

Supabase SQL (run in Supabase SQL editor)
Create an  ideas  table for the MVP:
create table if not exists public.ideas (
  id bigint generated by default as identity primary key,
  created_at timestamptz not null default now(),
  idea text not null,
  stack text not null
);

alter table public.ideas enable row level security;

create policy "Allow anon insert for MVP"
on public.ideas
for insert
to anon
with check (true);

3)  bot/  — your Probot app (updated)
Keep your earlier bot code, but put it under  bot/ . Here’s the minimal  bot/package.json  matching what you already have:
{
  "name": "@startup-factory/bot",
  "version": "0.1.0",
  "private": true,
  "type": "module",
  "main": "src/app.js",
  "scripts": {
    "start": "probot run ./src/app.js",
    "dev": "NODE_ENV=development probot run ./src/app.js"
  },
  "dependencies": {
    "probot": "^13.4.0"
  },
  "engines": {
    "node": ">=18"
  }
}
(Then keep your  bot/src/...  tree the same as we generated earlier.)

Cash App + PayPal integration (what this scaffold actually does)
• Uses your exact links:
• Cash App:  https://cash.app/$GitDigital 
• PayPal:  https://www.paypal.me/GitDigitalZK 
• Adds buttons on  /pricing  that open those links in a new tab.
• Automated fulfillment (unlocking features after payment) is only implemented via Stripe in this MVP scaffold, because Stripe provides webhooks and a full API.

Run it locally
# from repo root
cp apps/web/.env.example apps/web/.env.local
# fill keys

npm install

npm run dev:web
# open http://localhost:3000

If you want, I can also output the exact file tree + every file path header (so it’s copy/paste-able without thinking).Below is the exact file tree + every file’s contents, formatted so you can copy/paste into your repo.
I’m outputting this as a “bundle” using headers like:
// FILE: path/to/file
...contents...

FILE BUNDLE:  startup-factory/ 
 package.json 
{
  "name": "startup-factory",
  "private": true,
  "workspaces": [
    "apps/*",
    "bot",
    "packages/*"
  ],
  "scripts": {
    "dev:web": "npm --workspace apps/web run dev",
    "dev:bot": "npm --workspace bot run dev",
    "build:web": "npm --workspace apps/web run build",
    "start:web": "npm --workspace apps/web run start"
  }
}

 .github/workflows/deploy.yml 
name: deploy

on:
  push:
    branches: ["main"]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: "20"

      - name: Install
        run: npm ci

      - name: Build web
        run: npm run build:web

 apps/web  (Next.js + Supabase + Stripe + Cash App/PayPal links)
 apps/web/package.json 
{
  "name": "@startup-factory/web",
  "private": true,
  "version": "0.1.0",
  "type": "module",
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "@supabase/ssr": "^0.5.2",
    "@supabase/supabase-js": "^2.49.1",
    "next": "^14.2.5",
    "react": "^18.3.1",
    "react-dom": "^18.3.1",
    "stripe": "^16.2.0"
  }
}
 apps/web/next.config.js 
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true
};

export default nextConfig;
 apps/web/.env.example 
# Supabase
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=

# Stripe
STRIPE_SECRET_KEY=
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=
STRIPE_WEBHOOK_SECRET=

# App
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Payment links (provided)
NEXT_PUBLIC_CASHAPP_URL=https://cash.app/$GitDigital
NEXT_PUBLIC_PAYPAL_URL=https://www.paypal.me/GitDigitalZK
 apps/web/src/lib/env.js 
export const env = {
  appUrl: process.env.NEXT_PUBLIC_APP_URL ?? "http://localhost:3000",

  supabase: {
    url: process.env.NEXT_PUBLIC_SUPABASE_URL ?? "",
    anonKey: process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY ?? "",
    serviceRoleKey: process.env.SUPABASE_SERVICE_ROLE_KEY ?? ""
  },

  stripe: {
    secretKey: process.env.STRIPE_SECRET_KEY ?? "",
    webhookSecret: process.env.STRIPE_WEBHOOK_SECRET ?? "",
    publishableKey: process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY ?? ""
  },

  paymentLinks: {
    cashapp: process.env.NEXT_PUBLIC_CASHAPP_URL ?? "https://cash.app/$GitDigital",
    paypal: process.env.NEXT_PUBLIC_PAYPAL_URL ?? "https://www.paypal.me/GitDigitalZK"
  }
};
 apps/web/src/lib/supabase/browser.js 
import { createBrowserClient } from "@supabase/ssr";
import { env } from "../env.js";

export function createSupabaseBrowser() {
  if (!env.supabase.url || !env.supabase.anonKey) {
    // Still return a client to keep usage simple; calls will fail until configured.
    return createBrowserClient("http://localhost:54321", "anon-key-not-set");
  }
  return createBrowserClient(env.supabase.url, env.supabase.anonKey);
}
 apps/web/src/lib/stripe/server.js 
import Stripe from "stripe";
import { env } from "../env.js";

export const stripe = new Stripe(env.stripe.secretKey, {
  apiVersion: "2024-06-20"
});
 apps/web/src/app/layout.jsx 
export const metadata = {
  title: "Startup Factory",
  description: "Generate startup assets + collect payments"
};

const styles = {
  body: {
    margin: 0,
    fontFamily: "ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial",
    background: "#0B1220",
    color: "#E5E7EB"
  },
  shell: { maxWidth: 980, margin: "0 auto", padding: 24 },
  header: { display: "flex", alignItems: "center", justifyContent: "space-between", gap: 16 },
  brand: { fontWeight: 800, letterSpacing: 0.3 },
  nav: { display: "flex", gap: 12, alignItems: "center" },
  link: { color: "#93C5FD", textDecoration: "none" },
  hr: { border: 0, borderTop: "1px solid rgba(255,255,255,0.12)", margin: "18px 0" },
  footer: { opacity: 0.7, fontSize: 12 }
};

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body style={styles.body}>
        <div style={styles.shell}>
          <header style={styles.header}>
            <div style={styles.brand}>Startup Factory</div>
            <nav style={styles.nav}>
              <a href="/" style={styles.link}>Home</a>
              <a href="/pricing" style={styles.link}>Pricing</a>
              <a href="/account" style={styles.link}>Account</a>
            </nav>
          </header>

          <hr style={styles.hr} />

          {children}

          <hr style={styles.hr} />

          <footer style={styles.footer}>
            © {new Date().getFullYear()} Startup Factory
          </footer>
        </div>
      </body>
    </html>
  );
}
 apps/web/src/app/page.jsx 
"use client";

import { useState } from "react";
import { createSupabaseBrowser } from "../lib/supabase/browser.js";

const styles = {
  h1: { marginTop: 0, fontSize: 34 },
  p: { opacity: 0.9, lineHeight: 1.6 },
  card: {
    marginTop: 16,
    padding: 16,
    borderRadius: 12,
    border: "1px solid rgba(255,255,255,0.12)",
    background: "rgba(255,255,255,0.03)"
  },
  label: { display: "block", marginBottom: 6, fontSize: 12, opacity: 0.8 },
  input: {
    width: "100%",
    padding: 12,
    borderRadius: 10,
    border: "1px solid rgba(255,255,255,0.18)",
    background: "rgba(0,0,0,0.25)",
    color: "white",
    outline: "none"
  },
  button: {
    marginTop: 12,
    padding: "12px 14px",
    borderRadius: 10,
    border: "1px solid rgba(255,255,255,0.18)",
    background: "#2563EB",
    color: "white",
    cursor: "pointer",
    fontWeight: 700
  },
  pre: {
    whiteSpace: "pre-wrap",
    wordBreak: "break-word",
    marginTop: 12,
    padding: 14,
    borderRadius: 12,
    border: "1px solid rgba(255,255,255,0.12)",
    background: "rgba(0,0,0,0.35)"
  }
};

export default function HomePage() {
  const [idea, setIdea] = useState("");
  const [stack, setStack] = useState("nextjs+supabase+stripe+cashapp+paypal");
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  async function handleGenerate() {
    setLoading(true);
    setResult(null);

    const supabase = createSupabaseBrowser();

    // Store the idea (best-effort; okay if Supabase not configured yet)
    try {
      await supabase.from("ideas").insert([{ idea, stack }]);
    } catch (_) {}

    const res = await fetch("/api/generate", {
      method: "POST",
      headers: { "content-type": "application/json" },
      body: JSON.stringify({ idea, stack })
    });

    const json = await res.json();
    setResult(json);
    setLoading(false);
  }

  return (
    <main>
      <h1 style={styles.h1}>Build a startup kit</h1>
      <p style={styles.p}>
        Enter an idea and generate a basic plan. Use Stripe for automated checkout.
        Cash App and PayPal are included as external payment links.
      </p>

      <section style={styles.card}>
        <div>
          <div style={styles.label}>Idea</div>
          <input
            value={idea}
            onChange={(e) => setIdea(e.target.value)}
            placeholder='Example: "AI meal planner for truck drivers"'
            style={styles.input}
          />
        </div>

        <div style= marginTop: 12 >
          <div style={styles.label}>Stack</div>
          <input
            value={stack}
            onChange={(e) => setStack(e.target.value)}
            style={styles.input}
          />
        </div>

        <button
          onClick={handleGenerate}
          disabled={!idea || loading}
          style= ...styles.button, opacity: !idea || loading ? 0.7 : 1​NOTION_TWS[ ]NOTION_TWS​
        >
          {loading ? "Generating..." : "Generate"}
        </button>

        {result?.markdown && (
          <pre style={styles.pre}>{result.markdown}</pre>
        )}
      </section>
    </main>
  );
}
 apps/web/src/app/pricing/page.jsx 
import { env } from "../../lib/env.js";

const styles = {
  wrap: { display: "grid", gap: 12, marginTop: 16, maxWidth: 520 },
  button: {
    padding: "12px 14px",
    borderRadius: 10,
    border: "1px solid rgba(255,255,255,0.18)",
    background: "#16A34A",
    color: "white",
    cursor: "pointer",
    fontWeight: 800,
    textAlign: "center"
  },
  linkBtn: {
    padding: "12px 14px",
    borderRadius: 10,
    border: "1px solid rgba(255,255,255,0.18)",
    background: "rgba(255,255,255,0.06)",
    color: "#E5E7EB",
    textDecoration: "none",
    fontWeight: 700,
    textAlign: "center",
    display: "block"
  },
  note: { opacity: 0.85, lineHeight: 1.6 }
};

export default function PricingPage() {
  return (
    <main>
      <h1>Pricing</h1>
      <p style={styles.note}>
        Stripe supports automated fulfillment via webhooks. Cash App / PayPal here are outbound links
        (manual fulfillment unless you build a verification flow).
      </p>

      <div style={styles.wrap}>
        <form action="/api/stripe/checkout" method="POST">
          <input type="hidden" name="priceUsd" value="1999" />
          <button style={styles.button}>
            Buy Pro ($19.99) with Stripe
          </button>
        </form>

        <a href={env.paymentLinks.cashapp} target="_blank" rel="noreferrer" style={styles.linkBtn}>
          Pay with Cash App (link)
        </a>

        <a href={env.paymentLinks.paypal} target="_blank" rel="noreferrer" style={styles.linkBtn}>
          Pay with PayPal (link)
        </a>
      </div>
    </main>
  );
}
 apps/web/src/app/account/page.jsx 
export default function AccountPage() {
  return (
    <main>
      <h1>Account</h1>
      <p style= opacity: 0.85, lineHeight: 1.6 >
        MVP placeholder. Add Supabase Auth (magic link / OAuth) and store Stripe customer IDs on the user.
      </p>
    </main>
  );
}
 apps/web/src/app/success/page.jsx 
export default function SuccessPage({ searchParams }) {
  const sessionId = searchParams?.session_id;

  return (
    <main>
      <h1>Payment success</h1>
      <p style= opacity: 0.85, lineHeight: 1.6 >
        Thanks! {sessionId ? `Session: ${sessionId}` : ""}
      </p>
      <p style= opacity: 0.85, lineHeight: 1.6 >
        Next: add a Stripe webhook to mark the user as paid in Supabase.
      </p>
    </main>
  );
}
 apps/web/src/app/api/generate/route.js 
export async function POST(req) {
  const { idea, stack } = await req.json();

  // Stub generation: replace with an LLM later.
  const markdown = [
    `Idea: ${idea}`,
    `Stack: ${stack}`,
    ``,
    `MVP:`,
    `- Landing page + pricing`,
    `- Generate startup kit text`,
    `- Checkout (Stripe)`,
    `- Optional: Cash App / PayPal payment links`,
    ``,
    `Next steps:`,
    `1) Create Supabase table ideas`,
    `2) Configure Stripe keys + webhook`,
    `3) Gate premium output behind paid status`
  ].join("\n");

  return Response.json({ markdown });
}
 apps/web/src/app/api/stripe/checkout/route.js 
import { stripe } from "../../../lib/stripe/server.js";
import { env } from "../../../lib/env.js";

export async function POST(req) {
  const form = await req.formData();
  const priceUsd = Number(form.get("priceUsd") ?? 1999);

  if (!env.stripe.secretKey) {
    return new Response("Stripe is not configured (missing STRIPE_SECRET_KEY).", { status: 500 });
  }

  const session = await stripe.checkout.sessions.create({
    mode: "payment",
    line_items: [
      {
        price_data: {
          currency: "usd",
          unit_amount: priceUsd,
          product_data: { name: "Startup Factory Pro" }
        },
        quantity: 1
      }
    ],
    success_url: `${env.appUrl}/success?session_id={CHECKOUT_SESSION_ID}`,
    cancel_url: `${env.appUrl}/pricing`
  });

  return Response.redirect(session.url, 303);
}

 bot/  (Probot GitHub App)
 bot/package.json 
{
  "name": "@startup-factory/bot",
  "version": "0.1.0",
  "private": true,
  "type": "module",
  "main": "src/app.js",
  "scripts": {
    "start": "probot run ./src/app.js",
    "dev": "NODE_ENV=development probot run ./src/app.js"
  },
  "dependencies": {
    "probot": "^13.4.0"
  },
  "engines": {
    "node": ">=18"
  }
}
 bot/src/app.js 
import { getCommandHandler } from "./commands/index.js";

export default (app) => {
  app.log.info("startup-factory bot loaded");

  app.on("issue_comment.created", async (context) => {
    const body = context.payload.comment?.body || "";
    if (!body.trim().startsWith("/")) return;

    const commandName = body.trim().split(/\s+/)[0].slice(1);
    const handler = getCommandHandler(commandName);
    if (!handler) return;

    try {
      await handler({ app, context, body });
    } catch (err) {
      app.log.error(err, "Command failed");
      await context.octokit.issues.createComment(
        context.issue({
          body: `⚠️ Command failed: \`${String(err?.message || err)}\``
        })
      );
    }
  });
};
 bot/src/commands/index.js 
import { startupCommand } from "./startup.js";

const registry = new Map([
  ["startup", startupCommand]
]);

export function getCommandHandler(name) {
  return registry.get(name);
}
 bot/src/commands/startup.js 
import { runStartupFactory } from "../agents/orchestrator.js";

function parseArgs(body) {
  const tokens = body.replace(/^\/\w+\s*/, "").trim();
  const args = {};
  const re = /(\w+)\s*:\s*(?:"(*)"|'(*)'|(+))/g;

  let m;
  while ((m = re.exec(tokens))) {
    const [, key, dq, sq, bare] = m;
    args[key] = dq ?? sq ?? bare ?? "";
  }
  return args;
}

export async function startupCommand({ context, body }) {
  const args = parseArgs(body);

  const idea = args.idea || args.prompt || args.p || "Missing idea: add idea:\"...\"";
  const stack = args.stack || "nextjs+supabase+stripe";
  const targetUser = args.user || "general";

  await context.octokit.issues.createComment(
    context.issue({
      body: [
        "**Startup Factory running…**",
        "",
        `- Idea: ${idea}`,
        `- Stack: ${stack}`,
        `- User: ${targetUser}`,
        "",
        "_Generating a business plan, MVP plan, brand kit tokens, and deployment notes…_"
      ].join("\n")
    })
  );

  const result = await runStartupFactory({ idea, stack, targetUser });

  await context.octokit.issues.createComment(
    context.issue({
      body: result.markdown
    })
  );
}
 bot/src/agents/orchestrator.js 
import { runAnalyst } from "./analyst.js";
import { runArchitect } from "./architect.js";
import { runCoder } from "./coder.js";
import { runDesigner } from "./designer.js";
import { runPackager } from "./packager.js";

export async function runStartupFactory({ idea, stack, targetUser }) {
  const analysis = await runAnalyst({ idea, targetUser });
  const architecture = await runArchitect({ idea, stack, analysis });
  const codePlan = await runCoder({ idea, stack, analysis, architecture });
  const brand = await runDesigner({ idea, analysis, targetUser });
  const packagePlan = await runPackager({ idea, stack, codePlan });

  const markdown = [
    `## Startup Factory Output`,
    ``,
    `### 1) Market analysis`,
    analysis.markdown,
    ``,
    `### 2) System architecture`,
    architecture.markdown,
    ``,
    `### 3) MVP build plan`,
    codePlan.markdown,
    ``,
    `### 4) Brand kit`,
    brand.markdown,
    ``,
    `### 5) Packaging & deployment`,
    packagePlan.markdown
  ].join("\n");

  return { markdown, analysis, architecture, codePlan, brand, packagePlan };
}
 bot/src/agents/analyst.js 
export async function runAnalyst({ idea, targetUser }) {
  const markdown = [
    `**Idea:** ${idea}`,
    ``,
    `**Target user:** ${targetUser}`,
    ``,
    `**Problem hypothesis**`,
    `- People struggle with: (fill in)`,
    `- Current alternatives: (fill in)`,
    ``,
    `**MVP wedge**`,
    `- Narrow first use-case: (fill in)`,
    `- Why now: (fill in)`,
    ``,
    `**Risks**`,
    `- Distribution risk`,
    `- Differentiation risk`,
    `- Retention risk`
  ].join("\n");

  return { markdown, keyRisks: ["distribution", "differentiation", "retention"] };
}
 bot/src/agents/architect.js 
export async function runArchitect({ stack }) {
  const markdown = [
    `**Stack preference:** ${stack}`,
    ``,
    `**Architecture (proposed)**`,
    `- Frontend: Next.js`,
    `- Backend/API: Next.js route handlers`,
    `- Database/Auth: Supabase`,
    `- Payments: Stripe (automated), Cash App/PayPal (links)`,
    ``,
    `**Core entities**`,
    `- User`,
    `- Idea`,
    `- Artifact (plan, brand kit, etc.)`,
    ``,
    `**Key flows**`,
    `1. User enters idea`,
    `2. Generate artifacts`,
    `3. Optional upgrade via Stripe`,
    `4. Export artifacts`
  ].join("\n");

  return { markdown };
}
 bot/src/agents/coder.js 
export async function runCoder({ idea, stack }) {
  const markdown = [
    `**Build plan for:** ${idea}`,
    ``,
    `**Stack:** ${stack}`,
    ``,
    `**MVP features**`,
    `- Idea capture`,
    `- Generation endpoint`,
    `- Pricing page with Stripe checkout`,
    `- Success page`,
    ``,
    `**Milestones**`,
    `1. Supabase table + RLS`,
    `2. Stripe checkout`,
    `3. Stripe webhook -> paid flag`,
    `4. Premium gating`
  ].join("\n");

  return { markdown };
}
 bot/src/agents/designer.js 
export async function runDesigner({ idea }) {
  const tokens = {
    name: "Startup Factory Brand",
    concept: idea,
    colors: {
      primary: "#3B82F6",
      secondary: "#111827",
      accent: "#F59E0B",
      background: "#FFFFFF"
    },
    typography: { heading: "Inter", body: "Inter" }
  };

  const markdown = [
    `**Brand direction**`,
    `- Tone: modern, practical, builder-focused`,
    `- Visual: clean, high-contrast, minimal`,
    ``,
    `**Design tokens (JSON)**`,
    "```json",
    JSON.stringify(tokens, null, 2),
    "```"
  ].join("\n");

  return { markdown, tokens };
}
 bot/src/agents/packager.js 
export async function runPackager({ stack }) {
  const markdown = [
    `**Packaging**`,
    `- App: Next.js web`,
    `- Bot: Probot GitHub App`,
    ``,
    `**Deployment notes**`,
    `- Web: deploy to Vercel (typical) or any Node host`,
    `- Bot: deploy to Render/Fly/EC2/etc.`,
    `- Configure env vars for Supabase + Stripe`,
    ``,
    `**Stack:** ${stack}`
  ].join("\n");

  return { markdown };
}

 packages/config  (optional placeholder)
 packages/config/package.json 
{
  "name": "@startup-factory/config",
  "private": true,
  "version": "0.0.1",
  "type": "module"
}
 packages/config/README.md 
Shared config package (placeholder).

You can move shared constants, ESLint, TS configs, etc. here later.

Supabase SQL (run manually)
create table if not exists public.ideas (
  id bigint generated by default as identity primary key,
  created_at timestamptz not null default now(),
  idea text not null,
  stack text not null
);

alter table public.ideas enable row level security;

create policy "Allow anon insert for MVP"
on public.ideas
for insert
to anon
with check (true);

Quick start
# repo root
npm install

# web app
cp apps/web/.env.example apps/web/.env.local
# fill in Supabase + Stripe keys
npm run dev:web
# http://localhost:3000

# bot (optional)
npm run dev:bot
That’s the full copy/paste bundle.
