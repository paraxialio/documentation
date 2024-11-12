# Paraxial.io Ruby Agent Changelog 

The Paraxial.io Ruby Agent is hosted on RubyGems - [https://rubygems.org/gems/paraxial](https://rubygems.org/gems/paraxial)

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
