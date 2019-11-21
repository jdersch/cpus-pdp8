
MACRO = ../../utils/macro/macro8x

all: chekmo.bin

clean:
	rm -f *.bin *.prm *.lst *.err *.mem *.dump

chekmo.bin:
	rm -f chekmo.err chekmo.lst
	$(MACRO) -o chekmo chekmo.pal

