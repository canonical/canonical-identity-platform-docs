# Canonical Identity Platform Documentation

[![pre-commit](https://img.shields.io/badge/pre--commit-enabled-brightgreen?logo=pre-commit)](https://github.com/pre-commit/pre-commit)
[![Conventional Commits](https://img.shields.io/badge/Conventional%20Commits-1.0.0-%23FE5196.svg)](https://conventionalcommits.org)
[![Documentation Status](https://app.readthedocs.com/projects/canonical-identity/builds/?version__slug=latest)](https://app.readthedocs.com/projects/canonical-identity/builds/?version__slug=latest)

## Description

This repository contains the documentation for the [Canonical Identity Platform](https://charmhub.io/identity-platform). 

The documentation is built using [Sphinx] and is automatically published to **Read the Docs**. It utilizes Canonical's documentation standards to provide a comprehensive guide to architecture, deployment, and integration of the identity stack.

See the full documentation: [https://canonical-identity.readthedocs-hosted.com/](https://canonical-identity.readthedocs-hosted.com/)

## Structure

This section outlines the layout of this repository and the purpose of key directories.

### `docs/`

This directory contains the source files for the documentation, primarily written in **reStructuredText (.rst)**.

To build and preview the documentation locally in your browser:
1. Navigate to the root directory.
2. Run `make run`.

### `Diagram_sources/`

This directory serves as the central storage for all visual assets, including architecture and sequence diagrams, and component illustrations.

### `.github/workflows/`

This directory contains the CI/CD configurations used for documentation quality assurance.

* **Build checks:** Automated tests verify that the documentation builds without errors.
* **Link & Spelling checks:** Workflows ensure all internal/external links are valid and the text follows standard spelling conventions.
* **Read the Docs Integration:** Merges to the main branch trigger an automatic deployment to the live documentation site via a webhook integration.

## Contributing

We warmly welcome contributions to the Canonical Identity Platform documentation! Whether it's fixing a typo, improving a technical explanation, or adding a new guide, your help is appreciated.

### How to contribute
1. Fork the repository and create a new branch for your changes.
2. Ensure your documentation follows the [Canonical Documentation Style Guide](https://docs.ubuntu.com/styleguide/en/).
3. Open a Pull Request and describe the changes you've made.

Please read and sign our [Contributor Licence Agreement (CLA)] before submitting any changes.

[Sphinx]: https://www.sphinx-doc.org/
[Contributor Licence Agreement (CLA)]: https://ubuntu.com/legal/contributors
