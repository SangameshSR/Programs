You are an expert Java/Spring + PostgreSQL + banking-domain debugger. Diagnose and fix my existing project WITHOUT destroying or refactoring anything.

ISSUE:
POST /fbp-des-channels/v2/bill-payments returns HTTP 500.

Response:
{
  "errorResponse": {
    "category": "Limit Validation Error",
    "correlationId": "d16aced9-4bc0-4c6d-b6e4-ab790f2f100d",
    "errors": [
      {
        "code": "LIMITS203301",
        "codeDesc": "limits.business.error.limit.technical_error",
        "message": "Technical error occurred while processing the limit operation."
      }
    ]
  }
}

Because Limit Validation fails, workflow is not reached and I get isWorkflowRequired = "N".

IMPORTANT EVIDENCE:
The server/debug log for the SAME correlation ID:
d16aced9-4bc0-4c6d-b6e4-ab790f2f100d

shows:
OpLimitsServiceImpl.evaluateLimit

and PostgreSQL stack trace:
QueryExecutorImpl.processResults
QueryExecutorImpl.execute

with:
Position: 239

The failing SQL starts approximately:
select oled1_0.id, oled1_0.aggregate_capping_behaviour, oled1_0.chargeable_...

We found this code:

try {
    RequestContext requestContext = getRequestContext(limitModel);
    EvaluateLimitRequestDTO evaluateLimitRequestDTO =
        getEvaluateLimitRequest(limitModel);

    evaluateLimitRequestDTO.setTrialMode("true");

    return opLimitsService.evaluateLimit(
        requestContext,
        evaluateLimitRequestDTO
    );

} catch (Exception e) {
    log.error("Error occurred while validating limits {}", e.getMessage());
    return null;
}

The actual failure is inside:
opLimitsService.evaluateLimit(...)

We also found populateEvaluateLimitsResponse(...), but DO NOT assume that method is the root cause. It appears to process already-fetched limit data.

REQUEST DATA:
financialActivityType = BILL
account = 123456789
registrationIdentification = 246888
consumerReference = MOBILE_NUMBER / 9876543210
invoiceNumber = ARG1234
paymentAmount = 2000 INR
frequencyType = ONE_TIME
paymentDate = 2026-12-10

YOUR TASK:

1. Trace the complete call chain:
   Limit validation
   -> opLimitsService.evaluateLimit()
   -> implementation
   -> repository/DAO
   -> PostgreSQL query.

2. Find the EXACT database query that fails.

3. Use PostgreSQL "Position: 239" to identify the exact SQL token/column/table/expression causing the failure.

4. Find the COMPLETE PostgreSQL exception. Do not stop at LIMITS203301 because that is only the wrapped business error.

5. Identify the actual root cause:
   - DB column mismatch
   - missing table/view
   - wrong Hibernate/JPA mapping
   - missing migration
   - wrong schema
   - invalid SQL
   - bad data
   - configuration
   - connection issue
   - or another proven cause.

6. Do NOT guess. If the provided code is insufficient, tell me exactly which file/method/log/query you need.

7. After proving the root cause, make ONLY the smallest required fix.

STRICT RULES:
- Do NOT refactor unrelated code.
- Do NOT rewrite classes.
- Do NOT change API contracts.
- Do NOT change DTOs unless proven necessary.
- Do NOT change workflow configuration.
- Do NOT change isWorkflowRequired just to hide the error.
- Do NOT disable limit validation.
- Do NOT hardcode values to make the test pass.
- Do NOT return fake success responses.
- Do NOT remove business validations.
- Do NOT change database schema unless a missing migration is proven to be the cause.
- Preserve existing business logic.
- Preserve security and transaction behavior.
- Do not modify unrelated files.
- Do not make speculative changes.

Before giving any code change, provide:

1. ROOT CAUSE
2. EXACT FILE
3. EXACT METHOD
4. EXACT LINE/QUERY
5. EVIDENCE FROM LOG/CODE
6. WHY IT FAILS
7. MINIMAL FIX

Then provide:
BEFORE CODE
AFTER CODE
WHY THE CHANGE IS SAFE
HOW TO TEST
ROLLBACK PLAN

Also check whether this catch block is hiding the real exception:

catch (Exception e) {
    log.error("Error occurred while validating limits {}", e.getMessage());
    return null;
}

Do NOT remove it automatically. First determine whether returning null is intentional. If useful, recommend only a separate minimal logging improvement to preserve the full stack trace.

FINAL GOAL:
Fix the actual LIMITS203301/PostgreSQL failure so that Limit Validation succeeds, then the existing workflow flow can continue naturally.

Do not change workflow code unless the investigation proves workflow itself is the cause.
