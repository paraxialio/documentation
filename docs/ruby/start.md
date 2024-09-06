# Getting started with Paraxial.io (Ruby)

This tutorial is a step-by-step guide to setup Paraxial.io with a Ruby on Rails application. It will walk through creating an account, installing the agent, and getting results flowing to the backend. Functionality is limited in the free tier, for example you cannot send HTTP traffic to the Paraxial.io backend for analysis.

## Free Tier Limits

1. Maximum of 2 Sites.
2. You cannot invite users to a Site.
3. HTTP traffic cannot be ingested by the backend.
4. Limit of 30 scans per site, per month. 

If you have questions about Paraxial, need enterprise support, or would like to upgrade your plan, email `support@paraxial.io`.

## 1. Create your Paraxial.io account 

Go to [https://app.paraxial.io/](https://app.paraxial.io/) in your web browser. Create a new account. You will receive a confirmation email, use it to confirm your account and sign in. You have no sites at this point. 

## 2. Create your site

Create a new site, but pick a different domain besides `sample_app`. Note that the `domain` is really treated as a comment by Paraxial.io, you can put any value you want here, it does not have to be a valid URL and no HTTP requests are ever sent to it.

![start0](./assets/start0.png)

![start1](./assets/start1.png)

## 3. Sample App install

This section is to setup the reference sample app for the Ruby on Rails Tutorial (7th edition) by Michael Hartl. If you are installing Paraxial.io in your own project you can skip this section.

GitHub - [https://github.com/learnenough/rails_tutorial_sample_app_7th_ed](https://github.com/learnenough/rails_tutorial_sample_app_7th_ed)


```
% git clone https://github.com/learnenough/rails_tutorial_sample_app_7th_ed 
% cd rails_tutorial_sample_app_7th_ed/

Gemfile
% gem "rails",                      "7.0.4" # bump to 7.0.5 for Apple M1 CPU
% rm -rf Gemfile.lock 

% bundle 
% rails db:migrate
% rails db:seed
% rails server
```

## 4. Install Paraxial.io

`Gemfile`
```
...
gem "puma",                       "5.6.4"
gem "bootsnap",                   "1.12.0", require: false
gem "paraxial",                   "0.6.0"

group :development, :test do
...
```

Find your API key in "Site Settings". 

export PARAXIAL_API_KEY=your_key_here

Create `.rubocop.yml` and add:

```
% cat .rubocop.yml 
require:
- rubocop-erb
```

This is so Paraxial.io can scan ERB files for security issues.

Start your application:

`rails s`

Then check your site page:

![start2](./assets/start2.png)

You should see data for libraries and open source licenses. 

Now run a scan:

```
% paraxial scan
[Paraxial] Scan starting...
[Paraxial] .rubocop.yml is valid.

[Paraxial] Scan count: 12

(Gemfile.lock) actionpack: Possible XSS via User Supplied Values to redirect_to - 7.0.5
- Title: actionpack: Possible XSS via User Supplied Values to redirect_to
- Installed Version: 7.0.5
- Fixed Version: ~> 6.1.7.4, >= 7.0.5.1

(Gemfile.lock) rubygem-actionpack: Possible XSS on translation helpers - 7.0.5
- Title: rubygem-actionpack: Possible XSS on translation helpers
- Installed Version: 7.0.5
- Fixed Version: ~> 7.0.8, >= 7.0.8.1, >= 7.1.3.1

(Gemfile.lock) rubygem-actionpack: Missing security headers in Action Pack on non-HTML responses - 7.0.5
- Title: rubygem-actionpack: Missing security headers in Action Pack on non-HTML responses
- Installed Version: 7.0.5
- Fixed Version: ~> 6.1.7, >= 6.1.7.8, ~> 7.0.8, >= 7.0.8.4, ~> 7.1.3, >= 7.1.3.4, >= 7.2.0.beta2

(Gemfile.lock) Arbitrary Code Execution Vulnerability in Trix Editor included in ActionText - 7.0.5
- Title: Arbitrary Code Execution Vulnerability in Trix Editor included in ActionText
- Installed Version: 7.0.5
- Fixed Version: ~> 7.0.8.3, >= 7.1.3.3

(Gemfile.lock) rubygem-activestorage: Possible Sensitive Session Information Leak in Active Storage - 7.0.5
- Title: rubygem-activestorage: Possible Sensitive Session Information Leak in Active Storage
- Installed Version: 7.0.5
- Fixed Version: ~> 6.1.7, >= 6.1.7.7, >= 7.0.8.1

(Gemfile.lock) rubygem-activesupport: File Disclosure of Locally Encrypted Files - 7.0.5
- Title: rubygem-activesupport: File Disclosure of Locally Encrypted Files
- Installed Version: 7.0.5
- Fixed Version: ~> 6.1.7, >= 6.1.7.5, >= 7.0.7.1

(Gemfile.lock) Bootstrap Cross-Site Scripting (XSS) vulnerability - 3.4.1
- Title: Bootstrap Cross-Site Scripting (XSS) vulnerability
- Installed Version: 3.4.1
- Fixed Version: 

(Gemfile.lock) rubygem-puma: HTTP request smuggling when parsing chunked transfer encoding bodies and zero-length content-length headers - 5.6.4
- Title: rubygem-puma: HTTP request smuggling when parsing chunked transfer encoding bodies and zero-length content-length headers
- Installed Version: 5.6.4
- Fixed Version: ~> 5.6.7, >= 6.3.1

(Gemfile.lock) rubygem-puma: HTTP request smuggling when parsing chunked Transfer-Encoding Bodies - 5.6.4
- Title: rubygem-puma: HTTP request smuggling when parsing chunked Transfer-Encoding Bodies
- Installed Version: 5.6.4
- Fixed Version: ~> 5.6.8, >= 6.4.2

(Gemfile.lock) rubygem-actionpack: Possible XSS on translation helpers - 7.0.5
- Title: rubygem-actionpack: Possible XSS on translation helpers
- Installed Version: 7.0.5
- Fixed Version: 7.0.8.1, 7.1.3.1

(Rubocop) CSRF, no protect_from_forgery in ApplicationController.
- Path: app/controllers/application_controller.rb
- Line: 1

(Rubocop) `send` causes remote code execution if called on user input.
- Path: app/models/user.rb
- Line: 49

[Paraxial] Scan UUID your_value_here
[Paraxial] Scan URL your_url_here
```

![start3](./assets/start3.png)