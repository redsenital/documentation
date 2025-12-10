# Configuration
MAIN = main
OUTPUT_DIR = outputs

# Check if 'out' parameter is given
ifeq ($(out),)
	OUTPUT_FILE = $(MAIN).pdf
	BUILD_TARGET = build-local
else
	OUTPUT_FILE = $(OUTPUT_DIR)/$(MAIN).pdf
	BUILD_TARGET = build-output
endif

# Ensure output directory exists
$(shell mkdir -p $(OUTPUT_DIR))

all: $(BUILD_TARGET)

build-local: $(MAIN).tex
	pdflatex -shell-escape $(MAIN).tex
	bibtex $(MAIN) || true
	pdflatex -shell-escape $(MAIN).tex
	pdflatex -shell-escape $(MAIN).tex

build-output: $(OUTPUT_DIR)/$(MAIN).pdf

$(OUTPUT_DIR)/$(MAIN).pdf: $(MAIN).tex
	pdflatex -shell-escape $(MAIN).tex
	bibtex $(MAIN) || true
	pdflatex -shell-escape $(MAIN).tex
	pdflatex -shell-escape $(MAIN).tex
	mv $(MAIN).pdf $(OUTPUT_DIR)/

clean:
	rm -f *.aux *.log *.out *.toc *.lof *.lot *.bbl *.blg *.acn *.glo *.ist
	rm -f includes/*.aux chapters/*.aux
	rm -rf _minted-$(MAIN)/

cleanall: clean
	rm -rf $(OUTPUT_DIR)/*.pdf

view: $(OUTPUT_DIR)/$(MAIN).pdf
	xdg-open $(OUTPUT_DIR)/$(MAIN).pdf 2>/dev/null || open $(OUTPUT_DIR)/$(MAIN).pdf 2>/dev/null || echo "Please open $(OUTPUT_DIR)/$(MAIN).pdf manually"

.PHONY: all clean cleanall view
