# n8n Workflow Optimizer

## Purpose
Optimize and create n8n workflows that are app-ready with proper data intake/output structures.

## Workflow Process

### 1. Analyze
- Review existing workflow structure
- Identify input/output requirements
- Determine pattern: Webhook, API Integration, Database, AI Agent, or Scheduled

### 2. Optimize
Ensure workflows have:
- **Clear Entry Point**: Webhook trigger or appropriate intake node
- **Structured Input**: Defined expected data schema
- **Error Handling**: Graceful failures with meaningful responses
- **Structured Output**: Consistent response format

### 3. Output
- Export optimized workflow as JSON
- Name based on workflow name: `{workflow-name}.json`

## File Structure
```
/workflows/     # Exported workflow JSON files
```

---

## n8n Technical Reference

### 5 Core Patterns
1. **Webhook Processing** - Receive HTTP → Process → Respond
2. **HTTP API Integration** - Fetch API → Transform → Store
3. **Database Operations** - Read/Write/Sync database data
4. **AI Agent Workflow** - AI agents with tools and memory
5. **Scheduled Tasks** - Recurring automation

### Expression Syntax Rules
- All dynamic content uses `{{expression}}`
- `$json` - Current node output
- `$node["Node Name"].json` - Reference other nodes (case-sensitive)
- `$now` - Timestamp with `.toFormat('yyyy-MM-dd')`
- `$env.VAR_NAME` - Environment variables

**Critical**: Webhook data is nested under `.body`:
```
✅ {{$json.body.fieldName}}
❌ {{$json.fieldName}}
```

### Code Node (JavaScript)
- Must return `[{json: {...}}]` array format
- Use "Run Once for All Items" for 95% of cases
- No `{{}}` syntax - use direct JS: `$json.fieldName`
- Access all items: `$input.all()`
- Access single: `$input.first()`

### Validation Profiles
| Profile | Use Case |
|---------|----------|
| minimal | Quick edits |
| runtime | Pre-deployment (recommended) |
| ai-friendly | AI-generated configs |
| strict | Production/critical |

### Common Errors
- `missing_required` - Essential fields absent
- `invalid_value` - Value outside permitted options
- `type_mismatch` - Wrong data type
- `invalid_reference` - Node doesn't exist
- `invalid_expression` - Syntax error

---

## Decision Framework

**Use Deterministic** when:
- Rule-based logic with clear conditions
- Outputs must be predictable/reproducible
- Compliance requirements exist

**Use Non-Deterministic (AI Agent)** when:
- Input requires interpretation
- Natural language processing needed
- Flexibility in response acceptable
