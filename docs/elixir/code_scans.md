# Code Scans 

## Ignore Dependencies

This requires version `2.11.0` or later.

To ignore dependency-related findings in Paraxial.io code scans, create the file `.paraxial-ignore-deps` in your project directory. Each line is one rule:

- **Ignore an entire dependency** — the dependency name on its own line. All findings for that dependency are ignored, from both `mix deps.audit` and `mix hex.audit` (retired packages and security advisories).
- **Ignore a specific vulnerability** — the dependency name, a space, then the advisory ID. Only that finding is ignored; other vulnerabilities in the same dependency are still reported.

Blank lines and lines starting with `#` are skipped.

Example:

```
% cat .paraxial-ignore-deps
# Entire dependencies
hackney
jason
remote_ip

# Specific vulnerabilities
postgrex CVE-2026-32687
paginator GHSA-w98m-2xqg-9cvj
```

Notes on matching:

- Matching is case-insensitive: `postgrex CVE-2026-32687`, `postgrex cve-2026-32687`, and `Postgrex cve-2026-32687` are all equivalent.
- `CVE`, `GHSA`, and `EEF-CVE` advisory IDs are accepted.
- Advisory aliases are matched. For example, `mix hex.audit` may report `EEF-CVE-2026-43969` with the alias `CVE-2026-43969` — writing either ID in `.paraxial-ignore-deps` ignores the finding.

Sample output:

```
% mix paraxial.scan

16:53:49.918 [info] [Paraxial] v2.11.0, scan starting
16:53:49.920 [info] [Paraxial] API key found, scan results will be uploaded
[Paraxial] .paraxial-ignore-deps read, the following will be ignored
  Entire Dependencies:
    - hackney
    - jason
    - remote_ip
  Specific Vulnerabilities:
    - postgrex CVE-2026-32687
    - paginator GHSA-w98m-2xqg-9cvj
...
```

### Erlang/OTP advisories

This requires version `2.12.0` or later.

Erlang/OTP advisory findings use the reserved dependency name `erlang_otp`. It is not a hex package, so it never appears in `mix.lock`, and no "does not match any dependency" warning is printed for it.

```
erlang_otp                   # ignore every OTP advisory finding
erlang_otp CVE-2026-89422    # ignore one advisory
```

Either the CVE ID or the GHSA ID of an advisory can be used, and matching is case-insensitive. Advisory IDs listed in hex's `:ignore_advisories` config are also applied to OTP findings.

### Elixir advisories

This requires version `2.13.0` or later.

Elixir advisory findings use the reserved dependency name `elixir`, and follow the same rules as `erlang_otp` above.

```
elixir                       # ignore every Elixir advisory finding
elixir CVE-2026-109821       # ignore one advisory
```

### Invalid lines

A line that does not match one of the two formats is skipped and a warning is printed. Common mistakes:

```
postgrex 0.19.3 CVE-2026-32687       <- version numbers are not allowed
postgrex 0.19.3 - EEF-CVE-2026-32687 <- version numbers and dashes are not allowed
EEF-CVE-2026-32687                   <- missing the dependency name
CVE-2026-32687                       <- missing the dependency name
```

Each invalid line produces:

```
[warning] [Paraxial] .paraxial-ignore-deps invalid line, this will NOT be ignored: "postgrex 0.19.3 CVE-2026-32687"
[warning] [Paraxial] Correct formats: "dep_name" (ignore entire dependency) or "dep_name CVE-2026-12345" (ignore one vulnerability)
```

You will also see a warning if a rule names a dependency that is not in your project's `mix.lock`, which usually means a typo:

```
[warning] [Paraxial] .paraxial-ignore-deps: hackny does not match any dependency in mix.lock
```



## Erlang/OTP Advisories

This requires version `2.12.0` or later.

