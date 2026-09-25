# Native FSDK image validation

`.github/workflows/fsdk-ci.yml` validates image changes on pull requests to
`testing`, pushes to `testing`, and manual dispatch. It uses native x86_64 and
AArch64 runners with independent six-hour limits. A new commit to a pull request
cancels that pull request's previous run; pushes are not cancelled.

Both jobs inherit only `contents: read`. Checkout does not persist credentials,
and the workflow has no registry login, secrets, publishing step, or privileged
`pull_request_target` trigger. Public builder images and sources must be readable
without registry credentials. Legacy Snap/Rock packaging and registry workflows
remain separate; FSDK validation does not trigger a release workflow.

Image inputs include the BuildStream graph and refs, Justfile, source, patches,
runtime files, scripts, and tests. Documentation-only and Snap/Rock-only changes
do not schedule native builds. Update the path list when adding another image
input outside these locations. Both architectures use the same image inputs.

## Prerequisite integration

Issues #2-#5 have not yet supplied the FSDK graph, runtime, and real socket-sink
harness. Until that work lands, the prerequisite job explicitly reports a skip;
a green prerequisite job is **not** proof of a passing image build. Issue #6
must remain open until both native builds and real verification have passed.
Remove the bootstrap skip once the graph is integrated.

The workflow follows the Ghostscript appliance's build interface:

- `project.conf` and `elements/oci/ps-printer-app.bst` define the image graph.
- `Justfile` (or `justfile`) provides `fetch`, `build`, and `verify` recipes.
- `just fetch` fetches immutable sources with bounded retries.
- `just build` builds and exports the complete local OCI image.
- `just verify` runs full appliance and payload verification, including executable
  `tests/core-appliance.sh` and `tests/core-payload.sh`. The payload test must send
  a real IPP job through the driver/filter/socket backend, check format-specific
  output bytes, and require job completion. It must also cover the persistence
  and coexistence contracts from #4 and #5. Synthetic echoes are not sufficient.

A partial graph fails the prerequisite check instead of skipping native builds.
The prerequisite check only checks interface files; the recipes and tests must
implement the behavior above. Physical paper output remains unverified without
hardware.

Validate workflow edits with `actionlint .github/workflows/fsdk-ci.yml`. After
prerequisites land, both native jobs must pass before claiming #6 is complete.
