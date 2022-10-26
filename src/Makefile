# This is the top-level Makefile for Kaldi.
# Also see kaldi.mk which supplies options and some rules
# used by the Makefiles in the subdirectories.

SHELL := /bin/bash

# Please keep sorted alphabetically, and start each with a letter which
# is different from the first one in the last word one the row above it.
SUBDIRS := base bin decoder		\
           feat featbin fgmmbin fstbin fstext			\
           gmm gmmbin hmm                               	\
           lat latbin lm lmbin matrix				\
           online2 online2bin             	\
           transform tree util

MEMTESTDIRS = $(filter-out $(SUBDIRS))

# Optional subdirectories
EXT_SUBDIRS := online onlinebin  # python-kaldi-decoding
EXT_SUBDIRS_LIB = $(filter-out %bin, $(EXT_SUBDIRS))

include kaldi.mk

SUBDIRS_LIB = $(filter-out %bin, $(SUBDIRS))
SUBDIRS_BIN = $(filter     %bin, $(SUBDIRS))

KALDI_SONAME ?= libkaldi.so

# Reset the default goal, so that the all target will become default
.DEFAULT_GOAL :=
all: $(SUBDIRS) matrix/test
	-echo Done

mklibdir:
	test -d $(KALDILIBDIR) || mkdir $(KALDILIBDIR)

# Don't call rm -rf.
rmlibdir:
ifneq ($(KALDILIBDIR), )
	-rm $(KALDILIBDIR)/*{.so,.a,.o}
	-rmdir $(KALDILIBDIR)
else
	@true
endif

.PHONY: checkversion
checkversion:
ifeq ($(shell ./configure --version),$(CONFIGURE_VERSION))
	@echo "The version of configure script matches kaldi.mk version. Good."
else
	@echo ""
	@echo "The kaldi.mk file was generated using a different version of configure script. Please rerun the configure again"
	@test -f ./kaldi.mk && echo  "Hint: Previous configure command line: " && head -n 2 ./kaldi.mk | grep configure | sed 's/^# *//g'
	@echo ""
	@false
endif

biglib: $(SUBDIRS_LIB)
ifeq ($(KALDI_FLAVOR), dynamic)
ifeq ($(shell uname), Darwin)
	$(CXX) -dynamiclib -o $(KALDILIBDIR)/libkaldi.dylib -install_name @rpath/libkaldi.dylib -framework Accelerate $(LDFLAGS) $(wildcard $(SUBDIRS_LIB:=/*.dylib))
else
ifeq ($(shell uname), Linux)
#	$(warning the following command will probably fail, in that case add -fPIC to your CXXFLAGS and remake all)
	$(CXX) -shared -o $(KALDILIBDIR)/$(KALDI_SONAME) -Wl,-soname=$(KALDI_SONAME),--whole-archive  $(wildcard $(SUBDIRS_LIB:=/kaldi-*.a)) -Wl,--no-whole-archive  $(LDLIBS)
else
	$(error Dynamic libraries not supported on this platform. Run configure with --static flag. )
endif
endif
endif

biglibext: $(EXT_SUBDIRS_LIB)
ifeq ($(KALDI_FLAVOR), dynamic)
ifeq ($(shell uname), Darwin)
	$(CXX) -dynamiclib -o $(KALDILIBDIR)/libkaldi_ext.dylib -install_name @rpath/libkaldi_ext.dylib -framework Accelerate $(LDFLAGS) $(EXT_SUBDIRS_LIB:=/*.dylib)
else
ifeq ($(shell uname), Linux)
#	$(warning The following command will probably fail, in that case add -fPIC to your CXXFLAGS and remake all.)
	$(CXX) -shared -o $(KALDILIBDIR)/libkaldi_ext.so -Wl,-soname=libkaldi_ext.so,--whole-archive  $(EXT_SUBDIRS_LIB:=/kaldi-*.a) -Wl,--no-whole-archive
else
	$(error Dynamic libraries not supported on this platform. Run configure with --static flag. )
endif
endif
endif

kaldi.mk:
	@[ -f kaldi.mk ] || { echo "kaldi.mk does not exist; you have to run ./configure"; exit 1; }

# Compile optional stuff
ext: ext_depend $(SUBDIRS) $(EXT_SUBDIRS)
	-echo Done

check_portaudio:
	@[ -d ../tools/portaudio ] || ( cd ../tools;  ./install_portaudio.sh )

clean: rmlibdir
	-for x in $(SUBDIRS) $(EXT_SUBDIRS); do $(MAKE) -C $$x clean; done

distclean: clean
	-for x in $(SUBDIRS) $(EXT_SUBDIRS); do $(MAKE) -C $$x distclean; done

test: $(addsuffix /test, $(SUBDIRS_LIB))

ext_test: $(addsuffix /test, $(EXT_SUBDIRS_LIB))

# Define an implicit rule, expands to e.g.:
#  base/test: base
#     $(MAKE) -C base test
%/test: % mklibdir
	$(MAKE) -C $< test

valgrind:
	-for x in $(MEMTESTDIRS); do $(MAKE) -C $$x valgrind || { echo "valgrind on $$x failed"; exit 1; }; done

base/.depend.mk:
	$(MAKE) depend

depend: $(addsuffix /depend, $(SUBDIRS))

%/depend:
	$(MAKE) -C $(dir $@) depend


ext_depend: check_portaudio
	-for x in $(EXT_SUBDIRS); do $(MAKE) -C $$x depend; done

# 'make libs' to skip binaries, and build only libraries.
.PHONY: libs $(SUBDIRS_LIB)
libs: $(SUBDIRS_LIB) ;
$(SUBDIRS_LIB) : checkversion kaldi.mk mklibdir
	$(MAKE) -C $@

.PHONY: bins $(SUBDIRS_BIN) libs
$(SUBDIRS_BIN) : checkversion kaldi.mk mklibdir
	$(MAKE) -C $@

.PHONY: $(EXT_SUBDIRS)
$(EXT_SUBDIRS) : checkversion kaldi.mk mklibdir ext_depend
	$(MAKE) -C $@

### Dependency list ###
# Three base libraries that mostly everything depends on
BMU = base matrix util

# this is necessary for correct parallel compilation
#1) The tools depend on all the libraries (which is a nonsense)
bin featbin fgmmbin fstbin gmmbin \
    latbin lmbin online2bin: \
 $(BMU) decoder feat fstext gmm hmm \
        lat lm online2 transform tree

#2) The libraries have inter-dependencies
base:   base/.depend.mk
matrix: base
util:   base matrix
feat:      $(BMU) gmm transform tree
tree:      $(BMU)
gmm:       $(BMU) tree
transform: $(BMU) gmm tree
fstext:    $(BMU) tree
hmm:       $(BMU) tree
lm:        $(BMU) fstext
decoder:   $(BMU) fstext gmm hmm lat transform tree
lat:       $(BMU) hmm tree
#3) Dependencies for optional parts of Kaldi
onlinebin: $(BMU) decoder feat fstext gmm hmm lat lm online transform tree
# python-kaldi-decoding: $(BMU) decoder feat fstext gmm hmm lat online transform tree
online:    $(BMU) decoder feat gmm hmm lat transform tree
online2:   $(BMU) decoder feat gmm hmm lat transform tree
