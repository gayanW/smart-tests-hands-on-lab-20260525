# Lab 1: SE Guide - Predictive Test Selection (PTS) Local Evaluation

## Purpose & Goals

This lab demonstrates PTS re-ranking behavior to help customers evaluate whether Smart Tests can effectively identify relevant tests for their codebase.

**What we're measuring:**
- **Ranking quality**, NOT time saved
- Do "expected impacted tests" move **meaningfully higher** (e.g., into the top ~50%) after a code change?

**Key concept to explain:**
- In this lab, customers see a **full ranked list** - that's intentional
- In production, they'd set an **optimization target** (e.g., "target 50%") to get a subset
- Today we focus only on **re-ranking behavior** to validate PTS effectiveness

## Step 1: Generate an initial ranked test list

### What's happening
- Creates a **baseline test session** associated with the baseline build
- Requests an initial ranked list using `--get-tests-from-guess` (returns all tests, but ranked)

### Why `--get-tests-from-guess`?
We don't yet have execution history, so we can't calculate an optimization target. This flag tells Smart Tests to return all tests in relevance order so we can observe ranking behavior.

### Common questions
**Q: Why are all tests selected (100%)?**
A: This is expected. We're using `--get-tests-from-guess` to see the full ranking. In production with execution history, you'd specify `--target 50%` or similar.

**Q: Why is duration 0.00?**
A: AI doesn't have test duration data yet to analyze. After recording actual test results, it will show estimated durations based on AI analysis of historical data.

## Step 2: Make a small code change

### What's happening
Customer makes a targeted code change so they can evaluate if Smart Tests detects the impact.

### SE guidance
- Suggest they modify a well-isolated file (e.g., a utility function or specific module)
- This makes evaluation clearer - they'll know which tests SHOULD be affected
- Remind them: this is temporary, won't be merged

### Critical: Identify expected tests upfront
Before they make the code change, have the customer identify 3-5 specific tests they expect PTS should pick up based on the file they plan to modify. They should write these test names down - this creates concrete expectations they'll validate in Step 5.

## Step 3: Record a new build

### What's happening
Records the new code state with a different build name (`mychange`) so Smart Tests can:
1. Calculate the diff from baseline
2. Use that diff to inform test selection

### Key concept
A "build" in Smart Tests = a snapshot of the code state (commit SHAs from all repos involved)

## Step 4: Generate a new ranked test list

### What's happening
- Creates a new test session for the `mychange` build
- Uses `--use-case one-commit` to optimize for "what tests should run for this single commit?"
- Generates a new ranked list informed by the code change

### Why `--use-case one-commit`?
Tells Smart Tests' AI to optimize for "short-lived branch" scenario (vs. long-running feature branches or main branch testing).

## Step 5: Analyze impact on test rankings

### What's happening
The `analyze subset` command provides an **interactive analysis** of PTS effectiveness:
1. Shows which code files were modified
2. Prompts customer to select which tests they expected PTS to prioritize (interactive fuzzy search)
3. Displays an effectiveness summary

### How to guide the customer

**Step 1: Show modified files**
The command displays which files changed. This helps confirm PTS analyzed the right code changes.

**Step 2: Interactive test selection**
- Customer sees a fuzzy-searchable list of all tests
- They select the 1-2 tests that they expected to be prioritized
- Use Tab/Space to toggle selection, Enter to confirm

**Step 3: Review effectiveness summary**
After selecting expected tests, the command will show one or more of these sections:

- **✅ Tests Promoted**: Expected tests that moved up significantly (e.g., "promoted by 42 positions to rank #1")
  - Shows how much each test moved UP in ranking
  - This is the "flashiest" metric - clear evidence PTS is working

- **✅ Tests Prioritized**: Expected tests in top 50%
  - Shows which expected tests landed in top half of rankings
  - Indicates PTS understood the impact area

- **✅ Test Selection**: High-confidence picks (density > 0.6)
  - Density measures correlation strength between code change and test
  - 0.8+ = very strong, 0.7+ = strong, 0.6+ = moderate
  - Shows PTS has strong signal for these tests

### Success criteria
**Strong signal (ideal):**
- Expected tests show promotion (moved up significantly)
- Expected tests in top 50%
- High density scores (0.7+)

**Moderate signal:**
- Expected tests in top 50% but minimal promotion
- Density scores 0.6-0.7

**Weak signal:**
- Expected tests not in top 50%
- Low/no promotion
- Low density scores

### What if expected tests aren't prioritized?

The command will offer **AI-powered suggestions**:
- Suggests tests that clearly relate to the changed files
- Customer can select a suggestion to see its effectiveness summary

### What if results are poor?
- Check: Was the code change too broad or cross-cutting?
- Check: Are test names/structure representative of what they test?
- Note: With zero execution history, AI has limited data to analyze - AI analysis improves with more historical data
- Consider: Schedule follow-up after Lab 2 when execution data exists

## Troubleshooting

**Issue:** Customer can't find subset IDs for comparison
**Solution:** Subset ID is in the output table. Also available via: `smart-tests inspect subset --subset-id <ID>` or in the web UI.

**Issue:** All tests rank exactly the same
**Solution:** Likely the code change wasn't detected. Check:
- Did they commit the change?
- Did they run `smart-tests record build --build mychange` AFTER the commit?
- Is the file they changed actually in the repo (not .gitignore'd)?

**Issue:** Rankings seem random
**Solution:** With zero history, AI has limited data to analyze. Emphasize that AI analysis improves after Lab 2 when we record actual test results.
