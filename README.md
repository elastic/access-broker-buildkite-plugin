# Access Broker Gate Buildkite Plugin

Verifies that Access Broker allows a pull request before a Buildkite step
continues. It is intended for early gate steps, especially dynamic pipeline
upload steps that run before pull-request-controlled pipeline code.

The plugin sends only the repository slug and pull request number to Access
Broker:

```json
{"repository":"elastic/kibana","pull_request":12345}
```

Access Broker is expected to authenticate the Buildkite OIDC token, read signed
Buildkite claims such as `build_commit`, perform the configured checks, and
return an allow verdict.

## Usage

Reference this plugin from any Buildkite step `plugins` block:

```yaml
steps:
  - label: ":lock: Access Broker gate"
    command: buildkite-agent pipeline upload .buildkite/pipeline.yml
    plugins:
      - elastic/access-broker-gate#v1.0.0:
          broker-url: https://access-broker.example/gate
          repository: elastic/kibana
```

Pin the plugin to an immutable commit SHA or a protected release tag. The
plugin uses Buildkite OIDC for authentication and does not store Access Broker
credentials. The `repository` option is optional when `BUILDKITE_REPO` is set
to the target repository, but setting it explicitly keeps the request stable
for PR builds from forks.

## Configuration

| Option | Required | Default | Description |
| --- | --- | --- | --- |
| `broker-url` | yes | | Full Access Broker gate endpoint URL. |
| `repository` | no | parsed from `BUILDKITE_REPO` | Repository slug sent to Access Broker. |
| `audience` | no | `elastic-access-broker` | OIDC audience expected by Access Broker. |
| `oidc-claims` | no | `organization_slug,pipeline_id,pipeline_slug,build_id,build_branch,build_commit,cluster_id,queue_id,queue_key,step_key,job_id` | Optional Buildkite OIDC claims to request. |
| `oidc-lifetime` | no | `30` | Requested OIDC token lifetime in seconds. |
| `curl-timeout-seconds` | no | `30` | Maximum Access Broker request duration. |

## Verdict Contract

The plugin fails closed unless Access Broker returns HTTP 2xx with:

```json
{"verdict":"allow"}
```

Any non-2xx response, missing `allow` verdict, or denial response fails the
step.
