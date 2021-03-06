###################################################
# Makefile for OpenOpenHyperFLOW2D project        #
###################################################
SHELL     = /bin/bash
DATE      = date +%d-%b-%Y
#Version
include   .version
# Compilers
include   .compiler
# Parallel code tunings
include   .parallel
# Extended models
include   .models

ifeq     ("$(COMPILER)","Intel")
include   icc.compiler
else
include   gcc.compiler
endif

ifeq ("$(DEBUGLEVEL)","0")
LIBEXCEPT=-lexcept
endif

ARCH=`arch`

VERSION     = $(CURRENT_VER)
#-`basename $(CC)`-$(ARCH)-$(CPU)$(PARALLEL_SUFFIX)
#-D_REMOVE_SWAPFILE_
CFLAGS    =  $(OPTIONS)   -D_VER=${CURRENT_VER} \
             -D_TECPLOT_ -D_GNUPLOT_ -D_WRITE_LARGE_FILE_ -D_STREAM_COMPAT $(MODELS)
#-DSRC_ADD
LIBCFLAGS =

# StdLibs
CLFLAGS   +=
#-lm

#STATIC = -static

# Libs
#
LLIBS_2D     += $(STATIC) -L ./lib  -lDEEPS2D ${LIBEXCEPT} -lobj_data -lutl2d  -lhf2d -lflow2d -lOutCFD
	
TARGET_BASE  = OpenHyperFLOW2D
TARGET_2D    = $(TARGET_BASE)-$(VERSION)
SOURCES_2D   = hf2d_start.cpp
OBJECTS_2D   = hf2d_start.o
HF2D_START   = $(OBJECTS_2D)
DATAFILE     = CIAM-Resonator.dat
SOURCE       = $(SOURCE_2D)
OBJECTS      = $(OBJECTS_2D)

all:
	@echo '--- $(TARGET_BASE)2D v.$(CURRENT_VER)/DEEPS  ---'
	@echo 'Usage: make {command}'
	@echo 'now implemented next command:'
	@echo '"build"        - compile+link $(TARGET_BASE)2D v.$(CURRENT_VER)'
	@echo '"rebuild"      - rebuild all $(TARGET_BASE)2D v.$(CURRENT_VER) modules'
	@echo '"clean"        - delete all libriares, objects and executable files'
	@echo '"run"          - run $(TARGET_BASE)2D v.$(CURRENT_VER) program'
	@echo '"debug"        - start debugger for $(TARGET_BASE)2D v.$(CURRENT_VER)'
	@echo '"backup"       - make backup copy'
	@echo '"test"         - run tests'

build: $(TARGET_2D)

.SUFFIXES: .cpp  .c
	
.cpp.s:
	$(CXXC) -S $(INCPATH) $(CXXOPTIONS) $(CFLAGS) $<
.cpp.o:
	$(CXXC) -c $(INCPATH) $(CXXOPTIONS) $(CFLAGS) $<
.c.s:
	$(CC) -S $(INCPATH) $(CFLAGS) $<
.c.o:
	$(CC) -c $(INCPATH) $(CFLAGS) $<
libexcept:
	@if [ $(DEBUGLEVEL) -le 1 ];\
	    then\
	    cd libExcept;make -f Makefile;cd ..;\
	fi
libflow:
	@cd libFlow;make -f Makefile;cd ..;
libhyperflow:
	@cd libOpenHyperFLOW2D;make -f Makefile;cd ..;
objData:
	@cd obj_data;make -f Makefile;cd ..;
liboutcfd:
	@cd libOutCFD;make -f Makefile;cd ..;
Utl:
	@cd utl;make -f Makefile;cd ..;
libdeeps2d:
	@cd libDEEPS2D;make -f Makefile;cd ..;
clean_lib_symlinks:
	rm -f ./lib/*.a
local:
	echo -e "${MPIBIN}" > .mpi
	echo -e "${MPILIB}" > .mpilib
	./get_SIMD.sh > .simd
test: ObliqueShock Wedge Step ShowResults
	
ObliqueShock:
	bin/OpenHyperFLOW2D.sh TestCases/ObliqueShock ${NUM_NODES}
Step:
	bin/OpenHyperFLOW2D.sh TestCases/Step ${NUM_NODES}
Wedge:
	bin/OpenHyperFLOW2D.sh TestCases/Wedge ${NUM_NODES}
Bubble2D:
	bin/OpenHyperFLOW2D.sh TestCases/Bubble2D ${NUM_NODES}

ShowResults:
	gnuplot TestCases/T.dat
	gnuplot TestCases/pressure.dat
	gnuplot TestCases/Mach.dat
	gnuplot TestCases/Rho.dat
	gnuplot TestCases/U.dat
	gnuplot TestCases/V.dat
#
$(TARGET_2D): local Utl libexcept libflow objData libhyperflow libdeeps2d liboutcfd $(OBJECTS_2D)
	$(CXXC) $(STATIC) $(CLFLAGS) $(OBJECTS_2D) $(OPTIONS) $(LLIBS_2D) -o $(TARGET_2D)
	cp $(TARGET_2D) bin/$(TARGET_2D)
clean:
	rm -fR $(TARGET_2D) bin/$(TARGET_2D) .simd .mpi .mpilib .hosts *.plt *.hf2d *.o lib/* doc/doxygen/*;
	@cd libFlow;make -f Makefile clean;cd ..;\
	cd libOpenHyperFLOW2D;make -f Makefile clean;cd ..;\
	cd libExcept;make -f Makefile clean;cd ..;\
	cd obj_data;make -f Makefile clean;cd ..;\
	cd utl;make -f Makefile clean;cd ..;\
	cd libDEEPS2D;make -f Makefile clean;cd ..;\
	cd libOutCFD;make -f Makefile clean;cd ..;
backup: clean tar.bz2
	
zip:
	@zip -ur BackUp/$(TARGET_BASE)-$(VERSION)-`date +%d-%b-%Y`.zip  * .version .compiler .parallel .models -x *.bz2 *.zip *.v* *.plt *.swp *.rar *.unv BackUp/* lib/* bin/*
tar.bz2:
	@ls -A *.bz2 *.zip *.v* *.plt *.swp *.tar.* *.hf2d *.unv *.dat *.rar BackUp/* Run/* Release/* 1> EXCLUDE.txt 2>/dev/null;echo EXCLUDE.txt >> EXCLUDE.txt
	@echo $(TARGET) >> EXCLUDE.txt
	@echo BackUp/OpenHyperFLOW2D-$(VERSION)-`date +%d-%b-%Y`.tar.bz2 >> EXCLUDE.txt
	@tar -X EXCLUDE.txt -cjf  BackUp/$(TARGET_BASE)-$(VERSION)-`date +%d-%b-%Y`.tar.bz2 * .compiler .parallel .models .version
	@rm -f EXCLUDE.txt

builddoc:
	doxygen OpenHyperFLOW-doxygen.cfg

rebuild: clean build

run:
	$(TARGET) $(DATAFILE)

debug:
		
ifdef MPI
		/opt/intel/impi/4.1.0/bin64/mpiexec -np 4 ./ddd-mpi.sh ./$(TARGET_2D)  $(DATAFILE)
else
		ddd  ./$(TARGET_2D)  $(DATAFILE)
endif

