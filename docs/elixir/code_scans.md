# Code Scans 

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

# Ignore Dependencies

This features requires Paraxial.io Elixir version `2.8.2` or later. 

To ignore a dependency related finding in Paraxial.io code scans:

1. Create the file `.paraxial-ignore-deps` in your project directory
2. Add the name of each dependency you want to ignore on a new line

Example:

```
% cat .paraxial-ignore-deps 
hackney 
jason
remote_ip

```

Sample output:

```
% mix paraxial.scan        

16:53:49.918 [info] [Paraxial] v2.8.2, scan starting
16:53:49.920 [info] [Paraxial] API key found, scan results will be uploaded
16:53:50.840 [info] [Paraxial] Ignoring dependencies listed in .paraxial-ignore-deps
[Paraxial] .paraxial-ignore-deps list: ["hackney", "jason", "remote_ip"]
...
```
