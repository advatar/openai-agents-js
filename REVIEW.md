# Code Review: openai-agents-js

Review date: 2026-05-11
Tracker: https://github.com/advatar/Tracker/issues/53
Scope: top-level app folder `openai-agents-js` and nested project manifests under this folder, excluding generated dependency/build directories such as `.git`, `node_modules`, `target`, `.build`, `dist`, and virtual environments.

## Executive Summary

- Overall risk from this sweep: **High**
- Findings by severity: High 1, Medium 3, Low 0
- Source footprint: 616 source files by extension scan (TypeScript 597, JavaScript 13, CSS 4, HTML 2)
- Test footprint: 125 test-like files detected
- CI footprint: 7 GitHub Actions workflow files detected
- Git posture: 2 changed/untracked paths before review generation
- Pattern scan budget used: 765 text/source files scanned

## Architecture Snapshot

Detected project and build surfaces:
- `docs/package-lock.json`
- `docs/package.json`
- `examples/agent-patterns/package.json`
- `examples/ai-sdk-ui/package.json`
- `examples/ai-sdk-v1/package.json`
- `examples/ai-sdk/package.json`
- `examples/basic/package.json`
- `examples/connectors/package.json`
- `examples/customer-service/package.json`
- `examples/docs/package.json`
- `examples/financial-research-agent/package.json`
- `examples/handoffs/package.json`
- `examples/mcp/package.json`
- `examples/memory/package.json`
- `examples/model-providers/package.json`
- `examples/nextjs/package.json`
- `examples/realtime-demo/package.json`
- `examples/realtime-next/package.json`
- `examples/realtime-twilio-sip/package.json`
- `examples/realtime-twilio/package.json`

Nested manifest owners sampled:
- `.`
- `docs`
- `examples/agent-patterns`
- `examples/ai-sdk`
- `examples/ai-sdk-ui`
- `examples/ai-sdk-v1`
- `examples/basic`
- `examples/connectors`
- `examples/customer-service`
- `examples/docs`
- `examples/financial-research-agent`
- `examples/handoffs`
- `examples/mcp`
- `examples/memory`
- `examples/model-providers`
- `examples/nextjs`
- `examples/realtime-demo`
- `examples/realtime-next`
- `examples/realtime-twilio`
- `examples/realtime-twilio-sip`

Package scripts sampled:
- ``docs/package.json`: build`
- ``examples/agent-patterns/package.json`: build-check`
- ``examples/ai-sdk-ui/package.json`: build, build-check, lint`
- ``examples/ai-sdk-v1/package.json`: build-check`
- ``examples/ai-sdk/package.json`: build-check`
- ``examples/basic/package.json`: build-check`
- ``examples/connectors/package.json`: build-check`
- ``examples/customer-service/package.json`: build-check`

Local instruction/status files:
- `AGENTS.md`

## Findings

### 1. [High] Dynamic code or shell execution needs input-boundary review

These APIs are legitimate in tooling, but they become high-risk when command strings or evaluated input can be influenced by users, files, networks, or model output. Scanner count: 3.

Evidence:
- integration-tests/cloudflare-workers/worker/worker-configuration.d.ts:2368 `exec(input?: string | URLPatternInit, baseURL?: string): URLPatternResult | null;`
- integration-tests/cloudflare-workers/worker/worker-configuration.d.ts:5127 `exec(query: string): Promise<D1ExecResult>;`
- scripts/changeset-validation-lite.mjs:26 `return execSync(cmd, {`
### 2. [Medium] Potential credential/config material needs a focused secret audit

Names commonly used for credentials or sensitive tokens appear in app-owned files. Some hits may be fixtures or placeholders, but every example should be verified, documented as fake, or moved to secret management. Values are redacted here. Scanner count: 1509.

Evidence:
- docs/src/scripts/translate.ts:144 `setDefaultOpenAIKey(process.env.OPENAI_API_KEY || '');`
- docs/src/scripts/translate.ts:275 `"* The terms 'temperature', 'top_p', 'max_tokens', 'presence_penalty', 'frequency_penalty' as parameter names must be kept as is.",`
- docs/src/scripts/translate.ts:292 `"* 'instructions', 'tools'와 같은 API 매개변수 이름과 temperature, top_p, max_tokens, presence_penalty, frequency_penalty 등은 영문 그대로 유지하세요.",`
- examples/ai-sdk-v1/ai-sdk-v1.ts:433 `maxTokens: request.modelSettings.maxTokens,`
- examples/ai-sdk-v1/ai-sdk-v1.ts:485 `inputTokens: Number.isNaN(result.usage?.promptTokens)`
- examples/ai-sdk-v1/ai-sdk-v1.ts:487 `: (result.usage?.promptTokens ?? 0),`
- examples/ai-sdk-v1/ai-sdk-v1.ts:488 `outputTokens: Number.isNaN(result.usage?.completionTokens)`
- examples/ai-sdk-v1/ai-sdk-v1.ts:490 `: (result.usage?.completionTokens ?? 0),`
### 3. [Medium] Many nested project manifests increase ownership and verification complexity

This app folder contains many buildable surfaces. Document ownership and canonical verification commands so fixes do not verify the wrong package.

Evidence:
- docs/package-lock.json
- docs/package.json
- examples/agent-patterns/package.json
- examples/ai-sdk-ui/package.json
- examples/ai-sdk-v1/package.json
- examples/ai-sdk/package.json
- examples/basic/package.json
- examples/connectors/package.json
### 4. [Medium] Multiple JavaScript package-manager lockfiles are present

Mixed package managers make installs non-reproducible and can cause CI/local dependency drift. Pick one package manager per app boundary or document nested ownership.

Evidence:
- docs/package-lock.json
- pnpm-lock.yaml

## Testing and Build Posture

Detected tests:
- `helpers/tests/console-guard.ts`
- `helpers/tests/setup.ts`
- `integration-tests/bun.test.ts`
- `integration-tests/cloudflare.test.ts`
- `integration-tests/deno.test.ts`
- `integration-tests/node-ai-sdk-v2-ts.test.ts`
- `integration-tests/node-ai-sdk-v3-ts.test.ts`
- `integration-tests/node-zod3.test.ts`
- `integration-tests/node-zod4-ts.test.ts`
- `integration-tests/node-zod4.test.ts`
- `integration-tests/node.test.ts`
- `integration-tests/vite-react.test.ts`

Detected CI workflows:
- `.github/workflows/changeset.yml`
- `.github/workflows/docs.yml`
- `.github/workflows/issues.yml`
- `.github/workflows/release-tag.yml`
- `.github/workflows/release.yml`
- `.github/workflows/test.yml`
- `.github/workflows/update-docs.yml`

Inferred verification commands to standardize:
- JavaScript: run the owning package-manager install/build/test scripts from the relevant `package.json`.

## Review Limitations

- This was a broad static review across many local apps, not a full manual product walkthrough.
- Generated directories and dependency trees were pruned so findings focus on app-owned source.
- Secret-like values were not reproduced; examples are redacted or limited to path/line evidence.
- Pattern scanning is capped per app to keep the cross-repository sweep tractable; high-risk folders need focused follow-up review.

## Recommended Next Steps

1. Resolve every High finding first, especially secret material, tracked generated output, and dynamic execution paths.
2. Add or tighten the app's canonical CI workflow so build and tests run on every push.
3. Convert inferred build/test commands into documented commands in the app README or STATUS file.
4. Add smoke tests around app launch, persistence, API boundaries, and security-sensitive adapters.
5. Re-run this review after cleanup and replace this file with a human-reviewed release checklist.
