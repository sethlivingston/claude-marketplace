---
phase: quick
plan: 01
type: execute
wave: 1
depends_on: []
files_modified:
  - /Users/sethlivingston/dev/sethlivingston/the-typescript-narrows/plugin/the-typescript-narrows/README.md
  - /Users/sethlivingston/dev/sethlivingston/the-typescript-narrows/README.md
autonomous: true
requirements: [update-readmes]
must_haves:
  truths:
    - "Plugin README shows marketplace install as the primary method"
    - "Root README references marketplace installation in Getting Started"
    - "Both READMEs include the exact CLI commands to add marketplace and install plugin"
  artifacts:
    - path: "/Users/sethlivingston/dev/sethlivingston/the-typescript-narrows/plugin/the-typescript-narrows/README.md"
      provides: "Plugin install instructions with marketplace-first approach"
      contains: "claude plugin add-marketplace"
    - path: "/Users/sethlivingston/dev/sethlivingston/the-typescript-narrows/README.md"
      provides: "Root README with marketplace install reference"
      contains: "claude plugin install"
  key_links: []
---

<objective>
Update both README files in the-typescript-narrows repo to include marketplace installation instructions as the primary install method.

Purpose: Users should be able to install the plugin via the marketplace with two simple CLI commands.
Output: Updated README.md files in both root and plugin directories.
</objective>

<execution_context>
@/Users/sethlivingston/.claude/get-shit-done/workflows/execute-plan.md
@/Users/sethlivingston/.claude/get-shit-done/templates/summary.md
</execution_context>

<context>
@/Users/sethlivingston/dev/sethlivingston/the-typescript-narrows/README.md
@/Users/sethlivingston/dev/sethlivingston/the-typescript-narrows/plugin/the-typescript-narrows/README.md
</context>

<tasks>

<task type="auto">
  <name>Task 1: Update plugin README with marketplace install instructions</name>
  <files>/Users/sethlivingston/dev/sethlivingston/the-typescript-narrows/plugin/the-typescript-narrows/README.md</files>
  <action>
Rewrite the Install section of the plugin README. Replace the current two options with a marketplace-first approach. The new Install section should be:

```
## Install

Add the marketplace, then install the plugin:

    claude plugin add-marketplace github:sethlivingston/claude-marketplace
    claude plugin install the-typescript-narrows

The skill triggers automatically when Claude generates TypeScript, reviews TypeScript code, or decides between multiple valid TypeScript patterns.

**Alternative: Reference from GitHub directly**

    github:sethlivingston/the-typescript-narrows/plugin/the-typescript-narrows
```

Remove the old "Copy into your project" option entirely -- marketplace is the canonical method now. Keep the GitHub direct reference as a fallback alternative. Use indented code blocks (4 spaces), not fenced blocks, for the CLI commands so they render cleanly. Keep all other sections of the README unchanged.
  </action>
  <verify>
    <automated>grep -q "claude plugin add-marketplace" /Users/sethlivingston/dev/sethlivingston/the-typescript-narrows/plugin/the-typescript-narrows/README.md && grep -q "claude plugin install the-typescript-narrows" /Users/sethlivingston/dev/sethlivingston/the-typescript-narrows/plugin/the-typescript-narrows/README.md && echo "PASS" || echo "FAIL"</automated>
  </verify>
  <done>Plugin README Install section shows marketplace as primary method with two CLI commands, GitHub direct reference as alternative, and a sentence about auto-triggering behavior.</done>
</task>

<task type="auto">
  <name>Task 2: Update root README Getting Started section</name>
  <files>/Users/sethlivingston/dev/sethlivingston/the-typescript-narrows/README.md</files>
  <action>
Update the "Getting Started" section's AI-assisted development paragraph. Replace the current text:

```
**For AI-assisted development:** Add the Claude plugin to your project so Claude follows these opinions when writing TypeScript. See [plugin README](plugin/the-typescript-narrows/README.md) for installation options.
```

With:

```
**For AI-assisted development:** Install the Claude plugin from the marketplace:

    claude plugin add-marketplace github:sethlivingston/claude-marketplace
    claude plugin install the-typescript-narrows

The skill triggers automatically when writing or reviewing TypeScript. See the [plugin README](plugin/the-typescript-narrows/README.md) for details.
```

Use indented code blocks (4 spaces) for the CLI commands. Leave the ESLint paragraph and all other sections unchanged.
  </action>
  <verify>
    <automated>grep -q "claude plugin add-marketplace" /Users/sethlivingston/dev/sethlivingston/the-typescript-narrows/README.md && grep -q "claude plugin install the-typescript-narrows" /Users/sethlivingston/dev/sethlivingston/the-typescript-narrows/README.md && echo "PASS" || echo "FAIL"</automated>
  </verify>
  <done>Root README Getting Started section includes marketplace CLI commands inline and a note about auto-triggering, with a link to plugin README for details.</done>
</task>

</tasks>

<verification>
Both README files contain the marketplace URL and install commands. The plugin README has marketplace as primary install method. The root README has inline install commands in Getting Started.
</verification>

<success_criteria>
- Plugin README Install section leads with `claude plugin add-marketplace` and `claude plugin install` commands
- Plugin README mentions auto-triggering behavior
- Root README Getting Started includes the same two CLI commands inline
- No other sections of either README are changed
</success_criteria>

<output>
After completion, create `.planning/quick/260319-dmu-update-readmes-in-the-typescript-narrows/260319-dmu-SUMMARY.md`
</output>
