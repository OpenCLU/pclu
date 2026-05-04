PCLU  = ${CLUHOME}/exe/pclu
PLINK = ${CLUHOME}/exe/plink

test: test_uninit

uninit: uninit.o
	${PLINK} -opt -o $@ $<
uninit.o: uninit.clu
	${PCLU} -opt -compile uninit
# This test is broken, (since uninitialized variables don't work right).
test_uninit: uninit
	if ./uninit; then echo "broken test started passing:" $@; false; else echo "broken test:" $@; fi



