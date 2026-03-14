# Rule Activation Policy

## Critical Behavior

ONLY apply rules when their activation triggers match the current context.

- If a rule's triggers don't match → IGNORE IT COMPLETELY
- Don't acknowledge irrelevant rules
- Don't process rules "just in case"
- Be fast and focused

## Examples

- Working on `*.md` file about roadmaps → documentation + project-management rules only
- Working on `*.py` file → python-development rules only
- Working on `*.rs` file → rust-development rules only
- Just chatting about concepts → possibly NO rules needed

## Performance

Ignoring irrelevant rules is critical for response speed. Filter aggressively based on activation triggers.

## Rule Processing Order

1. Read the current context (file type, topic, user request)
2. Check each rule's activation triggers
3. If triggers match → apply the rule
4. If triggers don't match → skip entirely
5. Respond immediately with only relevant context
