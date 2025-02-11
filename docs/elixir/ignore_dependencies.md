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