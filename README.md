# HPE Storage Container Orchestrator Documentation
This is the source files for [https://scod.hpedev.io](https://scod.hpedev.io). The reference documentation for all things HPE Storage Container Orchestration integration, including Docker, Kubernetes, and their derives.

![Action: Publish docs via GitHub Pages](https://github.com/hpe-storage/scod/workflows/Publish%20docs%20via%20GitHub%20Pages/badge.svg)

# Build, edit and preview
Before considering contributions to SCOD, ensure you agree with the [license](docs/legal/license/index.md) and [contribution guidelines](docs/legal/contributing/index.md) established by Hewlett Packard Enterprise.

Fork [this repository](https://github.com/hpe-storage/scod/fork) and clone it.

```
git clone https://github.com/< your username or organization >/scod
cd scod
```

SCOD uses [MkDocs](https://www.mkdocs.org) 1.0.4. Ensure you have Python (with `pip`) preinstalled, then install `mkdocs` along with the required plugins.

```
pip install -r requirements.txt
```

Startup a local instance of MkDocs while working on your local copy.

```
mkdocs serve
```

MkDocs is now listening on [http://127.0.0.1:8000](http://127.0.0.1:8000).

All the documentation lives in [docs](docs). Your edits should immediately reload the web browser. Adding navigation is done by adding leaves in `mkdocs.yml`. Adding a new top leaf require organizing the markdown files in subfolder under docs. Images and other binary assets should live in [docs/img](docs/img) and follow the leaf, i.e an image that belongs to [docs/legal/license/index.md](docs/legal/license/index.md) should be placed in [docs/legal/license/img](docs/legal/license/img) to be easily referenced from the markdown file with relative links, `img/asset.png`.

Once edits are done, commit and push your branch (don't forget the sign-off, see [contributing](docs/legal/contributing/index.md)) and submit a [pull request](https://github.com/hpe-storage/scod/pulls) (PR).

# Style guides
The goal is to try to keep content as cohesive as possible. Some old sources may require some refactoring to fit into the MkDocs styles we adopt.

## Embedding objects
Using external sources such as YouTube and Asciinema is encouraged. Here are a few hints on how to get the best results.

### YouTube
Figure out the video ID of the video you want to embed. It's the `v` variable in the URL of the YouTube video. Let's assume the source has a 16:9 aspect ratio. Pay attention to the `width` and `height`. Replace the `<VIDEO ID>` string with the video you would like to embed.

```
<iframe width="696" height="392" src="https://www.youtube.com/embed/<VIDEO ID>" frameborder="2" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
```

**Note:** Make sure the video is "embeddable" (hit play in your local rendering).

### Asciinema
Set the column width of your terminal to 96 columns. Copy & paste the embed code straight from asciinema.org (click "Share" on the recording and copy "Embed the player"). It should look like this:
```
<script id="asciicast-236786" src="https://asciinema.org/a/236786.js" async></script>
```

## Admonitions
Adding Spinx-style exclamations into the docs is encouraged to emphasize a point in a paragraph. Triple bangs will be translated to an admonition block.

```
!!! note
    This is my note!
```

It's also possible to create your own header.
```
!!! note "Did you know?"
    Admonitions are cool!
```

There are four different types of blocks that are mapped by the following keywords:

* Blue: `note`, `seealso`
* Green: `tip`, `hint`, `important`
* Orange: `warning`, `caution`
* Purple: `danger`, `error`

## Fenced code blocks
All code blocks need to be labeled by language or style. MkDocs is not very clever to fall back to plain text; if the code block doesn't render properly, use `text` for red text and `markdown` for black text.

Start fenced code blocks like this:
```
```json
{ 
  "key": "value"
}
```

## Fenced code tabs
It's common that different incarnations of a certain command or manifest is unique to a situation where a single variable may differ. This is tedious to point out with a code block in an efficient manner as users most likely want to copy and paste the code block into a terminal. Using the `markdown-fenced-code-tabs` plugin, it's possible to "stack" code blocks on top of each other and they will appear as tabs when rendered in MkDocs.

Example: 

```
```json fct_label="Kubernetes 1.18"
{ 
  "version": "1.18"
  "dosomestuff": {
    "key": "val"
  }
}
```

```
```json fct_label="Kubernetes 1.17
{ 
  "version": "1.17",
  "dostuff": {
    "key": "val"
  }
}
```

There should be an example of this [here](https://scod.hpedev.io/csi_driver/deployment.html#advanced_install).

# Get in touch
The HPE storage team hangs out on [hpedev.slack.com](https://hpedev.slack.com) (sign up [here](https://slack.hpedev.io) if you don't have an @hpe.com address) in #kubernetes and their respective product channels, like #nimblestorage. Feel free to reach out to us there or simply file an [issue](//github.com/hpe-storage/scod/issues) if you have any questions.

We appreciate your support and contributions!
