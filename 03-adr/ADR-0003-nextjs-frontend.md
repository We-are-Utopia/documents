# ADR-0003: Next.js for Frontend

| Field         | Value                                |
|---------------|--------------------------------------|
| **Version**   | 1.0.0                                |
| **Status**    | Accepted                             |
| **Author**    | Vox                                  |
| **Reviewer**  | Vox                                  |
| **Created**   | 2026-03-27                           |
| **Updated**   | 2026-03-27                           |

---

## Context

Utopia needs a frontend framework that produces a production-optimized, containerized web application. The frontend is secondary to the DevOps focus, so it must be quick to scaffold with a mature component ecosystem while still being production-grade. It must integrate cleanly with Keycloak for authentication and produce Docker images suitable for Kubernetes deployment.

## Decision Drivers

- **Docker optimization** — standalone build output for minimal container images
- **Production readiness** — SSR/SSG/ISR support, built-in performance optimizations
- **DevOps ecosystem** — extensive CI/CD examples, Docker best practices available
- **Component ecosystem** — mature UI libraries for rapid development
- **TypeScript support** — first-class TypeScript with strict mode
- **SEO capability** — server-side rendering for future public-facing pages
- **Developer experience** — fast hot reload, clear conventions

## Considered Options

1. **Next.js 14+ (App Router)** — React meta-framework by Vercel
2. **Angular 17+** — Google's full-featured framework
3. **Nuxt 3 (Vue)** — Vue meta-framework
4. **SvelteKit** — Svelte meta-framework

## Decision

We will use **Next.js 14+ with App Router** and **TypeScript strict mode** for the frontend.

UI components will use **shadcn/ui** (built on Radix UI) with **Tailwind CSS 4** for styling. State management will use **Zustand** (client state) and **TanStack Query v5** (server state).

## Comparison

| Criterion | Next.js | Angular | Nuxt 3 | SvelteKit |
|-----------|---------|---------|--------|-----------|
| Docker standalone output | ✅ `output: 'standalone'` | ⚠️ Static build only | ⚠️ Nitro server | ✅ Adapter-node |
| Container image size | ✅ ~50 MB (standalone) | ✅ ~20 MB (static) | ⚠️ ~100 MB | ✅ ~40 MB |
| SSR / SSG / ISR | ✅ All three | ⚠️ SSR (Universal) | ✅ All three | ✅ SSR + SSG |
| Component ecosystem | ✅ Largest (React) | ✅ Large (Angular Material) | ⚠️ Medium | ❌ Small |
| TypeScript | ✅ First-class | ✅ Native | ✅ First-class | ✅ First-class |
| DevOps CI/CD examples | ✅ Most documented | ✅ Well documented | ⚠️ Moderate | ❌ Limited |
| Learning curve | ⚠️ Medium | ❌ Steep | ✅ Easy | ✅ Easy |
| Community size | ✅ Largest | ✅ Large | ⚠️ Medium | ⚠️ Growing |
| Keycloak integration | ✅ next-auth / Auth.js | ✅ angular-oauth2-oidc | ✅ nuxt-auth | ⚠️ Community libs |
| shadcn/ui support | ✅ Native | ❌ No | ✅ Port available | ❌ No |

## Consequences

### Positive

- `output: 'standalone'` produces a self-contained Node.js server, ideal for Docker (~50 MB image)
- Largest React ecosystem — shadcn/ui, TanStack Query, Zustand all first-class
- Excellent DevOps documentation and Docker examples from Vercel and community
- App Router with React Server Components reduces client-side JavaScript
- Auth.js (NextAuth) provides battle-tested Keycloak/OIDC integration
- Built-in image optimization, code splitting, and performance defaults

### Negative

- React Server Components (RSC) add complexity to mental model — mitigation: clear documentation on when to use `'use client'`
- Vercel-specific features (Edge Runtime, ISR) may not work in self-hosted K8s — mitigation: use only standard Node.js runtime features
- Rapid release cadence may introduce breaking changes — mitigation: pin versions, upgrade deliberately

### Risks

- Next.js App Router API stability — likelihood: Low (stable since v14), impact: Medium — mitigation: follow stable APIs only
- Bundle size growth with many dependencies — likelihood: Medium, impact: Low — mitigation: monitor with `@next/bundle-analyzer`

## References

- [Next.js Documentation](https://nextjs.org/docs)
- [Next.js Docker Example](https://github.com/vercel/next.js/tree/canary/examples/with-docker)
- [shadcn/ui](https://ui.shadcn.com/)
- [Auth.js](https://authjs.dev/)
- [CODING-STANDARD.md](../00-standards/CODING-STANDARD.md) — Frontend conventions

## Changelog

| Version | Date       | Author | Description          |
|---------|------------|--------|----------------------|
| 1.0.0   | 2026-03-27 | Vox    | Initial decision     |
