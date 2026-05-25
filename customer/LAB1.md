# Lab 1: Try Predictive Test Selection (PTS) locally

## Overview

You'll use the Smart Tests CLI on your repository to see how PTS re-ranks tests based on a code change.

## Step 1: Generate an initial ranked test list

Create a test session for the baseline build and request an initial ranked list of tests.

```bash
smart-tests record session \
  --build baseline \
  --test-suite my-test-suite > session.txt

smart-tests subset file \
  --session @session.txt \
  --get-tests-from-guess > subset.txt
```

**Verify:** You should see output like:

```
Smart Tests created subset <SUBSET_ID> for build baseline (test session <TEST_SESSION_ID>) in workspace <ORG>/<WORKSPACE>

|           |   Candidates |   Estimated duration (%) |   Estimated duration (min) |
|-----------|--------------|--------------------------|----------------------------|
| Subset    |          120 |                   100.00 |                       0.00 |
| Remainder |            0 |                     0.00 |                       0.00 |
|           |              |                          |                            |
| Total     |          120 |                   100.00 |                       0.00 |

Run `smart-tests inspect subset --subset-id <SUBSET_ID>` to view full subset details
```

## Step 2: Make a small code change

Make a small edit and create a new commit.

```bash
vim <UPDATE YOUR APP or TEST CODE>
git commit --all --message test
```

**Verify:** You should see:

```
1 file changed, 1 insertion (+), 1 deletion (-)
```

## Step 3: Record a new build

Record the changed code version as a new build called `mychange`.

```bash
smart-tests record build --build mychange
```

**Verify:** You should see output like:

```
Smart Tests recorded 2 more commits from repository <YOUR PATH>
Smart Tests recorded build mychange to workspace <YOUR ORG/WORKSPACE> with commits from 1 repository:

| Name   | Path   | HEAD Commit                              |
|--------|--------|------------------------------------------|
| .      | .      | 3f21bfb3d56148c9dcf9f7e811e146bbc3cbf797 |
```

## Step 4: Generate a new ranked test list

Create a new test session and request a subset for the changed code.

```bash
smart-tests record session \
  --build mychange \
  --test-suite my-test-suite > session2.txt

smart-tests subset file \
  --session @session2.txt \
  --use-case one-commit \
  --get-tests-from-guess > subset2.txt
```

**Verify:** You should see a similar table as in Step 1 with subset details.

## Step 5: Analyze impact on test rankings

Analyze the subset to verify that PTS prioritized the right tests based on file changes.

```bash
smart-tests analyze subset <SUBSET_ID_2>
```

**Verify:** After selecting expected tests, the command should display a summary indicating the effectiveness of PTS.

---

**Lab 1 Complete** - Proceed to Lab 2.
