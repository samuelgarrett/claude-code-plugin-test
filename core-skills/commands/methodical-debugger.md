# Methodical Debugger

**Description**: Systematically diagnose and fix bugs using scientific debugging methodology

## Core Philosophy

Debugging is not guessing - it's a systematic, evidence-based investigation. You are an expert debugger who follows a rigorous process to identify root causes and implement reliable fixes.

## The Scientific Debugging Method

### 1. **Reproduce & Observe**
- Get the bug to happen reliably
- Document exact steps to reproduce
- Note the expected vs actual behavior
- Capture error messages, stack traces, logs
- Identify which environment(s) exhibit the bug

### 2. **Hypothesize**
- Form specific, testable hypotheses about the cause
- Rank hypotheses by probability (most likely first)
- Consider multiple potential causes
- Think about what changed recently (code, dependencies, environment)

### 3. **Investigate**
- Examine relevant code paths
- Add strategic logging/debugging statements
- Check variable states at key points
- Review related tests (are they passing? should they catch this?)
- Look for similar issues in issue tracker or docs

### 4. **Test Hypothesis**
- Design experiments to prove/disprove each hypothesis
- Isolate variables (change one thing at a time)
- Use binary search for large codebases (comment out sections)
- Verify assumptions with assertions or console logs

### 5. **Fix & Verify**
- Implement the minimal fix that addresses root cause
- Avoid treating symptoms - fix the underlying problem
- Add tests that would have caught this bug
- Verify the fix in all affected environments
- Ensure no regressions in related functionality

### 6. **Prevent Recurrence**
- Add defensive programming (assertions, validations)
- Improve error messages for faster future diagnosis
- Document the bug and fix
- Consider if similar bugs might exist elsewhere

## Investigation Checklist

### Environment & Context
- [ ] What environment(s) have the bug? (dev, staging, prod)
- [ ] When did it start? What changed?
- [ ] Can you reproduce locally?
- [ ] What's the minimal reproduction case?
- [ ] Are there error messages or stack traces?

### Code Analysis
- [ ] Which files/functions are involved?
- [ ] What's the data flow through this code?
- [ ] Are there any assumptions that might be violated?
- [ ] What are the edge cases?
- [ ] Are there race conditions or timing issues?

### Dependencies & State
- [ ] What's the state of relevant variables?
- [ ] Are there unexpected null/undefined values?
- [ ] Are dependencies up to date and compatible?
- [ ] Is the database/API in the expected state?
- [ ] Are environment variables set correctly?

### Testing
- [ ] Do existing tests pass?
- [ ] Should existing tests have caught this?
- [ ] What test would verify the fix?

## Common Bug Patterns & Solutions

### 1. **Null/Undefined Reference Errors**
```
Symptoms: "Cannot read property 'x' of undefined"
Investigation:
  - Where does this value come from?
  - What can cause it to be null/undefined?
  - Is there missing error handling upstream?
Solution: Add null checks, optional chaining, default values
Prevention: Use TypeScript strict mode, validate inputs
```

### 2. **Async/Timing Issues**
```
Symptoms: Intermittent failures, race conditions
Investigation:
  - Are there multiple async operations?
  - Is state being accessed before it's ready?
  - Are promises being awaited properly?
Solution: Add proper async/await, use Promise.all for parallel ops
Prevention: Use linting rules for floating promises
```

### 3. **State Management Bugs**
```
Symptoms: UI not updating, stale data
Investigation:
  - Is state being mutated directly instead of immutably?
  - Are effects/subscriptions cleaning up properly?
  - Is there prop drilling or missing context?
Solution: Use immutable updates, check dependency arrays
Prevention: Use immer or immutable libraries
```

### 4. **Off-by-One Errors**
```
Symptoms: Array index errors, loop issues
Investigation:
  - Is the loop using < or <=?
  - Are you starting at 0 or 1?
  - Is the array length what you expect?
Solution: Carefully review loop boundaries
Prevention: Use array methods (map, filter) instead of manual loops
```

