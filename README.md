[![Website](https://img.shields.io/badge/tstn--047-lsst.io-brightgreen.svg)](https://tstn-047.lsst.io)
[![CI](https://github.com/lsst-tstn/tstn-047/actions/workflows/ci.yaml/badge.svg)](https://github.com/lsst-tstn/tstn-047/actions/workflows/ci.yaml)

# Operator Control of Rubin Observatory NVR Camera Infrared Illuminators

## TSTN-047

A solution is proposed to provide web dashboard control of the infrared illuminators on the UniFi web cameras deployed throughout the observatory, with needing to provide and admin-level UniFi account for each operator.

**Links:**

- Publication URL: https://tstn-047.lsst.io
- Alternative editions: https://tstn-047.lsst.io/v
- GitHub repository: https://github.com/lsst-tstn/tstn-047
- Build system: https://github.com/lsst-tstn/tstn-047/actions/


## Build this technical note

You can clone this repository and build the technote locally if your system has Python 3.11 or later:

.. code-block:: bash

```sh
git clone https://github.com/lsst-tstn/tstn-047
cd tstn-047
make init
make html
```

Repeat the `make html` command to rebuild the technote after making changes.
If you need to delete any intermediate files for a clean build, run `make clean`.

The built technote is located at `_build/html/index.html`.

## Publishing changes to the web

This technote is published to https://tstn-047.lsst.io whenever you push changes to the `main` branch on GitHub.
When you push changes to a another branch, a preview of the technote is published to https://tstn-047.lsst.io/v.

## Editing this technical note

The main content of this technote is in `index.md` (a Markdown file parsed as [CommonMark/MyST](https://myst-parser.readthedocs.io/en/latest/index.html)).
Metadata and configuration is in the `technote.toml` file.
For guidance on creating content and information about specifying metadata and configuration, see the Documenteer documentation: https://documenteer.lsst.io/technotes.
