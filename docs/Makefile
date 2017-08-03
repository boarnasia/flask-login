# MakeFile for building all the docs at once.
# Inspired by the Makefile used by bazaar.
# http://bazaar.launchpad.net/~bzr-pqm/bzr/2.3/

PYTHON = python
ES_HOST =

.PHONY: all clean html latexpdf epub htmlhelp website website-dirs rebuild-index

# Languages that can be built.
LANGS = en ja

# pdflatex does not like ja, zh & tr for some reason.
PDF_LANGS = en

# Dependencies to perform before running other builds.
# Clone the en/Makefile everywhere.
SPHINX_DEPENDENCIES = $(foreach lang, $(LANGS), $(lang)/Makefile)

# Copy-paste the English Makefile everywhere it's needed (if non existing).
%/Makefile: en/Makefile
	cp -n $< $@

#
# The various formats the documentation can be created in.
#
# Loop over the possible languages and call other build targets.
#
html: $(foreach lang, $(LANGS), html-$(lang))
htmlhelp: $(foreach lang, $(LANGS), htmlhelp-$(lang))
epub: $(foreach lang, $(LANGS), epub-$(lang))
latex: $(foreach lang, $(PDF_LANGS), latex-$(lang))
pdf: $(foreach lang, $(PDF_LANGS), pdf-$(lang))
htmlhelp: $(foreach lang, $(LANGS), htmlhelp-$(lang))
populate-index: $(foreach lang, $(LANGS), populate-index-$(lang))
server: $(foreach lang, $(LANGS), server-$(lang))
rebuild-index: $(foreach lang, $(LANGS), rebuild-index-$(lang))


# Make the HTML version of the documentation with correctly nested language folders.
html-%: $(SPHINX_DEPENDENCIES)
	cd $* && make html LANG=$*

htmlhelp-%: $(SPHINX_DEPENDENCIES)
	cd $* && make htmlhelp LANG=$*

epub-%: $(SPHINX_DEPENDENCIES)
	cd $* && make epub LANG=$*

latex-%: $(SPHINX_DEPENDENCIES)
	cd $* && make latex LANG=$*

pdf-%: $(SPHINX_DEPENDENCIES)
	cd $* && make latexpdf LANG=$*

server-%: $(SPHINX_DEPENDENCIES)
	cd build/html/$* && python -m SimpleHTTPServer

clean:
	rm -rf build/*

