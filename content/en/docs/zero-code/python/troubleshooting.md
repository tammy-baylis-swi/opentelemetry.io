---
title: Troubleshooting Python automatic instrumentation issues
linkTitle: Troubleshooting
weight: 40
cSpell:ignore: gunicorn
---

## Installation issues

### Python package installation failure

The Python package installs require `gcc` and `gcc-c++`, which you may need to
install if you’re running a slim version of Linux, such as CentOS.

<!-- markdownlint-disable blanks-around-fences -->

- CentOS
  ```sh
  yum -y install python3-devel
  yum -y install gcc-c++
  ```
- Debian/Ubuntu
  ```sh
  apt install -y python3-dev
  apt install -y build-essential
  ```
- Alpine
  ```sh
  apk add python3-dev
  apk add build-base
  ```

## Instrumentation issues

### Flask debug mode with reloader breaks instrumentation

The debug mode can be enabled in the Flask app like this:

```python
if __name__ == "__main__":
    app.run(port=8082, debug=True)
```

The debug mode can break instrumentation from happening because it enables a
reloader. To run instrumentation while the debug mode is enabled, set the
`use_reloader` option to `False`:

```python
if __name__ == "__main__":
    app.run(port=8082, debug=True, use_reloader=False)
```

### Pre-fork servers can break metrics export

A pre-fork server, such as Gunicorn with multiple workers, can be run like this:

```sh
gunicorn YOUR_APP:app --workers 4
```

Specifying more than one `--workers` may not generate expected metrics when
instrumentation is applied. This is because each worker maintains its own meters
and exporter(s) instead of sharing a single state.

To instrument a server with multiple workers, consider setting up a separate
instance of [Prometheus](/docs/languages/python/exporters/#prometheus-setup) to
collect metrics from all workers.

Or, initialize OpenTelemetry inside the worker process with
[manual or programmatic instrumentation](/docs/zero-code/python/example/) after
the server fork, instead of automatically. For example:

```python
from opentelemetry.instrumentation.auto_instrumentation.sitecustomize import initialize
initialize()

from your_app import app
```

Alternatively, use a single worker in pre-fork with zero-code instrumentation:

```sh
opentelemetry-instrument gunicorn your_app:app --workers 1
```

## Connectivity issues

### gRPC Connectivity

To debug Python gRPC connectivity issues, set the following gRPC debug
environment variables:

```sh
export GRPC_VERBOSITY=debug
export GRPC_TRACE=http,call_error,connectivity_state
opentelemetry-instrument python YOUR_APP.py
```
