# GitLab App

## 1. Prerequisites 

Before starting with the GitLab App, ensure that Paraxial.io is setup. For example, you should be able to run:

`mix paraxial.scan`

And view the results of this scan in the web interface. If you have not completed this setup, go the [getting started guide](./start.md) and complete that first before continuing. 

If you can see recent events for "Vulnerabilities" in your site's overview page, then continue:

![gitlab_app](./assets/gl0.png)

## 2. GitLab API token 


You will need to create a GitLab access token. This may be:

- A personal access token
- A project access token (requires Maintainer role)
- A group access token (requires Maintainer role)

The access token you create must have the `api` and `write_repository` scopes. If you are creating a project or group token, ensure the role is Maintainer, or else the token may not work. It should have the format: `glpat-Yhg[redacted]`

It can be frustrating to create a token and place it in CI/CD, only to learn that the incorrect scope or role was set. To ensure the token is working before continuing, run the following command:

```
curl --request POST "https://gitlab.com/api/v4/projects/YOUR_PROJECT_ID/merge_requests/YOUR_MERGE_REQUEST/notes" \
     --header "PRIVATE-TOKEN: glpat-YOUR_VALUE_HERE" \
     --form "body=Token permission test."
```

You must plug in your own values for `YOUR_PROJECT_ID`, `YOUR_MERGE_REQUEST`, and `glpat-YOUR_VALUE_HERE`. If you are self-hosting GitLab, the URL will be different. If the above command results in a comment being created, your token is valid. 

Also note that GitLab requires a Paraxial.io version of `2.7.8` or higher. Run the following to test if your install is working locally:

```
mix paraxial.scan --gitlab_app --gitlab_token $GITLAB_KEY --gitlab_project $CI_PROJECT_ID --merge_request $CI_MERGE_REQUEST_IID 
```

Ensure you have an open merge request and replace `$GITLAB_KEY, $CI_PROJECT_ID, and $CI_MERGE_REQUEST_IID` with the values for your current project. 

If you are using self-hosted GitLab add the `--gitlab_url` flag:

```
mix paraxial.scan --gitlab_app --gitlab_token $GITLAB_KEY --gitlab_project $CI_PROJECT_ID --merge_request $CI_MERGE_REQUEST_IID --gitlab_url https://gitlab.example.com
```

Replace `https://gitlab.example.com` with your instance URL. If a comment is created by Paraxial.io, you are using the correct version and can continue. 

Now to run Paraxial.io in CI/CD. For the current project, go to:

**Settings > CI/CD > Variables**

And set the following:

- `PARAXIAL_API_KEY` (found in app.paraxial.io, under your site's settings)
- `GITLAB_KEY` (the token you just created)

![gitlab_app](./assets/gl1.png)

## 3. GitLab CI/CD

Create the following in your project root directory:

`.gitlab-ci.yml`

```
# The workflow must have a merge request for Paraxial.io to comment on
workflow:
  rules:
    - if: $CI_MERGE_REQUEST_IID

image: elixir:latest

services:
  - postgres:latest
variables:
  POSTGRES_DB: carafe
  POSTGRES_HOST: postgres
  POSTGRES_USER: postgres
  POSTGRES_PASSWORD: "postgres"
  MIX_ENV: "test"

before_script:
  - mix local.rebar --force
  - mix local.hex --force
  - mix deps.get

mix:
  script:
    - mix test
    - mix paraxial.scan --gitlab_app --gitlab_token $GITLAB_KEY --gitlab_project $CI_PROJECT_ID --merge_request $CI_MERGE_REQUEST_IID  --sobelow-config --add-exit-code

deploy:
  stage: deploy
  script: echo "Define your deployment script!"
  environment: production
```

Note the arguments:

- `--gitlab_app`     - required to send scan results to GitLab
- `--gitlab_token`   - the personal, project, or group access token
- `--gitlab_project` - the GitLab project ID
- `--merge_request`  - merge request IID 

The following optional arguments are included:

- `--sobelow-config` - use the `.sobelow-conf` file in the project
- `--add-exit-code`  - return non-zero exit code if findings exist, to fail CI/CD 

If you are using the self hosted version of GitLab, add the flag:

`--gitlab_url https://gitlab.example.com`

For example:

`mix paraxial.scan --sobelow-config --gitlab_app --gitlab_token $GITLAB_KEY --gitlab_project $CI_PROJECT_ID --merge_request $CI_MERGE_REQUEST_IID --gitlab_url https://gitlab.paraxial.io --add-exit-code`

If you are using the cloud version of GitLab, this flag is not needed. 

Now open a merge request and observe the findings from Paraxial.io:

<img src="../assets/gl2.png" alt="gl" width="600"/>

If no issues are found, you will get a green check comment:

<img src="../assets/gl3.png" alt="gl" width="600"/>
