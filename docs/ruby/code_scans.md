# Code Scans

## Ignoring RuboCop Findings

Paraxial.io uses RuboCop for static analysis. If a finding is a false positive, there are two ways to flag this:

1. Via a code comment
2. In the `.paraxial-rubocop.yml` file, which follows the normal [RuboCop configuration settings.](https://docs.rubocop.org/rubocop/configuration.html)

```
# rubocop:disable Paraxial/SQL
def show
  User.find_by_sql("select * from users WHERE id = #{params[:id]} ")
end 
```

If the function has multiple findings, separate the cop name with a comma:

```
# rubocop:disable Paraxial/SQL, Paraxial/System
def show
  system(params[:id])
  User.find_by_sql("select * from users WHERE id = #{params[:id]} ")
end 
```

Some cops are built into RuboCop, and others and written by Paraxial.io. The full list: 

- Paraxial/Constantize
- Paraxial/CSRF
- Paraxial/HTMLSafe
- Paraxial/Raw
- Paraxial/Send
- Paraxial/SQL
- Paraxial/System
- Security/Eval
- Security/IoMethods
- Security/JSONLoad
- Security/MarshalLoad
- Security/Open
- Security/YAMLLoad

If you want to disable a cop on a specific file, or disable it across the entire project, `.paraxial-rubocop.yml` is the best way to do that. 

`.paraxial-rubocop.yml`

```
require:
  - rubocop-erb

Paraxial/Constantize:
  Enabled: true
  Exclude:
    - 'app/controllers/users_controller.rb'

Paraxial/HTMLSafe:
  Enabled: true
  Exclude:
    - 'app/views/static_pages/home.html.erb'

Paraxial/Send:
  Enabled: false
```


## Ignoring Vulnerable Gems

To ignore a vulnerable Gem in the scan results use the `.paraxial.yml` file:

`.paraxial.yml`

```
ignore-gems:
- bootstrap-sass
- puma             

```
