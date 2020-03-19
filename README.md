# HPE Storage Container Orchestrator Documentation
This is the source files for [https://scod.hpedev.io](scod.hpedev.io). The reference documentation for all things HPE Storage Container Orchestration integration, including Docker, Kubernetes and their derives.

![Action: Publish docs via GitHub Pages](https://github.com/hpe-storage/scod/workflows/Publish%20docs%20via%20GitHub%20Pages/badge.svg)

# Build, edit and preview
Before considering contributions to SCOD, ensure you agree with the [license](docs/legal/license.md) and [contribution guidelines](docs/legal/contributing.md) established by Hewlett Packard Enterprise.

SCOD uses [MkDocs](https://www.mkdocs.org). Ensure you have Python (with `pip`) preinstalled, then install `mkdocs`.

```
pip install mkdocs
```

Fork [this repository](https://github.com/hpe-storage/scod/fork) and clone it.

```
git clone https://github.com/< your username or organization >/scod
```

Startup a local instance of MkDocs while working on your local copy.

```
mkdocs serve
```

MkDocs is now listening on [localhost](http://127.0.0.1:8000).

All the documentation lives in [docs](docs). Your edits should immediatly reload the web browser. Adding navigation is done by adding leaves in `mkdocs.yml`. Adding a new top leaf require organizing the markdown files in subfolder under docs. Images and other binary assets should live in [docs/img](docs/img) and follow the leaf, i.e an image that belongs to [docs/legal/license.md](docs/legal/license.md) should be placed in [docs/img/legal](docs/img/legal).

Once edits are done, commit and push your branch (don't forget the sign-off, see [contributing](docs/legal/contributing.md)) and submit a [pull request](https://github.com/hpe-storage/scod/pulls) (PR).

# Get in touch
The HPE storage team hang out on [hpedev.slack.com](https://hpedev.slack.com) feel free to reach out there or simply file an [issue](//github.com/hpe-storage/scod/issues) if you have any questions.

We appreciate your support and contributions!
