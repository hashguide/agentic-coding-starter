---
kind: playbook
domain: agent-development
topics: [agent, development, handling, hallucination]
authority: guidance
---

# Handling Hallucination

AI coding agents occasionally invent things: a package that doesn't exist, a method that doesn't exist on a real package, a CLI flag that was never implemented, a configuration option that the documentation doesn't mention. These inventions are called hallucinations. They are not random errors — they follow predictable patterns, and they are catchable before they ship.

## How Hallucinations Occur

Models generate the most plausible-sounding continuation of a prompt, based on training data. For well-documented, frequently-used packages (React, Express, Stripe's core API), the training signal is strong and hallucinations are rare. For less common packages, older documentation, niche configuration options, and recently-changed APIs, the signal is weaker and plausible-sounding fabrications become more likely.

## Verifying a Package Actually Exists

Before installing a package the agent suggested:

```bash
npm info <package-name>
# Should return package metadata
# If it returns "npm error 404 Not Found" — the package doesn't exist
```

For Python:
```bash
pip index versions <package-name>
# or check pypi.org/<package-name>
```

Never blindly run `npm install <agent-suggested-package>` without verifying the package exists and is the package you think it is (not a typosquat).

## Verifying an API Method Is Real

If an agent uses a method on a library you don't recognize:

1. Check the package's TypeScript type definitions:
   ```bash
   # Navigate to the type definitions
   cat node_modules/<package>/dist/index.d.ts | grep "methodName"
   ```

2. Check the official documentation directly — do not ask the agent to summarize the documentation.

3. Check the package's GitHub repository or changelog for the version you have installed. A method may exist in v3 but not in v2.

## Hallucination-Prone Areas

**Less common library methods:** Core methods are well-covered in training data. Secondary methods, utility functions on a library's sub-packages, and methods added in recent minor versions are more prone to hallucination.

**Config flags and options:** CLI flags, configuration file keys, environment variable names, and plugin options are frequently invented. Always verify against the official docs for the exact version installed.

**Recently-changed APIs:** If a library had a major version change after the model's training cutoff, the agent will describe the old API confidently. Check the current version's migration guide.

**Intersection of two libraries:** "How do I configure X to work with Y?" questions often produce plausible-but-wrong adapter patterns when the two libraries don't have a documented integration path.

**Security-adjacent configuration:** CORS settings, cookie configuration, JWT validation options — these areas are critically important and frequently hallucinated with subtle mistakes. Always verify against official security documentation.

## When the Agent Cites Documentation

If the agent says "according to the docs, you should do X," treat this as a hint to check, not a verified fact. Ask: "What documentation? What version? What URL?" Then go look at that source yourself.

## Related Documents

- [Reviewing AI code](reviewing-ai-code.md)
- [Verification strategies](verification-strategies.md)
- [Prompting for code](prompting-for-code.md)
