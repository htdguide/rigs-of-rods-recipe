# source/version_info/RoRVersion.cpp.in

> Build-time template that fills in the four version strings.

**Needs** — [`RoRVersion.h`](RoRVersion.h.md)
**Used by** — the build, which configures it into the definitions declared by [`RoRVersion.h`](RoRVersion.h.md)
**Tier floor** — T4: text substitution at build time

## Purpose

Data twin. A template whose placeholders (`@VERSION_YEAR@`, `@VERSION_MONTH@`, `@VERSION_SUFFIX@`, `@BUILD_DATE@`, `@BUILD_TIME@`) are substituted by the build before compilation, producing the definitions declared in [`RoRVersion.h`](RoRVersion.h.md).

## State

Stateless.

## Version derivation

**Contract** — the build decides year, month and suffix by one of three rules, in priority order.

```text
FUNCTION derive_version(custom_version: optional<text>, dev_build: bool, repo) -> (year, month, suffix)
  IF custom_version IS SET                    # release/installer builds pass e.g. "2026.01"
    PARSE custom_version AS "<year>.<month><rest>"
    RETURN (year, month, rest)
  IF dev_build
    year, month = current UTC year and zero-padded month
    IF repo IS a git work tree
      suffix = "-dev-" + short commit hash
      IF working tree differs from HEAD
        suffix = suffix + "-dirty"
    ELSE
      suffix = "-dev-without-git"
    RETURN (year, month, suffix)
  RETURN (year, month, "")                    # plain release with the project's own version
```

**Notes** — the date/time are always taken in UTC so two machines building the same commit agree. The companion template `RoRVersionDef.h.in` produces `ROR_RESOURCE_VERSION_STRING` and a numeric `year,month` pair for the Windows executable's version resource; only the Windows resource file consumes it.
