# Makefile for ~CLU/code/cmpasm (Linux version)

#  Builds files for the compiler.

OPT_FLAGS   = -O2
DEBUG_FLAGS = -g

RM    = /bin/rm -f

CFLAGS	 = -I../include
CFLAGS	+= $(OPT_FLAGS)

FILES = _bin_id _save_state _restore_state version
SOURCES = $(patsubst %,%.c,$(FILES))
OBJECTS = $(patsubst %,%.o,$(FILES))

clean:
	${RM} *.o

veryclean: clean
