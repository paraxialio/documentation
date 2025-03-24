# Paraxial.io Changelog

The Paraxial.io Elixir Agent is hosted on Hex - [https://hex.pm/packages/paraxial](https://hex.pm/packages/paraxial)

## `2.8.2`
- Add `.paraxial-ignore-deps` file to ignore dependency findings. See [Code Scans](./code_scans.md) for more information.

## `2.8.1`
- Add the `--quiet-no-issues` flag for use with `--github_app`. A GitHub comment is no longer created when a scan returns 0 findings. 

## `2.8.0`
- Add `ban_ip/3` and `Paraxial.PHPAttackPlug`
- Print the Paraxial agent version on startup and when running `mix paraxial.scan`

## `2.7.8`
- Add GitLab app.

## `2.7.7`
- `:paraxial_url` is no longer required in the config file.
- Warn when a `.sobelow_skips` file exists, but is not being read because `--sobelow-skip` was not passed.
- Do not send HTTP events to backend on free tier. 

## `2.7.6`
- Add `Paraxial.check_rate/6` for rate limiting with Paraxial.io reporting. Can be used to rate limit LiveView functions. 

## `2.7.5`
- Send the `conn.host` value to backend.

## `2.7.4`
- Fix a bug where `mix paraxial.scan` with `--add-exit-code` and without `--github_app` would always return 1 (error). It now returns the correct value.

## `2.7.3`
- Add a special check for the xz library, versions `5.6.0` and `5.6.1`, in App Audit. These versions have a high profile backdoor. 
- Fix App Audit error where the agent is started without a config. 

## `2.7.2`

- `mix paraxial.scan --no-license-scan`, the new flag will stop the license scan from running. 
- `mix paraxial.scan --add-exit-code` now returns 1 if an error condition occurs during the scan, for example the scan upload fails due to an invalid API key.
- If an invalid flag is passed to `mix paraxial.scan`, a warning is displayed. For example: `mix paraxial.scan --null` will show `[warning] [Paraxial] --null not a valid flag. Unexpected behavior may occur.`
- Fix `function Mix.Dep.loaded/1 is undefined` for newer versions of Elixir `(>= 1.16.0)`


## `2.7.1`
- `File .sobelow-conf found, but --sobelow-config not set, default scan will run` - this is now a warning, not an error. 

## `2.7.0`

- Changes to `mix paraxial.scan`:
- The `--sobelow-config` flag is required to read `.sobelow-conf`
- The `--sobelow-skip` flag is required to honor Sobelow skips (code comments or a `.sobelow-skips` file). Note that a `.sobelow-conf` files overrides this setting.
- The `--gpl-check` flag will create a vulnerability if a dependency using a GPL license is found.
- Add License Check, which uploads an inventory of dependencies taken at compile time with license info. This may result in different findings than App Audit (runtime) because the inventory is fetched at compile time. 

## `2.6.4`

- Add scan flags: `mix paraxial.scan --paraxial_url https://app.paraxial.io --paraxial_api_key API_KEY_HERE`
- If these flags are set, they will override the config file values.

## `2.6.3`
- Fix bug where `mix paraxial.scan` without `--sarif` flag crashed. 

## `2.6.2`
- Add `--sarif` flag to get enriched finding data.

## `2.6.1`
- The Sobelow scan in `mix paraxial.scan` now has the `--config` flag by default, so it can read `.sobelow-conf`.

## `2.6.0`

- Add the `mix paraxial.scan --github_app` flag, for use with the Paraxial.io Github App
- Additional required arguments: `--install_id`, `--repo_owner`, `--repo_name`, `--pr_number`
- See the Github App page for installation instructions.

## `2.5.2`
- `mix paraxial.scan` prints scan uuid.

## `2.5.1`

- `iptrie` from `~> 0.8.0` to `>= 0.8.0`.
- `sobelow` from `~> 0.12.2` to `>= 0.12.2`.
- Change `warn` to `error` to better reflect error conditions.

## `2.5.0` 

- Add App Audit to agent. 

## `2.4.0` 

- Add Exploit Guard to agent.

## `2.3.4`

- `mix paraxial.scan` now has the `--add-exit-code` flag, returns unix exit code 1 if scan has findings. Returns 0 if no findings. 

## `2.3.3`

- Allow HTTPoison versions `2.0.0` and higher

## `2.3.2`

- Upgrade Sobelow from `0.12.1` to `0.12.2`

## `2.3.1`

- Sobelow `0.12.0` required `castore` ~> 1.0
- Sobelow `0.12.1` relaxes this requirement for backwards compatibility

## `2.3.0`

- Upgrade Sobelow from `0.11.1` to `0.12.0`
- Sobelow now checks for XSS in HEEx templates

## `2.2.0`

- `mix paraxial.scan` now supports umbrella projects. 
- Requires you to add `sobelow: ["cmd mix sobelow"]` in your top-level mix file. https://github.com/nccgroup/sobelow/pull/108/files

## `2.1.0`

- You can now disable the Paraxial.io agent. If there is no configuration set for `:paraxial_api_key` or `:paraxial_url`, the agent will not start, and the Paraxial plugs will do nothing to conn. 
- To disable the agent in your `dev` or `test` environment, ensure there are no values set for your `:paraxial` configuration. If `:paraxial_api_key` and `:paraxial_url` have non-nil values, the agent will start and the Paraxial plugs will function normally. 

## `2.0.0`

- WARNING: Breaking changes to the `only/except` configuration values. Previously these were read via `Application.get_env`, and would not raise an error if runtime and compile time settings were different. 
- `only/except` are now read with `Application.compile_env/3` in `2.0.0`. From the docs, "By using compile_env/3, tools like Mix will store the values used during compilation and compare the compilation values with the runtime values whenever your system starts, raising an error in case they differ."
- There is no change in features from `1.1.0` to `2.0.0`. The reason for this release is to make debugging CI/CD pipelines easier, because `compile_env` will trigger an error if runtime and compile time configuration differs. 
- After changing `only/except` in your dev environment run `mix deps.clean paraxial`. If you don't, you will get an error, `ERROR! the application :paraxial has a different value set for key :except during runtime`.

## `1.1.0`

- Add `mix paraxial.scan`, code scanning for vulnerabilities. 

## `1.0.0`

- If `fetch_cloud_ips` is set to true, and there is no internet connection, `ip_trie` will be set to an empty trie. 
- `PARAXIAL_API_KEY` environment variable support added. 