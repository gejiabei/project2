
# Documentation

This repository contains the Sphinx/RST source code for the CoreNIC user guide
as well as the firmware design documentation. Many output formats are supported
by Sphinx such as PDF, Latex, HTML.

# Installing Dependencies

## Debian Systems:

Firstly install sphinx with:

`apt-get install python3-sphinx`

Enchant (C library) is required and can be installed with:

`sudo apt-get install libenchant1c2a`

Additionally the sphinxcontrib extensions: `spelling` and `imageembed` are
required and can be installed with:

`pip3 install sphinxcontrib-spelling`

`pip3 install sphinxcontrib-imageembed`

By default the documentation uses the sphinx_rtd_theme for visual styling.
To install this dependency, run:

`pip3 install sphinx-rtd-theme`

# Building Documentation

There are two documentation components:

* **CoreNIC User Guide** - Defined in `user-guide/`

* **Firmware Design Docs** - Defined in `firmware-design-docs/`

## Help

For a list of available build targets, simply run `make` in the
directory of each documentation component.

## Configuration

The source and build output directories are defined in a file called
**default-build-options.mk** in the repository's main directory. If
you wish to make changes, such as defining a different build output
directory, you may modify this file directly.

## HTML:

To build the HTML version of both documentation components run
`make html` in the directory of each documentation component
(directory that contains the makefile). An output folder called `build`
will then be created which contains a folder called `html`. Within the `html`
folder you will find `index.html` which is the main HTML file for the
respective documentation component.

## Note: Patching imageembed

The current version of `sphinxcontrib-imageembed` contains a bug which
results in the base64 content of images being corrupted during embedding.
A temporary patch can be applied to this library by running the following
commands:
```
sphinxpath=`pip3 show sphinxcontrib-imageembed | grep Location | awk '{print $2}'`
```
```
find $sphinxpath -name "imageembed.py" -exec sed -i 's/encoded = base64.b64encode(open(filename, "rb").read())\s*$/encoded = base64.b64encode(open(filename, "rb").read()).decode("ascii")/g' {} \;
```
Depending on your python3 context it may be necessary to append **sudo**
to the above two commands.
