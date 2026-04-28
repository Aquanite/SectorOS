BSEMU := bsemu
BSEMUFLAGS := --max-steps=0
EMULATEE := sector.bin

CE := chancec
CEPROJ := sector.ceproj
CEFLAGS := --freestanding

.PHONY: all build emulate

all: build
run: build emulate

build:
$(CE) $(CEPROJ) $(CEFLAGS)

emulate:
$(BSEMU) $(EMULATEE) $(BSEMUFLAGS)
