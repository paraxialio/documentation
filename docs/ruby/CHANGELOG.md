# Paraxial.io Ruby Agent Changelog 

The Paraxial.io Ruby Agent is hosted on RubyGems - [https://rubygems.org/gems/paraxial](https://rubygems.org/gems/paraxial)

## `1.4.4`
- If `.paraxial-rubocop.yml` does not exist when running `paraxial scan`, it will now automatically be created and configured to allow Rubocop to scan erb files. 

## `1.4.3`
- Improve cross site request forgery (CSRF) detection in Rubocop.

## `1.4.2`
- Better error handling when user on free tier is rate limited from Code Scans.

## `1.4.1`
- Add the `config.disable_http_tick = true` option to disable HTTP requests when bot defense is not in use. 

## `1.4.0`
- Fixed a bug where `Paraxial.block_cloud_ip` would block an IP on the allow list.
- Now an IP on the allow list will always be able to make a connection. For example, you have a VPN hosted in AWS and always want to allow traffic from it. 

## `1.3.1`
- Improve runtime checks to prevent the agent from starting during [rails commands.](https://guides.rubyonrails.org/command_line.html#command-line-basics)

## `1.3.0`
- Add the `Paraxial::PHPAttackMiddleware`, a pre-defined way to ban malicious clients requesting `.php` routes. 

## `1.2.0`
- Add the `.paraxial.yml` file. Users can define `ignore-gems:` to exclude gems from vulnerability scannings. 

## `1.1.0`
- Configure Paraxial.io RuboCop settings with `.paraxial-rubocop.yml` instead of the old `.rubocop.yml`

## `1.0.2`
- Fix bug in SQL cop for code scans

## `1.0.1`
- Add `--debug-rubocop` flag for `paraxial scan`, runs rubocop in debug mode
