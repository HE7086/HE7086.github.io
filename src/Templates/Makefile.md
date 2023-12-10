# Makefile

## basic C example

```makefile
CC ?= gcc
CFLAGS ?= -g -O0 -std=c17
ASM ?= nasm
ASMFLAGS ?= -felf64 -g
CXX ?= g++
CXXFLAGS ?= -O2 -flto -fPIC -std=c++20

SOURCES := main.c test.c
OBJECTS := $(SOURCES:.c=.o)

all: main main2

# explicit rules
main: test.o main.o
	$(CC) $(CFLAGS) -o main $^

test.o: test.asm
	$(ASM) $(ASMFLAGS) -o $@ $^

main.o: main.c
	$(CC) $(CFLAGS) -o $@ -c $^

# implicit rules	
main2: $(OBJECTS)
$(OBJECTS): %o: %c

.PHONY: clean
clean:
	rm -rf *.o *.out main
	
# automatic rules
pkgs := a b c
$(pkgs): %.c:
	@echo compile $@
```

## basic latex example
```makefile
FILE := main
OUT  := build

.PHONY: pdf
pdf:
	latexmk -interaction=nonstopmode -outdir=$(OUT) -pdf -halt-on-error $(FILE)

.PHONY: watch
watch:
	latexmk -interaction=nonstopmode -outdir=$(OUT) -pdf -pvc -halt-on-error $(FILE)

.PHONY: clean
clean:
	rm -rf $(filter-out $(OUT)/$(FILE).pdf, $(wildcard $(OUT)/*))

.PHONY: purge
purge:
	rm -rf $(OUT)
```
