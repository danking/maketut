genesis:
	printf "In the beginning there was nothing" > genesis

death: genesis
	cat $< | sed 's/nothing/everything/' > $@
