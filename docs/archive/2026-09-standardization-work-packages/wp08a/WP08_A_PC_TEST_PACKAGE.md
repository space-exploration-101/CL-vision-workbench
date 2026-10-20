# WP08-A PC software test package

This additive L0 module packages PC-only software evidence without changing the
WP01, WP06, WP07, WP09, WP10, or legacy WP08 A1 contracts. The normative design
is `ADR-20260916-WP08_A_PC_TEST_PACKAGE`.

## Scope

`pipeline.release_l0.pc_test_package` defines and validates three versioned
canonical JSON documents:

- `pc-test-package-member-set-v1`: the closed inventory of every non-control
  payload member;
- `pc-test-package-index-v1`: exact references to WP01 manifests, evaluation
  and backend inputs, a complete WP06 trace, and the WP07 outcome closure;
- `pc-test-package-wrapper-v1`: exact references to the index/member set plus
  the deterministic archive profile.

The archive contains those three documents, the closed payload, and
`SHA256SUMS`. It is a deterministic `ZIP_STORED` file with sorted paths, fixed
DOS epoch timestamps, mode `0640`, streaming copies, fsync, and no-replace
publication through a unique `.partial` file. Its outer size and SHA-256 are
returned as a locator candidate and are never written back into the archive.

## API

The stable entry points are:

```python
member_set_sha256(value)
package_index_sha256(value)
package_wrapper_sha256(value)
validate_pc_test_member_set(value, reader)
build_pc_test_package_index(...)
validate_pc_test_package_index(value, resolver, reader)
build_pc_test_package_wrapper(...)
validate_pc_test_package_wrapper(value, index, member_set)
assemble_pc_test_package(target, *, wrapper, index, member_set, reader, resolver)
verify_pc_test_package(path, *, resolver,
                       expected_outer_sha256=None,
                       expected_outer_size=None)
```

`PCTestPackageResolver` and `PackageReader` are read-only protocols. Assembly
requires a resolver because it performs the same semantic validation as
verification before writing. The module reuses the WP01, WP06, and WP07 public
validators; it does not duplicate or weaken their rules.

## State and safety rules

- A complete PC flow or an exact `generate_bin` known block is
  `software_validated_partial`; an ordinary execution failure is `failed`.
- A known block may make `merge_bin` and `bundle` `skipped_dependency`, but no
  evidence or artifact may be invented for those stages.
- An incomplete trace, context, resolver closure, or inventory is rejected and
  no package is published.
- The execution boundary is always `x86_64_pc`, with FPGA use, hardware
  validation, promotion, and production readiness all false.
- Legacy A1 release bundles are neither accepted nor upgraded automatically.

All validation failures raise `PCTestPackageError`; callers should branch on
its stable `code`, not on the human-readable message.

## L0 verification

Run the focused CPU tests from the repository root:

```bash
python3 -m unittest -v tests.release_l0.test_pc_test_package
```

The focused suite uses only in-memory fixtures and temporary files. Production
resolver integration, real traces/registry records, multi-gigabyte streaming
evidence, FPGA/HIL validation, deployment, and promotion remain outside L0.
