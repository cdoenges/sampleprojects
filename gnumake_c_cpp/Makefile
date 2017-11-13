#
# Requires GNUMake 4.2.1 oder newer.
#
# Dependency-handling as described in http://make.mad-scientist.net/papers/advanced-auto-dependency-generation/.
#
SHELL = /bin/bash

# Include GNUMake Standard Library (http://gmsl.sourceforge.net/).
-include gmsl




# ---------------------------------------------------------------------------
# Basic configuration and settings.
# ---------------------------------------------------------------------------

# Various directories
DEPDIR = .d
INCDIR = inc
LIBDIR = lib
LSTDIR = lst
OBJDIR = obj
RELDIR = rel
SRCDIR = src
TMPDIR = tmp

# Shell commands
CTAGS = ctags
MV = mv
RM = rm


INCS := $(wildcard $(INCDIR)/*.h)
# Find all .c and .cpp files in $(SRCDIR) and turns them into the filename
# $(OBJDIR)/*.o
OBJS := $(patsubst $(SRCDIR)/%.c,$(OBJDIR)/%.o,$(wildcard $(SRCDIR)/*.c)) \
        $(patsubst $(SRCDIR)/%.cpp,$(OBJDIR)/%.o,$(wildcard $(SRCDIR)/*.cpp))
SRCS := $(wildcard $(SRCDIR)/*.c) $(wildcard $(SRCDIR)/*.cpp)

# Determine which compiler to use.
HAVE_CLANG := $(shell which clang 2>/dev/null)
HAVE_GCC := $(shell which gcc)

ifdef HAVE_CLANG
CC  = clang
CPP = clang++
$(info Using clang.)
else
ifdef HAVE_GCC
CC = gcc
CPP = g++
$(info Using GCC.)
else
$(error No supported compiler found.)
endif # !HAVE_GCC
endif # !HAVE_CLANG

#$(info Using $(shell $(CC) --version).)
#$(info Using $(shell $(CPP) --version).)



# Enable pre-processor listings by using
# gcc/g++  -E -fdirectives-only  -o preprocessor_list.lst  MySrc.cpp

# Enable assembly listings by using
# gcc/g++ -Wa,-adhln -g MySrc.c -o assembly_list.s

CFLAGS   = -std=c11 -I$(INCDIR) -g -Wall
CPPFLAGS = -std=c++14 -I$(INCDIR) -g -Wall


UNIQUE_LABEL ?= $(USER)_$(shell date +%d%m%Y_%H%M%S)


# ---------------------------------------------------------------------------
# Build targets.
# ---------------------------------------------------------------------------

# The default target must be defined first.
# Build everything.
all: $(RELDIR)/sample.app
	@echo "all done."


# Output various variables.
.PHONY: debug
debug:
	@echo OBJS: $(OBJS)


# Create a tags file.
tags: $(INCS) $(SRCS)
	$(CTAGS) -o tags $(INCS) $(SRCS)


# Delete all generated files and directories.
.PHONY : clean
clean:
	-rm -Rf $(DEPDIR) $(LSTDIR) $(OBJDIR) $(RELDIR) $(TMPDIR) tags

$(DEPDIR):
	mkdir -p $(DEPDIR)

$(LSTDIR):
	mkdir -p $(LSTDIR)

$(OBJDIR):
	mkdir -p $(OBJDIR)

$(RELDIR):
	mkdir -p $(RELDIR)

$(TMPDIR):
	@mkdir -p $(TMPDIR)



# ---------------------------------------------------------------------------
# Stuff for Makefile-debugging.
# ---------------------------------------------------------------------------

# 'make print-XYZ' outputs the value of variable 'XYZ'
print-%: ; @echo $* = $($*)

# 'make printvars' outputs the value of all variables.
.PHONY: printvars
printvars:
	@$(foreach V,$(sort $(.VARIABLES)), \
	$(if $(filter-out environ% default automatic, \
	$(origin $V)),$(info $V=$($V) ($(value $V)))))

# 'make TRACE=<variable>' outputs all lines where <variable> is changed.
ifdef TRACE
.PHONY: _trace _value
_trace: ; @$(MAKE) --no-print-directory TRACE= \
    $(TRACE)='$$(warning TRACE $(TRACE))$(shell $(MAKE) TRACE=$(TRACE) _value)'
_value: ; @echo '$(value $(TRACE))'
endif




# ---------------------------------------------------------------------------
# Rules.
# ---------------------------------------------------------------------------

# Dependency handling.
DEPFLAGS = -MT $@ -MMD -MP -MF $(DEPDIR)/$*.Td
POSTCOMPILE = @$(MV) -f $(DEPDIR)/$*.Td $(DEPDIR)/$*.d && touch $@

# Rule to prevent deleting of dependency files.
$(DEPDIR)/%.d: ;
.PRECIOUS: $(DEPDIR)/%.d


# General rule and recipe to turn $(SRCDIR)/*.c into $(OBJDIR)/*.o.
$(OBJDIR)/%.o : $(SRCDIR)/%.c
$(OBJDIR)/%.o: $(SRCDIR)/%.c $(DEPDIR)/%.d | $(DEPDIR) $(OBJDIR)
	@echo Building $@ from $< ...
	$(CC) -c $(DEPFLAGS) $(CFLAGS) -o $@ $<
	$(POSTCOMPILE)


# General rule and recipe to turn $(SRCDIR)/*.cpp into $(OBJDIR)/*.o.
$(OBJDIR)/%.o: $(SRCDIR)/%.cpp
$(OBJDIR)/%.o: $(SRCDIR)/%.cpp $(DEPDIR)/%.d | $(DEPDIR) $(OBJDIR)
	@echo Building $@ from $< ...
	$(CPP) -c $(DEPFLAGS) $(CPPFLAGS) -o $@ $<
	$(POSTCOMPILE)


# The unique label file is generated for each build.
ULF := $(OBJDIR)/unique_label.h
.PHONY: $(ULF)
$(ULF): | $(OBJDIR)
	@echo >$(ULF) #define UNIQUE_LABEL \"$(UNIQUE_LABEL)\"

$(OBJDIR)/unique_label.o: $(ULF)



$(RELDIR)/sample.app: $(OBJS) $(OBJDIR)/unique_label.o | $(OBJDIR) $(RELDIR)
	@echo Building $@ from $< ...
	$(CPP) $(CCFLAGS) -lstdc++ -o $@ $(OBJS)

#	        -Map=$(LSTDIR)/sample.map





# Dependencies einlesen. Für jede *.c- und *.cpp-Datei in $(SRCDIR) gibt es
# eine Datei $(DEPDIR)/*.d, die gelesen wird.
include $(patsubst $(SRCDIR)/%.c,$(DEPDIR)/%.d,$(wildcard $(SRCDIR)/*.c))
include $(patsubst $(SRCDIR)/%.cpp,$(DEPDIR)/%.d,$(wildcard $(SRCDIR)/*.cpp))
