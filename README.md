<p align="center">
  <a href="https://nimblehq.co/">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="https://assets.nimblehq.co/logo/dark/logo-dark-text-320.png">
      <img alt="Nimble logo" src="https://assets.nimblehq.co/logo/light/logo-light-text-320.png">
    </picture>
  </a>
</p>

<h3 align="center">Mulesoft GitHub Actions</h3>
<p>A collection of <a href="#mulesoft-actions">Mulesoft Actions</a> and pre-defined <a href="#mulesoft-shared-workflows">Mulesoft Shared Workflows</a> that speed up the development process of Mulesoft projects. The pre-defined workflows are a ready-to-use solution which combines the Actions for running task like MUnit tests or deploying Mulesoft projects.</p>

## Mulesoft Actions

### Set up Mulesoft environment

Action to set up the Mulesoft environment. See [setup/action.yml](setup/action.yml)

#### Usage

```yml
- uses: nimblehq/mulesoft-actions/setup@v1
  with:
    # Version of Java to use
    # Default: 8
    java_version: 8

    # Distribution of Java to use
    # Default: zulu
    java_distribution: zulu
```

Basic:

```yml
name: My workflow
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Mulesoft environment
        uses: nimblehq/mulesoft-actions/setup@v1
```

### Run MUnit tests

Action to run MUnit tests. See [test/action.yml](test/action.yml)

#### Usage

