Documentation for Vapor cluster. The documentation is written in ReStructuredText and converted to
HTML to publish on Github Pages.

## Requirements

You will need `docutils` with `pygments`. Use `pipx` to install these in an isolated environment
without having to wrestle with virtual environments yourself.

```
pipx install docutils
pipx inject docutils pygments
```

## To Build

Simply run

```
./build.sh
```

This will generate a single HTML file `index.html` from the `vapor_user_guide.rst` file. When pushed to Github, it will be automatically visible at https://olcf.github.io/vapor-docs . 