`mix deps.audit` and `mix hex.audit` only check your hex dependencies. Erlang/OTP itself ships security fixes in patch releases — CVE-2025-32433, the SSH pre-authentication RCE, is one example — and those fixes never show up in `mix.lock`. `mix paraxial.scan` checks the Erlang/OTP version it is running on against the [erlang/otp GitHub advisory database](https://github.com/erlang/otp/security/advisories).

Findings have the source `otp_advisory`:

```
[Paraxial] Erlang/OTP 28.5, checked against 70 advisories (server data 2026-10-10T13:42:26Z)
[Paraxial] Scan resulted in 1 findings
[Paraxial] otp_advisory
  affected_applications: ssl
  cve_id: CVE-2026-89422
  dependency: erlang_otp
  first_patched_version: 28.5.0.7
  ghsa_id: GHSA-rgxr-4g4w-j875
  severity: critical
  summary: TLS 1.3 Client Skips Server Authentication on Unsolicited PSK
  title: Erlang/OTP CVE-2026-89422
  url: https://github.com/erlang/otp/security/advisories/GHSA-rgxr-4g4w-j875
  version: 28.5
```

When a branch is no longer maintained and never received a fix, `first_patched_version` names the lowest version on a newer branch instead, for example `None for OTP 26, upgrade to 27.3.4.18 or later`.

### Where the advisory data comes from

The agent ships a snapshot of the advisory database, and Paraxial.io keeps a copy that is refreshed continuously. A scan uses the server's copy when it can reach Paraxial.io, and the bundled snapshot otherwise, so a scan still checks OTP with no network access. The source is named in the output:

```
(server data 2026-10-10T13:42:26Z)    <- fetched from Paraxial.io
(bundled data 2026-10-09)             <- the snapshot in your paraxial dep
```

The request is made before the scanners start and is abandoned if it has not finished by the time they are done, so it does not slow a scan down. If the bundled snapshot is more than 90 days old and was the data used, the scan suggests upgrading the `paraxial` dep.

### Where the version comes from

`System.otp_release()` reports only the major version (`28`), which cannot be matched against patch-level advisories. The full version (`28.5`) is read from the `OTP_VERSION` file of the Erlang installation that is **running the scan**, which is the documented way to get it.

**Run `mix paraxial.scan` in the same image or build stage that compiles your release.** In a normal multi-stage Docker build the builder stage copies its own ERTS into the release, so the version checked is the version deployed. If a CI job pins a different Erlang version than your Dockerfile, the scan reports the CI one.

If the `OTP_VERSION` file is absent the check is skipped with a notice and no findings are produced:

```
[info] [Paraxial] OTP_VERSION file not found, Erlang/OTP advisory check skipped
```

This happens in target systems built by OTP tools and inside a `mix release`, neither of which contains the file. It is why this check runs at scan time rather than at runtime.

### Advisory data freshness

There are two sources of advisory data, and which one applies depends on where you are reading the result.

**Paraxial.io re-checks your OTP version against current data.** The scan sends the OTP version it found along with its findings, and the server matches it against advisories it refreshes from GitHub every few hours. So the findings in your dashboard, and in GitHub or GitLab pull request comments, reflect advisories published today — not whenever your `paraxial` version was released. Nothing to configure, and no new network destination: this rides on the upload the scan already does.

**The scan's own output uses the snapshot bundled in the hex package.** The findings printed in your terminal, and the exit code from `--add-exit-code`, are computed locally before the upload happens. That snapshot makes the check work with no network call, no API key, and in air-gapped CI — but it is only as current as the agent version you have installed. Every scan prints the date it was synced, and after 90 days it also prints:

```
[info] [Paraxial] Erlang/OTP advisory data is 94 days old, upgrade the paraxial dep or check connectivity to paraxial.io for current advisories
```

**So if you gate CI on `--add-exit-code`, keep the `paraxial` dependency current.** A newly published advisory shows up in Paraxial.io immediately but will not fail your build until you upgrade the agent.

Your `.paraxial-ignore-deps` rules apply to both. The OTP entries are sent with the scan so the server applies the same filtering, and an advisory you chose to ignore is not reintroduced by the fresher data.

### Ignoring findings

Add `erlang_otp` to `.paraxial-ignore-deps` to ignore every OTP advisory finding, or `erlang_otp` plus an advisory ID to ignore one. See the `Ignore Dependencies` section above for the matching rules.

```
% cat .paraxial-ignore-deps
erlang_otp
```

The check still runs and still prints its summary line, so an upgrade that resolves the advisories is visible in the scan output rather than silently ignored.

### Scope

Only Erlang/OTP advisories are checked. OpenSSL vulnerabilities are out of scope: `crypto` links against the system OpenSSL at runtime, so the OTP version says nothing about which OpenSSL you are running.

Advisories list the OTP applications they affect (`ssl`, `ssh`, `inets`, ...) as context in `affected_applications`. Matching is always done on the OTP version, never on individual application versions, because a single advisory can patch the same major version of an application on several OTP branches.

## Elixir Advisories

This requires version `2.13.0` or later.

Like Erlang/OTP, Elixir itself ships security fixes in patch releases that never show up in `mix.lock`. `mix paraxial.scan` checks the Elixir version it is running on (`System.version()`) against the [elixir-lang/elixir GitHub advisory database](https://github.com/elixir-lang/elixir/security/advisories). This covers the applications that ship with Elixir, such as `mix`, which are listed in `affected_applications`.

Findings have the source `elixir_advisory`:

```
[Paraxial] Elixir 1.20.1, checked against 5 advisories (server data 2026-11-02T13:42:26Z)
[Paraxial] elixir_advisory
  affected_applications: elixir
  cve_id: CVE-2026-109821
  dependency: elixir
  first_patched_version: 1.20.5
  ghsa_id: GHSA-327h-97cv-59xg
  severity: medium
  summary: Unbounded port parsing in URI.parse/1 enables CPU exhaustion denial of service
  title: Elixir CVE-2026-109821
  url: https://github.com/elixir-lang/elixir/security/advisories/GHSA-327h-97cv-59xg
  version: 1.20.1
```

`first_patched_version` is the fix on your Elixir branch. If your branch is no longer maintained, it is the lowest fixed version on a newer branch.

Advisory data works the same way as for Erlang/OTP: the scan uses Paraxial.io's copy, refreshed every few hours, when it can reach it, and the snapshot bundled in the `paraxial` hex package otherwise. The scan also sends the Elixir version and your `elixir` ignore rules, so the findings in your dashboard and pull request comments are matched against current data. See "Where the advisory data comes from" and "Advisory data freshness" above. As with OTP, run the scan with the same Elixir version that builds your release.

Add `elixir` to `.paraxial-ignore-deps` to ignore every Elixir advisory finding, or `elixir` plus an advisory ID to ignore one.

## Flags

Command line flags for `mix paraxial.scan`:

`--add-exit-code` - Return unix exit code 1 if a scan has findings, 0 if no findings. Useful in CI/CD pipelines to fail when findings are detected. Also returns 1 if an error occurs. 

`--sobelow-config` - If you would like to use a `.sobelow-conf` file, this flag is required to read it. By default, the file is ignored. 

`--sobelow-skip` - Required to skip Sobelow findings. Note that a `.sobelow-conf` file being used overrides this setting.

`--gpl-check` - Create a vulnerability if a dependency using a GPL license is found.

`--no-license-scan` - Do not perform the license scan that occurs automatically. 

`--paraxial_url` - Optional, Paraxial url as a cli flag instead of the config value. 

`--paraxial_api_key` - Optional, Paraxial API key as a cli flag instead of the config value. 

`--sarif` - Output findings in SARIF format. 

`--github_app` - Used to create PR comments in GitHub. See the `GitHub App` page for full details.

`--install_id` - GitHub App, the install ID in your GitHub account

`--repo_owner` - GitHub App, repo owner name

`--repo_name` - GitHub App, repo name

`--pr_number` - GitHub App, PR number

`--quiet-no-issues` - GitHub App, a GitHub comment is no longer created when a scan returns 0 findings. 

## Umbrella Applications

To use `mix paraxial.scan` with your Umbrella application, you must update the `aliases` functions in the top level `mix.exs` file to include:

```elixir
defp aliases do
  [
    sobelow: ["cmd mix sobelow"]
  ]
end
```

This is to run Sobelow against all child applications. 
