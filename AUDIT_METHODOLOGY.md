# Audit Failure Analysis & Re-Audit Methodology

**Document Created:** December 18, 2025
**Context:** Post-mortem analysis of a failed code audit that missed a 27% tax calculation error

---

## Part 1: What Happened

### The Failure
The original audit stated tax calculations were "accurate" when they were **27% wrong** (~$7,110 overstated on $108k income).

Two critical deductions were completely missing:
- **QBI Deduction (Section 199A):** ~$17,319 missing
- **Business Expense Deduction:** ~$17,519 not applied to tax calculation

### Root Causes Identified

| # | Root Cause | Description | Severity |
|---|------------|-------------|----------|
| 1 | **Code Review Mode vs Tax Expert Mode** | Auditor checked if code does what it says, not whether it's complete per tax law. Knowledge of QBI existed but wasn't applied. | CRITICAL |
| 2 | **Looked at What WAS There, Not What SHOULD Be** | QBI wasn't in code, so it wasn't reviewed. Backwards thinking - should have started with "what must exist?" | CRITICAL |
| 3 | **Trusted Variable Names** | `calculateTotalTax(netIncome, ...)` - accepted the name without verifying the parameter actually contained net income (it was gross). | HIGH |
| 4 | **Compartmentalized Features** | Saw expenses as "cash flow feature" and tax as "separate feature" - never asked "why don't these connect?" | HIGH |
| 5 | **No Validation Against Real Data** | Never compared output to actual tax return. 27% error would have been immediately obvious. | MEDIUM |
| 6 | **No Completeness Checklist** | Never established upfront "here's what a self-employed tax calculation requires" then verified each item. | CRITICAL |

### The Deeper Failure

The auditor (Claude) had knowledge of QBI/Section 199A in training but failed to apply it because:

1. **Task framing:** "Review this code" ≠ "Verify tax law compliance"
2. **Pattern matching on existing code** rather than questioning completeness
3. **Confirmation bias:** Professional-looking code assumed to be complete
4. **Reactive vs proactive:** Looked at what WAS there, not what SHOULD be there

**Key Insight:** Mechanical code review is insufficient for financial software. Domain-specific validation and real data comparison are essential.

---

## Part 2: Re-Audit Methodology

### Phase 1: Establish Domain Requirements FIRST

**Before looking at code**, define what a correct implementation requires.

For a self-employed sole proprietor (service business, single filer):

#### Required Tax Components Checklist

| Component | IRS Form/Line | What to Verify |
|-----------|---------------|----------------|
| **Gross Income** | Schedule C Line 1 | User input captured correctly |
| **Business Expenses** | Schedule C Lines 8-27 | Deducted from gross to get net profit |
| **Net Profit** | Schedule C Line 31 | Calculated as Gross - Expenses |
| **SE Tax Base** | Schedule SE | Net profit × 0.9235 |
| **Social Security Tax** | Schedule SE | 12.4% up to wage base ($176,100 for 2025) |
| **Medicare Tax** | Schedule SE | 2.9% on full SE base (no cap) |
| **Additional Medicare** | Schedule SE | 0.9% on amounts over $200k |
| **SE Tax Deduction** | Form 1040 Schedule 1 | 50% of SE tax reduces AGI |
| **Adjusted Gross Income** | Form 1040 | Net profit - SE tax deduction |
| **Standard Deduction** | Form 1040 | $15,000 single (2025) |
| **QBI Deduction** | Form 8995/8995-A | 20% of QBI, with SSTB limits |
| **Taxable Income** | Form 1040 | AGI - Standard Ded - QBI |
| **Income Tax** | Form 1040 | Progressive brackets applied |

**Audit must verify EACH of these exists and is implemented correctly.**

### Phase 2: Data Flow Tracing

For each user input, trace through to tax output:

```
User Input: Weekly Income
    ↓
Where stored? → appData.history[]
    ↓
How annualized? → averageWeekly × 52
    ↓
Is this GROSS or NET? → Must verify (don't trust variable names)
    ↓
Where do expenses get subtracted? → Must find explicit subtraction
    ↓
What's passed to tax calculation? → Trace actual parameter values
    ↓
What components are calculated? → Verify against checklist
```

**Rule: Never trust variable names. Trace actual values.**

### Phase 3: Skeptical Variable Audit

For each tax-related function, verify:

| Check | Question to Ask |
|-------|-----------------|
| Parameter names | Does `netIncome` actually receive net income? |
| Return values | Does `totalTax` include all tax components? |
| Constants | Are tax rates/brackets for the current year? |
| Edge cases | What happens at $0? At $1M? At wage base threshold? |

**Create explicit mapping:**
```
Variable Name → Expected Value → Actual Value → Match?
netIncome     → gross - expenses → [trace it]  → YES/NO
```

### Phase 4: Completeness Validation

Ask these questions explicitly:

1. "What tax deductions exist for [this type of taxpayer] that are NOT in this code?"
2. "What user inputs exist that should affect tax but don't flow to tax calculation?"
3. "What IRS forms would this person file, and does the app cover each line item?"

**Don't just verify what exists. Actively search for what's missing.**

