# Python/Kotlin Parity Testing

Use this reference to design evidence that the Kotlin implementation preserves the requested Python behavior.

## Fixture layers

Build fixtures in layers:

1. scalar and tiny-array cases for arithmetic and boundary rules;
2. synthetic images or tensors that isolate channel, resize, mask, and coordinate behavior;
3. model/inference fixtures with saved inputs and raw expected outputs;
4. representative real inputs;
5. malformed, empty, extreme, and unsupported inputs.

Store the input, Python output, configuration, model/resource identity, and comparison metadata together. Avoid fixtures that depend on an uncontrolled current working directory or mutable external service.

## Comparison rules

- Use exact comparisons for shapes, labels, counts, indices, and deterministic discrete fields.
- Use absolute and relative tolerances for floating-point values; record both values and the reason for the tolerance.
- For images and masks, report maximum error, mean error, changed-pixel ratio, and relevant regions rather than only a single pass/fail value.
- Compare ordering and absent-versus-empty fields in structured outputs.
- Use invariant checks for nondeterministic outputs, such as valid ranges, conservation rules, or set membership.

Do not loosen a tolerance until the first divergence has been localized and its cause understood.

## Diagnosing divergence

Check in this order:

- input decoding, orientation, color/channel order, and dtype;
- shape, axis, stride, and padding;
- normalization, rounding, clipping, and interpolation;
- coordinate origin and bounds;
- model file, runtime version, provider, and options;
- postprocessing thresholds, sorting, tie handling, and filtering;
- concurrency, random seeds, and lifecycle state.

Capture intermediate values at the first divergent stage. A final-output comparison alone often cannot distinguish a data-layout bug from a postprocessing difference.

## Reporting

For each parity run, record the fixture identifier, implementation versions, configuration, tolerances, result summary, and known differences. Classify each difference as accepted, environment-specific, unresolved, or requiring a design decision.
