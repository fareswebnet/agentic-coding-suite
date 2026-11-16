# Example Case Study - Fintech CFO B2B

This example shows how the suite can structure a B2B fintech product idea.

## Raw Idea

A CFO copilot for startups that tracks burn, runway, forecast, budget variance, investor metrics, and board-ready reporting.

## Suggested Skill Flow

1. `idea-framing`
2. `market-user-discovery`
3. `idea-framing`
4. `domain-modeling`
5. `bounded-context-map`
6. `bdd-specifications`
7. `ux-product-design`
8. `data-architecture`
9. `security-privacy-compliance`
10. `security-privacy-compliance`
11. `ai-product-architecture`
12. `delivery-roadmap-backlog`
13. `definition-of-done`
14. `observability-reliability`

## Candidate Bounded Contexts

- Company and tenant management
- Financial data ingestion
- Forecasting and scenarios
- KPI and investor metrics
- Alerts and recommendations
- Billing and entitlement
- Audit and compliance

## Example Behavior

```gherkin
Feature: Runway monitoring

Scenario: CFO sees a runway risk
  Given a company has 120000 EUR of cash
  And the average monthly burn is 30000 EUR
  When the CFO opens the finance dashboard
  Then the system shows a runway of 4 months
  And flags the company as below the 6 month safety threshold
```

## Important Disclaimer

This suite structures software/product work. It does not provide financial, legal, accounting, compliance, or investment advice.
