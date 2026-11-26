# Next

Where work stands right now. Point to the documents that hold the detail
instead of restating them, and replace this page's contents as work moves on.

- **Active change:** the founding set in `intent/`.
- **Stage:** Plan. [Problem statement](PROBLEM.md) and
  [Handover](HANDOVER.md) are drafts for the owner's review;
  `intent/intent.md` is still the template skeleton.
- **Next action:** the owner writes `intent/intent.md` from the problem
  statement, answering or carrying forward its open questions, and sets
  `status: accepted`. Then write `intent/spec.md`, starting with the
  model's joint factorization and the data contract.
- **Dates:** complete paper by mid-January 2027 for internal review; paper
  due end of January 2027.
- **Dependency:** `synthetic_evm` requires Python 3.13 or later, so it will
  not install into this 3.12 project until its floor is lowered and a
  release is tagged.
- **Repository settings:** after the first CI run, set the merge options and
  create the `main` ruleset listed under "GitHub settings" in
  `docs/DEVELOPMENT.md`. Then remove this line.
- **Blocked by:** nothing.