> [!IMPORTANT]\
> The Nexus enterprise repository username and password are required to run MUnit tests on the CI server. Refer to this [document](https://docs.mulesoft.com/mule-runtime/4.4/maven-reference#configure-mule-repositories) for more information.

```yml
- uses: nimblehq/mulesoft-actions/test@v1.2
  with:
    # Nexus username
    # Required
    nexus_username: ${{ secrets.NEXUS_USERNAME }}

    # Nexus password
    # Required
    nexus_password: ${{ secrets.NEXUS_PASSWORD }}

    # Maven settings file path
    # Required
    # Default: .maven/settings.xml
    maven_settings_path: .maven/settings.xml

    # Upload MUnit reports to GitHub Actions Artifacts
    # Default: false
    upload_coverage_reports: false

    # Artifact retention days
    # Default: 1
    retention_days: 1

    # Encryption Key for secure properties
    # Required
    encryption_key: ${{ secrets.ENCRYPTION_KEY }}
```

Basic:

```yml
name: My workflow
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Mulesoft environment
        uses: nimblehq/mulesoft-actions/setup@v1

      - name: Run MUnit tests
        uses: nimblehq/mulesoft-actions/test@v1.2
        with:
          nexus_username: ${{ secrets.NEXUS_USERNAME }}
          nexus_password: ${{ secrets.NEXUS_PASSWORD }}
          maven_settings_path: .maven/settings.xml
          encryption_key: ${{ secrets.ENCRYPTION_KEY }}
```

### Build with Maven

Action to build with Maven. See [build/action.yml](build/action.yml)

#### Usage

```yml
- uses: nimblehq/mulesoft-actions/build@v1
  with:
    # Upload build artifacts to GitHub Actions Artifacts
    # Default: true
    use_artifacts: true

    # Artifact name
    # Default: The name of the build and commit hash
    artifact_name: build-artifacts
```

Basic:

```yml
name: My workflow
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Mulesoft environment
        uses: nimblehq/mulesoft-actions/setup@v1

      - name: Build with Maven
        uses: nimblehq/mulesoft-actions/build@v1
        with:
          artifact_name: build-artifacts
```

### Deploy to CloudHub 1.0

Action to deploy to CloudHub 1.0. See [deploy_cloudhub_1_0/action.yml](deploy_cloudhub_1_0/action.yml)

> [!IMPORTANT]\
> This action requires a built artifact from the `build` action.
> Before deploying to CloudHub, you must configure the `cloudHubDeployment` element. Inside the `org.mule.tools.maven` plugin element in the project’s `pom.xml` file, add the following configuration:

```xml
<plugin>
    <groupId>org.mule.tools.maven</groupId>
    <artifactId>mule-maven-plugin</artifactId>
    <version>${mule.maven.plugin.version}</version>
    <extensions>true</extensions>
    <configuration>
        <!-- Add the following configuration -->
        <cloudHubDeployment>
          <uri>https://anypoint.mulesoft.com</uri>
          <muleVersion>${app.runtime}</muleVersion>
          <applicationName>${CLOUDHUB_APPLICATION_NAME}</applicationName>
          <environment>${CLOUDHUB_ENVIRONMENT}</environment>
          <businessGroupId>${CLOUDHUB_BUSINESS_GROUP_ID}</businessGroupId>
          <region>${CLOUDHUB_REGION}</region>
          <connectedAppClientId>${CONNECTED_APP_CLIENT_ID}</connectedAppClientId>
          <connectedAppClientSecret>${CONNECTED_APP_CLIENT_SECRET}</connectedAppClientSecret>
          <connectedAppGrantType>client_credentials</connectedAppGrantType>
          <overrideProperties>false</overrideProperties>
          <properties>
            <mule.env>${MULE_ENVIRONMENT}</mule.env>
            <securedKey>${encryptionKey}</securedKey>
          </properties>
        </cloudHubDeployment>
        <!-- End of configuration -->
    </configuration>
</plugin>
```

#### Usage

```yml
- uses: nimblehq/mulesoft-actions/deploy_cloudhub_1_0@v1.2
  with:
    # Use artifact from `build` action
    # Default: false
    use_artifact: true

    # Artifact name
    # Default: build-artifacts
    artifact_name: build-artifacts

    # Mule application artifact name
    # Required
    mule_artifact_name: ${{ steps.build.outputs.mule_artifact_name }}

    # CloudHub connected app client ID
    # Required
    connected_app_client_id: ${{ secrets.CONNECTED_APP_CLIENT_ID }}

    # CloudHub connected app client secret
    # Required
    connected_app_client_secret: ${{ secrets.CONNECTED_APP_CLIENT_SECRET }}

    # CloudHub environment
    # Required
    cloudhub_environment: ${{ secrets.CLOUDHUB_ENVIRONMENT }}

    # CloudHub Business Group ID
    # Required
    cloudhub_business_group_id: ${{ secrets.CLOUDHUB_BUSINESS_GROUP_ID }}

    # CloudHub Region
    # Required
    cloudhub_region: ${{ secrets.CLOUDHUB_REGION }}

    # CloudHub application name
    # Required
    cloudhub_application_name: ${{ secrets.CLOUDHUB_APPLICATION_NAME }}

    # Mule runtime version
    # Default: 4.4.0
    mule_runtime_version: 4.4.0

    # Mule environment
    # Default: production
    mule_environment: production

    # Encryption Key for secure properties
    # Required
    encryption_key: ${{ secrets.ENCRYPTION_KEY }}
```

Basic:

```yml
name: My workflow
on: [push, pull_request]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Mulesoft environment
        uses: nimblehq/mulesoft-actions/setup@v1

      - name: Build with Maven
        uses: nimblehq/mulesoft-actions/build@v1
        with:
          use_artifacts: false
        id: build

      - name: Deploy to CloudHub
        uses: nimblehq/mulesoft-actions/deploy_cloudhub_1_0@v1.2
        with:
          cloudhub_environment: ${{ secrets.CLOUDHUB_ENVIRONMENT }}
          cloudhub_business_group_id: ${{ secrets.CLOUDHUB_BUSINESS_GROUP_ID }}
          cloudhub_region: ${{ secrets.CLOUDHUB_REGION }}
          mule_runtime_version: ${{ secrets.MULE_VERSION }}
          mule_environment: ${{ secrets.MULE_ENVIRONMENT }}
          application_name: ${{ secrets.APPLICATION_NAME }}
          connected_app_client_id: ${{ secrets.CONNECTED_APP_CLIENT_ID }}
          connected_app_client_secret: ${{ secrets.CONNECTED_APP_CLIENT_SECRET }
          use_artifact: false
          mule_file_path: ${{ steps.build.outputs.mule_file_path }}
          encryption_key: ${{ secrets.ENCRYPTION_KEY }}
```

### Publish Assets to Anypoint Exchange

Action to Publish Assets to Anypoint Exchange. See [publish_assets/action.yml](publish_assets/action.yml)

#### Usage

```yml
- uses: nimblehq/mulesoft-actions/publish_assets@v1
  with:
    # AnyPoint Organization ID or Business Group ID
    # Required
    org_id: ${{ secrets.BUSINESS_GROUP_ID }}

    # Connected App Client ID
    # Required
    connected_app_client_id: ${{ secrets.CONTD_APP_CLIENT_ID }}

    # Connected App Client Secret
    # Required
    connected_app_client_secret: ${{ secrets.CONTD_APP_CLIENT_SECRET }}
```

Basic:

```yml
name: My workflow
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Publish Assets to Anypoint Exchange
        uses: nimblehq/mulesoft-actions/publish_assets@v1
        with:
          org_id: ${{ secrets.BUSINESS_GROUP_ID }}
          connected_app_client_id: ${{ secrets.CONTD_APP_CLIENT_ID }}
          connected_app_client_secret: ${{ secrets.CONTD_APP_CLIENT_SECRET }}
```

### Bump version for the documentation

Action to bump version for the documentation. See [bump_version_doc/action.yml](bump_version_doc/action.yml)

#### Usage

```yml
- uses: nimblehq/mulesoft-actions/bump_version_doc@v1
  with:
    # New version to bump
    # Required
    new_version: 1.0.1

    # Version file
    version_file: exchange.json

    # Committer for the pull request
    # Required
    committer: ${{ github.actor }}

    # Assignees for the pull request
    assignees: ${{ github.actor }}

    # GitHub token for the pull request
    # Required
    github_token: ${{ github.token }}
```

Basic:

```yml
name: My workflow
on: [push, pull_request]
jobs:
  bump_version:
    name: Bump Version
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - name: Bump Version
        uses: nimblehq/mulesoft-actions/bump_version_doc@v1
        with:
          new_version: ${{ inputs.new_version }}
          assignees: nimble-bot
          committer: Nimble Bot <bot@nimblehq.co>
          github_token: ${{ github.token }}

```

### Bump version for the Mule app

Action to bump version for the Mule app. See [bump_version_app/action.yml](bump_version_app/action.yml)

#### Usage

```yml
- uses: nimblehq/mulesoft-actions/bump_version_app@v1
  with:
    # New version to bump
    # Required
    new_version: 1.0.1

    # Committer for the pull request
    # Required
    committer: ${{ github.actor }}

    # Assignees for the pull request
    assignees: ${{ github.actor }}

    # GitHub token for the pull request
    # Required
    github_token: ${{ github.token }}
```

Basic:

```yml
name: My workflow
on: [push, pull_request]
jobs:
  bump_version:
    name: Bump Version
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - name: Bump Version
        uses: nimblehq/mulesoft-actions/bump_version_app@v1
        with:
          new_version: ${{ inputs.new_version }}
          assignees: nimble-bot
          committer: Nimble Bot <bot@nimblehq.co>
          github_token: ${{ github.token }}

```

## Mulesoft Shared Workflows

### Shared Test Workflow

Workflow to run MUnit tests for Mulesoft projects. See [.github/workflows/shared_test.yml](.github/workflows/shared_test.yml)

#### Usage

```yml
- uses: nimblehq/mulesoft-actions/.github/workflows/shared_test.yml@v1.2
  with:
    # Maven settings file path
    # Required
    # Default: .maven/settings.xml
    maven_settings_path: .maven/settings.xml

    # Upload MUnit reports to GitHub Actions Artifacts
    # Default: true
    upload_coverage_reports: true

  secrets:
    # Nexus username
    # Required
    NEXUS_USERNAME: ${{ secrets.NEXUS_USERNAME }}

    # Nexus password
    # Required
    NEXUS_PASSWORD: ${{ secrets.NEXUS_PASSWORD }}

    # Encryption Key for secure properties
    # Required
    ENCRYPTION_KEY: ${{ secrets.ENCRYPTION_KEY }}
```

Basic:

```yml
name: My workflow
on: [push, pull_request]
jobs:
  trigger_test:
    uses: nimblehq/mulesoft-actions/.github/workflows/shared_test.yml@v1.2
    name: Trigger the test workflow
    with:
      upload_coverage_reports: true
    secrets:
      NEXUS_USERNAME: ${{ secrets.NEXUS_USERNAME }}
      NEXUS_PASSWORD: ${{ secrets.NEXUS_PASSWORD }}
      ENCRYPTION_KEY: ${{ secrets.ENCRYPTION_KEY }}
```

### Shared Deploy Workflow

Workflow to deploy Mulesoft projects to CloudHub 1.0. See [.github/workflows/shared_deploy_cloudhub_1_0.yml](.github/workflows/shared_deploy_cloudhub_1_0.yml)

#### Usage

Create a new environment for deployment and set the needed environment variables, secrets.

```yml
- uses: nimblehq/mulesoft-actions/.github/workflows/shared_deploy_cloudhub_1_0.yml@v1.2
  with:
    # CloudHub application name
    # Required
    cloudhub_application_name: my-app

    # CloudHub region
    # Required
    cloudhub_region: ap-southeast-1

    # CloudHub environment
    # Required
    cloudhub_environment: DEV

    # Mule runtime version
    # Default: 4.4.0
    mule_runtime_version: 4.4.0

  secrets:
    # CloudHub connected app client ID
    # Required
    CONTD_APP_CLIENT_ID: ${{ secrets.CONNECTED_APP_CLIENT_ID }}

    # CloudHub connected app client secret
    # Required
    CONTD_APP_CLIENT_SECRET: ${{ secrets.CONNECTED_APP_CLIENT_SECRET }}

    # CloudHub business group ID
    # Required
    CLOUDHUB_BUSINESS_GROUP_ID: ${{ secrets.CLOUDHUB_BUSINESS_GROUP_ID }}

    # Encryption Key for secure properties
    # Required
    ENCRYPTION_KEY: ${{ secrets.ENCRYPTION_KEY }}
```

Basic:

```yml
name: My workflow
on: [push]
jobs:
  trigger_deploy:
    uses: nimblehq/mulesoft-actions/.github/workflows/shared_deploy_cloudhub_1_0.yml@v1.2
    name: Trigger the deploy workflow
    with:
      cloudhub_application_name: my-app
      cloudhub_region: southeast-1
      cloudhub_environment: DEV
    secrets:
      CONTD_APP_CLIENT_ID: ${{ secrets.CONTD_APP_CLIENT_ID }}
      CONTD_APP_CLIENT_SECRET: ${{ secrets.CONTD_APP_CLIENT_SECRET }}
      CLOUDHUB_BUSINESS_GROUP_ID: ${{ secrets.BUSINESS_GROUP_ID }}
      ENCRYPTION_KEY: ${{ secrets.ENCRYPTION_KEY }}
```

## License

This project is Copyright (c) 2014 and onwards Nimble. It is free software and may be redistributed under the terms specified in the [LICENSE] file.

[LICENSE]: /LICENSE

## About

<a href="https://nimblehq.co/">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://assets.nimblehq.co/logo/dark/logo-dark-text-160.png">
    <img alt="Nimble logo" src="https://assets.nimblehq.co/logo/light/logo-light-text-160.png">
  </picture>
</a>

This project is maintained and funded by Nimble.

We ❤️ open source and do our part in sharing our work with the community!
See [our other projects][community] or [hire our team][hire] to help build your product.

Want to join? [Check out our Jobs][jobs]!

[community]: https://github.com/nimblehq
[hire]: https://nimblehq.co/
[jobs]: https://jobs.nimblehq.co/
