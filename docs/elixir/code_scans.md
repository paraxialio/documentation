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
