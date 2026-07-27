# @mgreten/release-adoption

`@mgreten/release-adoption` is a deterministic campaign engine for adopting new
swamp releases across a fleet of repositories. It captures release notes and
the machine-readable CLI surface, computes upgrade deltas, inventories workflow
and extension usage, and stores adoption opportunities as durable swamp data.
The companion method report renders the latest campaign as an operator brief.

## Install

After reviewer approval and registry publication:

```sh
swamp extension pull @mgreten/release-adoption
```

The model uses `gh` for release notes and `swamp` for CLI surface capture. The
pre-flight checks verify each binary and GitHub authentication before the
relevant method runs. Fleet inventory only reads configured local paths and
isolates failures so one absent or malformed repository cannot abort the scan.

## Configure and run

```sh
swamp model create @mgreten/release-adoption release-adoption \
  --global-arg githubRepo=swamp-club/swamp \
  --global-arg 'fleetRepos=[{"name":"product","path":"/srv/product"}]'

swamp model method run release-adoption open-campaign \
  --input id=july --input fromVersion=20260720.000000.0 \
  --input toVersion=20260727.000000.0
swamp model method run release-adoption fetch-release-notes \
  --input campaignId=july --input fromVersion=20260720.000000.0 \
  --input toVersion=20260727.000000.0
swamp model method run release-adoption inventory-fleet --input campaignId=july
```

Capture CLI surfaces before and after an upgrade, then compare them:

```sh
swamp model method run release-adoption capture-surface
swamp model method run release-adoption diff-surface \
  --input fromVersion='swamp 2026.07.20' --input toVersion='swamp 2026.07.27'
```

Agents write proposals and outcomes with `record-opportunity`. Reusing an ID
merges the update with the existing ledger entry, preserving fields omitted by
the caller and always refreshing `updatedAt`.

## Resources

Campaigns, release notes, CLI surfaces, diffs, and opportunities are retained
indefinitely with bounded version history. Fleet inventories expire after 90
days. Resource names are deterministic and prefixed by their spec to prevent
cross-spec collisions.

## License

MIT — see [LICENSE](LICENSE).
