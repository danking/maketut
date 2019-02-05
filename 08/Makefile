include headers.mk

%: %.c
	cc $< -o $@

headers.mk: $(shell find . -name *.c)
	rm -f headers.mk
	$(foreach file,$^,grep $(file) -e '\#include' | sed -E 's/#include *"([^"]+)"/$(basename $(file)): \1/' >> $@)