### Phase 5: Real Data Validation

**If actual tax return data available:**
- Input prior year gross income and expenses
- Compare app output to actual tax paid
- Any variance > 5% = red flag requiring explanation

**If no actual data:**
- Use IRS tax scenarios/examples
- Cross-check with established tax calculator websites
- Document that validation was limited

### Phase 6: Variable Naming Audit

**Flag for human review any variable where:**
- Name implies a tax concept (net, gross, taxable, deduction, income)
- Actual value doesn't match the tax meaning of that name
- Create explicit mapping document for tax professional review

---

## Part 3: Application Scope

### What This App Covers (Business Tax)

| Feature | Status |
|---------|--------|
| Gross income tracking | Implemented |
| Business expense deduction (Schedule C) | Implemented |
| Net profit calculation | Implemented |
| SE tax with proper wage base cap | Implemented |
| QBI deduction with SSTB phase-out | Implemented |
| Income tax with 2025 brackets | Implemented |
| Standard deduction | Implemented |

### What This App Does NOT Cover (Personal)

- Student loan interest deduction
- Personal health insurance
- Retirement contributions (SEP-IRA, Solo 401k)
- Investment income/losses
- Itemized deductions
- Other income sources

**Design Decision:** This app calculates business tax for a self-employed sole proprietor. It is not a complete personal tax return calculator.

---

## Part 4: Action Items

### Completed Fixes
- [x] Added QBI deduction (Section 199A) with SSTB phase-out
- [x] Fixed business expense deduction in tax calculation
- [x] Added Social Security wage base cap ($176,100)
- [x] Split SE tax into components (SS, Medicare, Additional Medicare)

### Follow-Up Required

| Action | Priority | Status |
|--------|----------|--------|
| Verify all variable names match actual tax semantics | HIGH | COMPLETED |
| Validate against actual 2024 tax return data | HIGH | COMPLETED |
| Annual update: 2026 tax constants when available | MEDIUM | Future |

---

## Part 6: Re-Audit Results (December 18, 2025)

Using the methodology above, a re-audit was executed on the corrected codebase.

### Phase 1: Tax Components Checklist

| # | Component | Status |
|---|-----------|--------|
| 1 | Gross Income | PASS |
| 2 | Business Expenses | PASS |
| 3 | Net Profit (Gross - Expenses) | PASS |
| 4 | SE Tax Base (Net × 0.9235) | PASS |
| 5 | Social Security Tax (12.4%, capped) | PASS |
| 6 | Medicare Tax (2.9%, no cap) | PASS |
| 7 | Additional Medicare (0.9% over $200k) | PASS |
| 8 | SE Tax Deduction (50%) | PASS |
| 9 | Adjusted Gross Income | PASS |
| 10 | Standard Deduction | PASS |
| 11 | QBI Deduction (Section 199A) | PASS |
| 12 | Taxable Income | PASS |
| 13 | Income Tax (progressive brackets) | PASS |

**Result: 13/13 components verified**

### Phase 2: Data Flow Verification

Traced from user input through to tax output:
- Weekly income → stored correctly
- Annualized as gross (× 52) → confirmed
- Expenses calculated from budget → confirmed
- Net profit = gross - expenses → verified at line 264
- SE tax on net profit → verified at line 267
- Income tax with QBI on net profit → verified at line 270

**Result: VERIFIED**

### Phase 3: Variable Naming Audit

| Variable | Tax Meaning | Actual Value | Match? |
|----------|-------------|--------------|--------|
| projectedAnnual | Gross annual | avgWeekly × 52 | YES |
| grossIncome | Gross before expenses | projectedAnnual | YES |
| netProfit | Schedule C Line 31 | gross - expenses | YES |
| seTaxBase | 92.35% of net | netProfit × 0.9235 | YES |
| adjustedIncome | AGI | net - SE deduction | YES |
| taxableIncome | Final taxable | AGI - StdDed - QBI | YES |

**Result: All variable names now match actual values**

### Phase 5: Real Data Validation

Tested against actual 2024 tax return:
- Input: $126,286 gross, $17,519 expenses
- App calculates: $25,642 total tax
- Actual return: $25,662 total tax
- **Difference: $20 (0.08% error)**

**Result: PASS - within acceptable tolerance**

### Re-Audit Conclusion

The corrected implementation passes all verification phases. The tax calculation is now accurate for a self-employed sole proprietor (single filer, service business).

---

## Part 5: Lessons Learned

### For Future Audits

1. **Start with requirements, not code.** Define what "correct" means before reviewing implementation.

2. **Apply domain expertise proactively.** Don't wait for code to prompt tax knowledge - bring it in first.

3. **Never trust names.** Variable named `netIncome` might contain gross. Verify.

4. **Ask "what's missing?"** Not just "is this correct?" but "is this complete?"

5. **Validate against reality.** A single comparison to actual tax return would have caught this immediately.

6. **Connect the features.** If users enter expenses AND the app calculates tax, those should connect. Question when they don't.

### The Core Failure Mode

**The audit verified the code did what it said. It never verified the code said what it should.**

This is the difference between code review and domain audit. Financial software requires both.
