BSEMUFLAGS := --max-steps=0
BSEMUFILE := sector.bin

CHANCEPROJ := sector.ceproj

CHANCEFLAGS := --freestanding

IS_WSL := $(shell grep -qi microsoft /proc/version 2>/dev/null && echo 1 || echo 0)

.PHONY: all build emulate

all: build emulate

build:
ifeq ($(OS),Windows_NT)
	chancec $(CHANCEPROJ) $(CHANCEFLAGS)
else
	sudo chancec $(CHANCEPROJ) $(CHANCEFLAGS)
endif

emulate:
ifeq ($(IS_WSL), 1)
	bsemu.exe $(BSEMUFILE) $(BSEMUFLAGS)
else
	bsemu $(BSEMUFILE) $(BSEMUFLAGS)
endif