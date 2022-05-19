# Scraping pages using JavaScript

The `shot-scraper javascript` command can be used to execute JavaScript directly against a page and return the result as JSON.

This command doesn't produce a screenshot, but has interesting applications for scraping.

To retrieve a string title of a document:

    shot-scraper javascript https://datasette.io/ "document.title"

This returns a JSON string:
```json
"Datasette: An open source multi-tool for exploring and publishing data"
```
To return a JSON object, wrap an object literal in parenthesis:

    shot-scraper javascript https://datasette.io/ "({
      title: document.title,
      tagline: document.querySelector('.tagline').innerText
    })"

This returns:
```json
{
  "title": "Datasette: An open source multi-tool for exploring and publishing data",
  "tagline": "An open source multi-tool for exploring and publishing data"
}
```

You can pass an `async` function if you want to use `await`, including to import modules from external URLs. This example loads the [Readability.js](https://github.com/mozilla/readability) library from [Skypack](https://www.skypack.dev/) and uses it to extract the core content of a page:

```
shot-scraper javascript https://simonwillison.net/2022/Mar/14/scraping-web-pages-shot-scraper/ "
async () => {
  const readability = await import('https://cdn.skypack.dev/@mozilla/readability');
  return (new readability.Readability(document)).parse();
}"
```

To use functions such as `setInterval()`, for example if you need to delay the shot for a second to allow an animation to finish running, return a promise:

    shot-scraper javascript datasette.io "
    new Promise(done => setInterval(
      () => {
        done({
          title: document.title,
          tagline: document.querySelector('.tagline').innerText
        });
      }, 1000
    ));"

You can also save JavaScript to a file and execute it like this:

    shot-scraper javascript datasette.io -i script.js

Or read it from standard input like this:

    echo "document.title" | shot-scraper javascript datasette.io

## Handling JavaScript errors

If a JavaScript error occurs, a stack trace will be written to standard error and the tool will terminate with an exit code of 1.

This can be used to run JavaScript tests in continuous integration environments, by taking advantage of the `throw "error message"` JavaScript statement.

This example [uses GitHub Actions](https://docs.github.com/en/actions/quickstart):

```yaml
- name: Test page title
  run: |-
    shot-scraper javascript datasette.io "
      if (document.title != 'Datasette') {
        throw 'Wrong title detected';
      }"
```

Full `--help` for this command:

<!-- [[[cog
result = runner.invoke(cli.cli, ["javascript", "--help"])
help = result.output.replace("Usage: cli", "Usage: shot-scraper")
cog.out(
    "```\n{}\n```\n".format(help.strip())
)
]]] -->
```
Usage: shot-scraper javascript [OPTIONS] URL [JAVASCRIPT]

  Execute JavaScript against the page and return the result as JSON

  Usage:

      shot-scraper javascript https://datasette.io/ "document.title"

  To return a JSON object, use this:

      "({title: document.title, location: document.location})"

  To use setInterval() or similar, pass a promise:

      "new Promise(done => setInterval(
        () => {
          done({
            title: document.title,
            h2: document.querySelector('h2').innerHTML
          });
        }, 1000
      ));"

  If a JavaScript error occurs an exit code of 1 will be returned.

Options:
  -i, --input FILENAME            Read input JavaScript from this file
  -a, --auth FILENAME             Path to JSON authentication context file
  -o, --output FILENAME           Save output JSON to this file
  -b, --browser [chromium|firefox|webkit|chrome|chrome-beta]
                                  Which browser to use
  --user-agent TEXT               User-Agent header to use
  --reduced-motion                Emulate 'prefers-reduced-motion' media feature
  -h, --help                      Show this message and exit.
```
<!-- [[[end]]] -->