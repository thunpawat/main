# OpenAI authentication guide

`index.html` is a standalone, dependency-free presentation of the supplied
OpenAI authentication documentation. It follows the same isolated-page pattern
as `profile/`, `expenses/`, and `todo/`, so it does not change or depend on the
GitHub Docs application in this repository.

## Preview locally

From the repository root, run:

```shell
python3 -m http.server 4173 --directory openai-authentication
```

Then open <http://localhost:4173>.

The page is responsive and includes a mobile navigation drawer, an in-page
table of contents, highlighted active sections, and copy buttons for the TOML
configuration examples. It does not collect or store credentials.
