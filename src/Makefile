BSEMU := bsemu
BSEMUFLAGS := --max-steps=0
EMULATEE := sector.bin

CE := chancec
CEPROJ := sector.ceproj
CEFLAGS := -O3

.PHONY: all build emulate verbose

all: build
run: build emulate

build:
	$(CE) $(CEPROJ) $(CEFLAGS)

verbose:
	$(CE) $(CEPROJ) $(CEFLAGS) -vd

emulate:
	$(BSEMU) $(EMULATEE) $(BSEMUFLAGS)
