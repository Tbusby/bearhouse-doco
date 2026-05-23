# Change Management

## Change Model

All non-emergency changes should follow this flow:

1. update code or documentation
2. commit changes to Git
3. validate in CI
4. review changes
5. merge to main
6. allow automation to apply or reconcile changes

## Manual Changes

Manual changes are discouraged except:
- during initial bootstrap
- during incident response
- for temporary testing that will later be captured in code

## Documentation Requirement

Significant changes should update:
- architecture docs
- runbooks
- ADRs where appropriate