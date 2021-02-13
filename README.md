# Design Documentation Template

This repository is a Design Documentation template using `pandoc`.

If you'd like to see a deployed version of this repository: [Demo](https://design-documentation-template.netlify.app/index.html)

## Setup

- Run `make deps` to find the dependencies you need for this project.
- Otherwise, the only dependency required is `pandoc`, which can be found here: [Pandoc](https://github.com/jgm/pandoc)

## Development

- Run `make` to generate an HTML, PDF, and DOCX version of your design documentation in the `output` directory.
- Run `make html` to generate only an HTML version.
- Run `make pdf` to generate only a PDF version.
- Run `make docx` to generate only a DOCX version.
- Run `make deploy` to deploy the site to production.
