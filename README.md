# TugQL (formerly DTQL)

TugQL specification — an open, portable query language and structured query tree (**TugQT**) for relational, document, API, and graph-connected data.

- **Canonical Public Home:** [https://tugql.org](https://tugql.org)
- **Previous Name:** DTQL (dtql.org permanently redirects to tugql.org)
- **Query Language:** **TugQL** (SQL-like syntax)
- **Query Tree Representation:** **TugQT** (structured JSON/YAML query tree)

> **Naming & Rebrand Decision:**
> DTQL has been renamed to TugQL. TugQL is the canonical name of the SQL-like query language and TugQL.org is its canonical public home. DTQL.org is retained as a permanent redirect for backwards compatibility and discovery.
> TugQT (Tug Query Tree) is the intended name for the human-readable, machine-readable JSON/YAML representation of the parsed/canonical query tree.

## Used in DataTug development

[DataTug](https://datatug.app) is evolving its query contract around TugQL and TugQT, with work spanning query definitions and how they connect to data sources. [See the tools behind DataTug](https://datatug.app/en/under-the-hood/tools-we-built/#query).

## References

The [negative projection reference](reference/negative-projection.md) defines
how a query selects all columns except an explicit list, including qualified
source scope and missing-name behavior.

The [aggregation reference](reference/aggregation.md) defines YAML-native
grouping, aggregate expressions, HAVING, aliases, null/empty semantics, and the
provider-independent execution contract.

The [layered ACL MVP design packet](design/layered-acl/README.md) proposes portable policy and authorization contracts across TugQL, DALgo, InGitDB, OpenVaultDB and DataTug. It includes the current-state audit, implementation work packages and independent review material. Its implementation plan and contracts have independent review, reconciliation and correction verification, and await human dispatch approval; the MVP is not implemented by this packet.

The [human checkpoint and review report](design/layered-acl/reviews/round-3/README.md) includes both independent reviews, separate reconciliation, resolved findings and measured review cost/value. The implementation plan was reviewed alongside the design; implementation dispatch awaits human sign-off.
