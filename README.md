# openshift-fluentd

S2I configuration for building a [FluentD](https://www.fluentd.org/) container that forwards logs from OpenShift to [Splunk](https://www.splunk.com/) via the HTTP Event Collector (HEC).

## How It Works

1. **Source**: FluentD listens on port 24284 using the `secure_forward` input plugin, accepting log streams from OpenShift's FluentD pods (or any `fluent-logger` client).
2. **Processing**: Incoming records pass through a stdout filter for debugging.
3. **Sink**: Records are sent to Splunk HEC using the `fluent-plugin-splunkhec` output plugin, with batched JSON events and configurable buffering.

## Usage

Deploy as an OpenShift application using S2I (Source-to-Image):

```bash
oc new-app registry.access.redhat.com/rhscl/ruby-24-rhel7~https://github.com/larkly/openshift-fluentd.git \
  --name=secure-forwarder \
  -l component=fluentd-forwarder \
  -e GEM_PATH=/opt/app-root/src/bundle/ruby/2.4.0 \
  -n logging
```

This builds a Docker image from the S2I base (`ruby-24-rhel7`) and the source in this repository, then deploys it as a DeploymentConfig named `secure-forwarder` in the `logging` namespace.

## Configuration

The deployment accepts the following environment variables. If unset, defaults from `config/fluentd.conf` are used.

| Variable | Purpose | Default |
|----------|---------|---------|
| `SPLUNK_TOKEN` | Splunk HEC authentication token | Hardcoded placeholder in `fluentd.conf` |
| `SPLUNK_SERVER` | Splunk HEC hostname | `splunk.splunk.svc.cluster.local` |
| `SPLUNK_PORT` | Splunk HEC port | `8088` |
| `SPLUNK_PROTOCOL` | Protocol for Splunk HEC (`http` or `https`) | `http` |
| `SPLUNK_INDEX` | Splunk index name | `openshift` |
| `FLUENT_CONF` | Path to the FluentD config file | `/etc/config/fluentd.conf` (mounted ConfigMap) or `/opt/app-root/src/config/fluentd.conf` (built-in) |

If a ConfigMap is mounted at `/etc/config/fluentd.conf`, it takes precedence over the built-in config. Environment variables (`SPLUNK_*`) are applied via `sed` replacement at container startup (see `.s2i/bin/run`).

## File Structure

```
openshift-fluentd/
├── .s2i/
│   └── bin/
│       └── run            # S2I run script: sets env vars, starts FluentD
├── config/
│   └── fluentd.conf       # FluentD configuration (source, filter, output)
├── Gemfile                # Ruby gems (fluentd, splunkhec, secure-forward)
├── LICENSE
└── README.md
```

## Prerequisites

- OpenShift 3.x or 4.x with S2I support
- Access to the Red Hat SCL `ruby-24-rhel7` base image
- A Splunk instance with HEC enabled and a valid token
- An OpenShift project/namespace (e.g., `logging`) to deploy into

## Dependencies

All gems are listed in the `Gemfile`:

- `fluentd` (0.14.20)
- `fluent-plugin-secure-forward` (0.4.5)
- `fluent-plugin-splunkhec` (1.5)
- `multi_json`

## License

See [LICENSE](LICENSE).
