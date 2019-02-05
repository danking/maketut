BAR ?= 3

out:
	echo "BAR is $(BAR)" > $@

ifneq ($(shell cat env/BAR),$(BAR))
.PHONY: env/BAR
env/BAR:
	mkdir -p env
	echo "$(BAR)" > $@
endif

out2: env/BAR
	echo "BAR is $(BAR)" > $@
