# Testing Rule Extraction Feature

This document provides a test scenario for the rule extraction feature in Claudeception.

## Test Scenario: ESLint No-Console Rule

### Scenario
While working on a production application, you discover console.log statements accidentally left in the code
that are exposing sensitive information. You want to prevent this from happening in the future.

### Expected Extraction: Both Skill and Rule

#### Skill: "removing-console-logs-from-production"
**What it would contain:**
- How to find all console.log statements in a codebase
- Manual removal techniques
- How to verify console logs are removed
- What to use instead (proper logging libraries)

#### Rule: "eslint-no-console-production"
**What it would contain:**
- ESLint configuration for `no-console` rule
- Environment-specific configuration (allow in dev, error in prod)
- Integration with build pipeline
- Custom wrapper function exceptions

### Testing the Feature

To test that rule extraction is working:

1. **Trigger Claudeception** after solving a problem that could be prevented
2. **Verify** it asks: "Should we extract a skill, a rule, or both?"
3. **Check** that it uses the rule template from `resources/rule-template.md`
4. **Ensure** it creates proper cross-references between skill and rule
5. **Validate** the rule includes:
   - Complete configuration
   - Installation steps
   - Examples of violations and fixes
   - False positive handling

## Success Criteria

✅ SKILL.md describes both skill and rule extraction  
✅ Rule template exists in resources/  
✅ Example rule exists in examples/  
✅ README documents rule extraction  
✅ Cross-references work between related skills and rules  
✅ Quality gates include rule-specific criteria  

## Manual Verification Checklist

- [ ] Read through SKILL.md - does it clearly explain when to extract rules?
- [ ] Check rule-template.md - is it comprehensive and easy to follow?
- [ ] Review example rule (typescript-no-circular-imports) - is it well-structured?
- [ ] Verify README explains the skill vs rule distinction clearly
- [ ] Confirm cross-references work in both directions
- [ ] Test that the extraction process is intuitive
