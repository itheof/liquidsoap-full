.PHONY: all clean upload indexes dist

PWD=$(shell pwd)
# Should be built there
LIQ_DIR=$(PWD)/../liquidsoap
LIQI=$(LIQ_DIR)/doc/liqi/liqi
LIQUIDSOAP=$(LIQ_DIR)/src/liquidsoap

CONTENT=$(wildcard $(PWD)/content/*.txt $(PWD)/content/doc-svn/*.txt $(PWD)/content/doc-0.3.7/*.txt $(PWD)/content/doc-0.3.8/*.txt)
HTML=$(shell echo $(CONTENT:.txt=.html) | sed -e 's/content\//html\//g')

all: indexes $(PWD)/html/doc-svn/reference.html $(PWD)/html/doc-0.3.7/reference.html \
        $(PWD)/html/doc-0.3.8/reference.html $(PWD)/html/doc-svn/scripts/index.html \
        $(PWD)/html/doc-0.3.7/scripts/index.html $(PWD)/html/doc-0.3.8/scripts/index.html
	@find html -name '.svn' -exec rm -rf \{\} \; || true

# Removes all unecessary stuff before upload
dist: all
	@find html -name '*.txt' -exec rm -f \{\} \;

$(LIQI):
	@echo "Liqi parser does not seem to be built..."
	@exit 1

$(LIQUIDSOAP):
	@echo "Liquidsoap does not seem to be built..."
	@exit 1

indexes: $(HTML) $(PWD)/html/doc-svn/scripts/index.html $(PWD)/html/doc-0.3.7/scripts/index.html

$(PWD)/html/index.html: $(PWD)/content/index.txt $(LIQI)
	@echo Converting $(<) to $(@)...
	@test -d html || (mkdir -p html/scripts ; cp -rf orig/* html)
	@cd $(PWD)/html && $(LIQI) --main-main -i $(<) -o $(@) --basedir $(PWD)/html

$(PWD)/html/doc-0.3.7/reference.txt:
	@cp $(PWD)/reference-0.3.7.txt $(PWD)/html/doc-0.3.7/reference.txt

$(PWD)/html/doc-0.3.8/reference.txt:
	@cp $(PWD)/reference-0.3.8.txt $(PWD)/html/doc-0.3.8/reference.txt

$(PWD)/html/doc-svn/reference.txt: $(LIQUIDSOAP)
	@echo "Generating svn language reference..."
	@echo "Did you enable all features ?"
	@$(LIQUIDSOAP) $(LIQ_DIR)/scripts/utils.liq --list-plugins-xml | \
	$(LIQ_DIR)/doc/reference_to_liqi.pl > $(PWD)/html/doc-svn/reference.txt	

$(PWD)/html/doc-0.3.7/reference.html: $(PWD)/html/doc-0.3.7/reference.txt
	@cd html/doc-0.3.7 && $(LIQI) --main -i $(<) -o $(@) --basedir $(PWD)/html

$(PWD)/html/doc-0.3.8/reference.html: $(PWD)/html/doc-0.3.8/reference.txt
	@cd html/doc-0.3.8 && $(LIQI) --main -i $(<) -o $(@) --basedir $(PWD)/html

$(PWD)/html/doc-svn/reference.html: $(PWD)/html/doc-svn/reference.txt
	@cd html/doc-svn && $(LIQI) --main -i $(<) -o $(@) --basedir $(PWD)/html

$(PWD)/html/doc-%/scripts/index.html:
	@cd $(shell dirname $(@)) && \
          if [ "$(shell basename $(shell dirname $(@)))" != "html" ]; then \
          mkdir -p scripts; fi && \
          $(LIQI) --main -i $(shell dirname $(@))/index.txt -o $(@) --basedir $(PWD)/html		

$(PWD)/html/%.html: $(PWD)/content/%.txt $(LIQI)
	@echo Converting $(<) to $(@)...
	@test -d html || (mkdir html ; cp -rf orig/* html)
	@test -d $(shell dirname $(@)) || (mkdir -p $(shell dirname $(@)))
	@#Has to be called from html to put scripts in the right place
	@cd $(shell dirname $(@)) && \
	  if [ "$(shell basename $(shell dirname $(@)))" != "html" ]; then \
	  mkdir -p scripts; fi && \
	  $(LIQI) --main -i $(<) -o $(@) --basedir $(PWD)/html

clean:
	rm -rf html

RCP=rsync -rLtCP --exclude="*.swp" --exclude="*.svn" \
	--delete --stats
RHOST=$(USER)@web.sourceforge.net:/home/groups/s/sa/savonet/htdocs

upload: dist
	cd html ; $(RCP) . $(RHOST) || true
	ssh $(USER)@shell.sf.net "chgrp -R savonet \
	                            /home/groups/s/sa/savonet/htdocs \
	                            > /dev/null 2>&1"
	ssh $(USER)@shell.sf.net "find /home/groups/s/sa/savonet/htdocs \
	                            -type d -exec chmod 2775 '{}' ';' \
	                            > /dev/null 2>&1"
	ssh $(USER)@shell.sf.net "find /home/groups/s/sa/savonet/htdocs \
	                            -type f -exec chmod  664 '{}' ';' \
	                            > /dev/null 2>&1"