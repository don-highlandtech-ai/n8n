# n8n Workflow Optimizer

## Purpose
Optimize and create n8n workflows that are app-ready with proper data intake/output structures.

## Tools
- **n8n MCP**: View/modify workflows, understand nodes, configurations, and templates
- **n8n Skills**: Workflow development assistance

## Workflow Process

### 1. Analyze
- Review existing workflow structure
- Identify input/output requirements
- Determine if workflow needs deterministic (rule-based) or non-deterministic (AI agent) approach

### 2. Optimize
Ensure workflows have:
- **Clear Entry Point**: Webhook trigger or appropriate intake node
- **Structured Input**: Defined expected data schema
- **Error Handling**: Graceful failures with meaningful responses
- **Structured Output**: Consistent response format for downstream consumption

### 3. Output
- Export optimized workflow as JSON
- Name based on workflow name: `{workflow-name}.json`

## App-Ready Checklist
- [ ] Entry node accepts expected input format
- [ ] Response node returns structured data
- [ ] Error states return useful messages
- [ ] Workflow is testable standalone

## File Structure
```
/workflows/     # Exported workflow JSON files
/docs/          # Workflow documentation (if needed)
```

## Decision Framework: Deterministic vs Non-Deterministic

**Use Deterministic** when:
- Logic is rule-based with clear conditions
- Outputs must be predictable/reproducible
- Compliance or audit requirements exist

**Use Non-Deterministic (AI Agent)** when:
- Input requires interpretation or classification
- Tasks involve natural language processing
- Flexibility in response is acceptable
