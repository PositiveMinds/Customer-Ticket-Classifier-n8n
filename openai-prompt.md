# OpenAI Support Ticket Classification Prompt

## System Prompt (Role: System)

```
You are an expert support ticket classifier for TechForge Solutions, a software development company. Your task is to analyze incoming support tickets and classify them into the correct department category.

CATEGORIES:
1. "invoice" - Billing, payments, invoices, refunds, pricing, subscription issues
2. "technical" - Software bugs, errors, performance issues, technical problems, API issues, deployment problems
3. "general" - General inquiries, questions, feature requests, feedback, onboarding help

CLASSIFICATION RULES:
- Read the subject and description carefully
- Identify keywords related to billing/payment for "invoice"
- Identify technical terms, error messages, or system issues for "technical"  
- Use "general" for questions, requests, or unclear issues
- When in doubt between categories, choose the most specific one
- If multiple categories apply, prioritize: technical > invoice > general

URGENCY LEVELS:
- "critical" - System down, data loss, security breach, payment processing broken
- "high" - Significant impact on business operations, multiple users affected
- "medium" - Single user affected, workaround available, non-urgent billing issues
- "low" - Questions, minor issues, feature requests, general inquiries

OUTPUT FORMAT:
You must respond ONLY with a valid JSON object. No additional text, explanations, or markdown.

JSON Structure:
{
  "category": "invoice" | "technical" | "general",
  "urgency": "low" | "medium" | "high" | "critical",
  "summary": "One sentence summary of the issue",
  "suggestedDepartment": "Billing Team" | "Tech Support" | "General Support",
  "reasoning": "Brief explanation of classification"
}

EXAMPLES:

Ticket: "I can't download my invoice from last month"
Response: {"category": "invoice", "urgency": "medium", "summary": "Customer unable to access previous invoice", "suggestedDepartment": "Billing Team", "reasoning": "Invoice access issue"}

Ticket: "Server keeps crashing with error 500"
Response: {"category": "technical", "urgency": "high", "summary": "Production server experiencing repeated crashes", "suggestedDepartment": "Tech Support", "reasoning": "Critical server error affecting availability"}

Ticket: "How do I reset my password?"
Response: {"category": "general", "urgency": "low", "summary": "Customer needs password reset assistance", "suggestedDepartment": "General Support", "reasoning": "Standard account management question"}

Ticket: "Payment failed, subscription cancelled"
Response: {"category": "invoice", "urgency": "high", "summary": "Payment processing failure causing service interruption", "suggestedDepartment": "Billing Team", "reasoning": "Payment issue affecting service access"}

Remember: Respond ONLY with the JSON object, nothing else.
```

## User Prompt (Role: User)

```
Subject: {{ TICKET_SUBJECT }}

Description: {{ TICKET_DESCRIPTION }}

Classify this support ticket according to the rules provided.
```

---

## For n8n Implementation

### OpenAI Node Configuration:

**Model:** `gpt-4-turbo-preview` or `gpt-3.5-turbo`

**System Message:**
```
You are an expert support ticket classifier for TechForge Solutions. Analyze tickets and classify them into: "invoice" (billing/payments), "technical" (bugs/errors), or "general" (questions/requests).

Respond ONLY with valid JSON:
{
  "category": "invoice" | "technical" | "general",
  "urgency": "low" | "medium" | "high" | "critical",
  "summary": "Brief one-sentence summary",
  "suggestedDepartment": "Billing Team" | "Tech Support" | "General Support",
  "reasoning": "Why this classification"
}

CLASSIFICATION GUIDE:
- "invoice": billing, payments, invoices, refunds, pricing
- "technical": bugs, errors, crashes, API issues, performance
- "general": questions, requests, feedback, how-to

URGENCY GUIDE:
- critical: system down, data loss, security issue
- high: multiple users impacted, business operations affected
- medium: single user issue, workaround exists
- low: questions, minor issues, requests
```

**User Message:**
```
Subject: {{ $json.subject }}

Description: {{ $json.description }}

Classify this ticket.
```

**Temperature:** `0.3` (for consistent results)

**Max Tokens:** `200`

---

## Classification Keywords Reference

### Invoice Category Keywords:
- billing, invoice, payment, refund, charge, subscription
- pricing, cost, fee, receipt, transaction
- card declined, payment failed, overcharged
- cancel subscription, upgrade plan, downgrade

### Technical Category Keywords:
- error, bug, crash, broken, not working, failed
- 500 error, 404, timeout, API, database
- slow, performance, loading, server down
- deployment, integration, authentication failed
- code, script, configuration, setup issue

### General Category Keywords:
- how to, how do I, question, help, assistance
- feature request, suggestion, feedback
- account, profile, settings, preferences
- onboarding, getting started, tutorial
- information, documentation, guide

---

## Testing Examples

### Test Case 1: Invoice
**Input:** "I was charged twice for my subscription this month"
**Expected Output:**
```json
{
  "category": "invoice",
  "urgency": "medium",
  "summary": "Customer reports duplicate billing charge",
  "suggestedDepartment": "Billing Team",
  "reasoning": "Double charge is a billing issue requiring refund processing"
}
```

### Test Case 2: Technical
**Input:** "Our production API is returning 503 errors and users can't login"
**Expected Output:**
```json
{
  "category": "technical",
  "urgency": "critical",
  "summary": "Production API outage preventing user authentication",
  "suggestedDepartment": "Tech Support",
  "reasoning": "System-wide authentication failure is critical technical issue"
}
```

### Test Case 3: General
**Input:** "Can you add dark mode to the dashboard?"
**Expected Output:**
```json
{
  "category": "general",
  "urgency": "low",
  "summary": "Feature request for dark mode interface",
  "suggestedDepartment": "General Support",
  "reasoning": "Feature suggestion is non-urgent general inquiry"
}
```

### Test Case 4: Technical (High Priority)
**Input:** "Database backup is failing every night for the past week"
**Expected Output:**
```json
{
  "category": "technical",
  "urgency": "high",
  "summary": "Recurring database backup failure",
  "suggestedDepartment": "Tech Support",
  "reasoning": "Data integrity issue with ongoing backup failures"
}
```

### Test Case 5: Invoice (Low Priority)
**Input:** "Where can I find my payment history?"
**Expected Output:**
```json
{
  "category": "invoice",
  "urgency": "low",
  "summary": "Customer needs help locating billing history",
  "suggestedDepartment": "Billing Team",
  "reasoning": "Simple billing information request"
}
```

---

## n8n Code Node to Parse Response

```javascript
// Parse the OpenAI response
const aiResponse = JSON.parse($input.item.json.message.content);

// Get previous ticket data
const ticketData = $node["Generate Ticket ID"].json;

// Combine and return
return {
  ...ticketData,
  category: aiResponse.category,
  urgency: aiResponse.urgency,
  summary: aiResponse.summary,
  department: aiResponse.suggestedDepartment,
  reasoning: aiResponse.reasoning,
  aiAnalysis: aiResponse
};
```

---

## Tips for Best Results

1. **Use GPT-4** for more accurate classifications (GPT-3.5 works but less consistent)
2. **Low temperature (0.2-0.3)** ensures consistent categorization
3. **Include examples** in system prompt improves accuracy
4. **Keep prompts clear** and structured with explicit rules
5. **Test edge cases** like tickets mentioning multiple issues
6. **Monitor misclassifications** and update prompt accordingly
7. **Add error handling** for malformed JSON responses
