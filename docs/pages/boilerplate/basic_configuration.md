# Basic Configuration

The configuration is written in `mkdocs.yml`. To totally differentiate from the source code, `docs` folder will be the directory which will hold everything related to documentation.

## Explanation of configuration

1. `site_name` : This is the main title for the project's documentation. This will appear on top of the documentation's site.
2. `docs_dir` : Default is `docs`, but since we are following a nested folder structure, we are using `pages` to encapsulate our documentation.
3. `site_dir`: Default is `site`, we are using these as hardcoded values in Github workflow and hence hardcoded over here as well.

> Rest are optional and can be found in official documentation: [Link](https://www.mkdocs.org/user-guide/configuration/)