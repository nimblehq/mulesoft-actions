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
- uses: nimblehq/mulesoft-actions/test@v1
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
        uses: nimblehq/mulesoft-actions/test@v1
        with:
          nexus_username: ${{ secrets.NEXUS_USERNAME }}
          nexus_password: ${{ secrets.NEXUS_PASSWORD }}
          maven_settings_path: .maven/settings.xml
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
    # Default: build-artifacts
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

### Deploy to CloudHub

Action to deploy to CloudHub. See [deploy/action.yml](deploy/action.yml)

> [!IMPORTANT]\
> This action requires a built artifact from the `build` action.
Before deploying to CloudHub, you must configure the `cloudHubDeployment` element. Inside the `org.mule.tools.maven` plugin element in the project’s `pom.xml` file, add the following configuration:

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
            <muleVersion>${MULE_VERSION}</muleVersion>
            <applicationName>${CLOUDHUB_APPLICATION_NAME}</applicationName>
            <environment>${CLOUDHUB_ENVIRONMENT}</environment>
            <connectedAppClientId>${CONNECTED_APP_CLIENT_ID}</connectedAppClientId>
            <connectedAppClientSecret>${CONNECTED_APP_CLIENT_SECRET}</connectedAppClientSecret>
            <connectedAppGrantType>client_credentials</connectedAppGrantType>
            <properties>
							<mule.env>${MULE_ENVIRONMENT}</mule.env>
            </properties>
        </cloudHubDeployment>
        <!-- End of configuration -->
    </configuration>
</plugin>
```

#### Usage

```yml
- uses: nimblehq/mulesoft-actions/deploy@v1
  with:
    # Use artifact from `build` action
    # Default: false
    use_artifact: true

    # Artifact name
    # Default: build-artifacts
    artifact_name: build-artifacts

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

    # Mule version
    # Default: 4.4.0
    mule_version: 4.4.0

    # Mule environment
    # Default: production
    mule_environment: production

    # Application name
    # This will be used as the application name in CloudHub along with the CloudHub region and Mule environment
    # Example: my-app-my-region-my-environment
    # Required
    application_name: ${{ secrets.CLOUDHUB_APPLICATION_NAME }}
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
          artifact_name: build-artifacts
        id: build

      - name: Deploy to CloudHub
        uses: nimblehq/mulesoft-actions/deploy@v1
        with:
          cloudhub_environment: ${{ secrets.CLOUDHUB_ENVIRONMENT }}
          cloudhub_business_group_id: ${{ secrets.CLOUDHUB_BUSINESS_GROUP_ID }}
          cloudhub_region: ${{ secrets.CLOUDHUB_REGION }}
          mule_version: ${{ secrets.MULE_VERSION }}
          mule_environment: ${{ secrets.MULE_ENVIRONMENT }}
          application_name: ${{ secrets.APPLICATION_NAME }}
          connected_app_client_id: ${{ secrets.CONNECTED_APP_CLIENT_ID }}
          connected_app_client_secret: ${{ secrets.CONNECTED_APP_CLIENT_SECRET }}
```

### Publish Assets to Anypoint Exchange

Action to Publish Assets to Anypoint Exchange. See [publish_assets/action.yml](publish_assets/action.yml)

#### Usage

```yml
- uses: nimblehq/mulesoft-actions/publish_assets@main
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
        uses: nimblehq/mulesoft-actions/publish_assets@main
        with:
          org_id: ${{ secrets.BUSINESS_GROUP_ID }}
          connected_app_client_id: ${{ secrets.CONTD_APP_CLIENT_ID }}
          connected_app_client_secret: ${{ secrets.CONTD_APP_CLIENT_SECRET }}
```

## Mulesoft Shared Workflows

### Shared Test Workflow

Workflow to run MUnit tests for Mulesoft projects. See [.github/workflows/shared_test.yml](.github/workflows/shared_test.yml)

#### Usage

```yml
- uses: nimblehq/mulesoft-actions/.github/workflows/shared_test.yml@v1
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
```

Basic:

```yml
name: My workflow
on: [push, pull_request]
jobs:
  trigger_test:
    uses: nimblehq/mulesoft-actions/.github/workflows/shared_test.yml@v1
    name: Trigger the test workflow
    with:
      upload_coverage_reports: true
    secrets:
      NEXUS_USERNAME: ${{ secrets.NEXUS_USERNAME }}
      NEXUS_PASSWORD: ${{ secrets.NEXUS_PASSWORD }}
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
```
