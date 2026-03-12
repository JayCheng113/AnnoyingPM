# Change Checklist

Use for LARGE changes. For SMALL/MEDIUM, use as mental walkthrough — no need to fill this out.

## CLASSIFY
- [ ] Type: ADDITIVE / MODIFYING / REPLACING
- [ ] Why: ___
- [ ] Affected area: ___

## SURVEY
- [ ] Affected files listed (via Grep/Glob, NOT memory)
- [ ] Tests covering affected area identified
- [ ] Conflicts with existing requirements: none / [describe]
- [ ] (MODIFYING/REPLACING) Current behavior summarized before changes
- [ ] (REPLACING) Implicit/undocumented behaviors inventoried
- [ ] (AI/ML) Performance baselines documented with metrics

## SCOPE
- [ ] Task list with dependency order
- [ ] DO NOT TOUCH list
- [ ] Non-Goals: what this change does NOT address
- [ ] (MODIFYING/REPLACING) Preserve list: behaviors that must still work
- [ ] Scope creep check: stays within original request

## PRE-IMPLEMENTATION SELF-CHECK
- [ ] I can list ALL affected files
- [ ] I can describe current behavior of code I'm changing
- [ ] I have a DO NOT TOUCH list
- [ ] I have Non-Goals defined
- [ ] I am NOT about to change something "because I'm already here"

## VERIFY
- [ ] Full test suite passes (not just changed tests)
- [ ] All scope items addressed
- [ ] All DO NOT TOUCH items actually untouched
- [ ] Non-Goals confirmed not accidentally addressed
- [ ] (MODIFYING/REPLACING) Each preserved behavior verified
- [ ] Docs/specs updated

## LEARN
- [ ] Surprises: ___
- [ ] Failed attempts: ___
- [ ] Would-do-differently: ___
