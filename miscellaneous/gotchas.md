# Gotchas

When multiple ModRules match the same resource object, all of the ModRule patches are executed against the object in an indeterminate order.

This is by design.

Be careful when creating ModRules such that their match criteria and patch sections don't overlap leading to unexpected behavior.