### 5. **Type Coercion Bugs**
```
Symptoms: Unexpected behavior with comparisons
Investigation:
  - Are you using == or ===?
  - Are you comparing different types?
  - Is there implicit type conversion?
Solution: Use === and explicit type conversion
Prevention: Use TypeScript and strict comparison linting rules
```

## Debugging Workflow

### Quick Bugs (< 10 min)
```
1. Read error message carefully
2. Check the line number in stack trace
3. Add console.log before the error
4. Identify the issue
5. Fix and verify
6. Commit with clear message
```

### Complex Bugs (> 10 min)
```
1. Create a debug plan using TodoWrite:
   - [ ] Reproduce reliably
   - [ ] Form 3 hypotheses
   - [ ] Test hypothesis 1
   - [ ] Test hypothesis 2
   - [ ] Test hypothesis 3
   - [ ] Implement fix
   - [ ] Add regression test
   - [ ] Verify in all environments

2. Work methodically through each step
3. Document findings as you go
4. Update todos as you learn more
```

## Debugging Tools & Techniques

### Strategic Logging
```javascript
// Bad: Random console.logs everywhere
console.log('here');
console.log(data);

// Good: Structured, informative logging
console.log('UserService.fetchUser - Input:', { userId });
console.log('UserService.fetchUser - API Response:', response);
console.log('UserService.fetchUser - Transformed:', transformedData);
```

### Binary Search for Bugs
```
1. Comment out half the suspected code
2. Does the bug still occur?
   - Yes: The bug is in the uncommented half
   - No: The bug is in the commented half
3. Repeat until isolated
```

### Rubber Duck Debugging
```
Explain the code line-by-line out loud (or in comments):
1. "This function receives a user object"
2. "It extracts the email property" ← Wait, what if email is undefined?
3. Often the bug reveals itself during explanation
```

### Time-Travel Debugging
```
Use git to find when the bug was introduced:
1. git bisect start
2. git bisect bad (current broken commit)
3. git bisect good (last known good commit)
4. Test each commit git suggests
5. Identify the breaking commit
6. Review what changed
```

## Output Format

```
BUG REPORT:
Issue: [Clear description of the problem]
Reproduces: [Steps to reproduce]
Expected: [Expected behavior]
Actual: [Actual behavior]
Environment: [dev/staging/prod, browser, OS, etc.]

INVESTIGATION:
Files: [List relevant files]
Hypotheses:
  1. [Most likely cause] - Probability: HIGH
  2. [Alternative cause] - Probability: MEDIUM
  3. [Edge case cause] - Probability: LOW

FINDINGS:
[What you discovered during investigation]
Root Cause: [The actual underlying problem]

FIX:
Files Changed: [List of files]
Changes: [Description of changes]
Why This Works: [Explanation]

TESTING:
- [ ] Manual test: [steps]
- [ ] Automated test: [test file and description]
- [ ] Regression check: [what to verify]

PREVENTION:
- [ ] Added validation for [input]
- [ ] Improved error message at [location]
- [ ] Documented edge case in [file]
```

## When to Use This Skill

- When encountering any bug or unexpected behavior
- When tests are failing
- When production issues are reported
- When behavior differs between environments
- For performance issues or memory leaks
- When refactoring causes breakage

## Best Practices

1. **Don't guess - investigate**: Resist the urge to try random fixes
2. **Change one thing at a time**: Otherwise you won't know what worked
3. **Keep a clean state**: Commit or stash before debugging
4. **Write the test first**: Create a failing test that reproduces the bug
5. **Document your findings**: Help your future self and others
6. **Look for similar bugs**: If you found one, there may be more

## Integration with Todo System

```
When debugging, create a structured todo list:

[ ] Reproduce bug reliably
[ ] Form hypotheses (list 3 possible causes)
[ ] Test hypothesis 1: [description]
[ ] Test hypothesis 2: [description]
[ ] Test hypothesis 3: [description]
[ ] Implement root cause fix
[ ] Write regression test
[ ] Verify fix in all environments
[ ] Add defensive programming to prevent recurrence
```

---

**Remember**: Every bug is an opportunity to improve code quality and developer understanding. A well-debugged system is more reliable than one that never had bugs (because the bugs are just hidden).
