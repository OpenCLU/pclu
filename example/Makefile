# Makefile for ~CLU/example

# Sample makefile for building production executables from CLU sources

PCLU  = ${CLUHOME}/exe/pclu
PLINK = ${CLUHOME}/exe/plink

OBJS  = factorial.o

RM    = /bin/rm -f
FLUFF = *~ *.old *.bak .*~ .*.old .*.bak

factorial: ${OBJS}
	${PLINK} -opt -o $@ ${OBJS}

factorial.o: factorial.clu $(PCLU)
	${PCLU} -ext false -spec factorial -opt -cfiles -compile factorial

test: factorial
	(echo 5;echo)|./factorial>factorial.out
	diff factorial.out factorial.expect

clean:
	${RM} ${OBJS} factorial factorial.lib  pclu.xref *.c ${FLUFF}
	cd debug && ${MAKE} clean
	cd profile && ${MAKE} clean

veryclean: clean

