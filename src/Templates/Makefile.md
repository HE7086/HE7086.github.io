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

## c++
```makefile
LTOFLAGS ?= -flto=auto -ffat-lto-objects 
CXXFLAGS ?= -std=c++23 -march=x86-64 -O2 -s -fno-plt -pipe \
			-fPIC -fstack-clash-protection -fcf-protection \
			-Wall -Wextra -Wpedantic -Werror \
			$(LTOFLAGS)
CPPFLAGS ?= -Wp,-D_FORTIFY_SOURCE=2,-D_GLIBCXX_ASSERTIONS
LDFLAGS ?= -Wl,-O1,--sort-common,--as-needed,-z,relro,-z,now
LDLIBS ?= -lstdc++

PREFIX ?= /usr/local

SOURCES := $(wildcard *.cpp)
OBJECTS := $(SOURCES:.cpp=.o)
EXECUTABLE := name_of_exe

all: $(EXECUTABLE)

$(EXECUTABLE): $(OBJECTS)
$(OBJECTS): %o: %cpp

install: $(EXECUTABLE)
	install -Dm755 $^ $(PREFIX)/bin/$^

.PHONY: clean
clean:
	$(RM) $(EXECUTABLE) *.o
```

* use mold: `LD_FLAGS += -fuse-ld=mold`
