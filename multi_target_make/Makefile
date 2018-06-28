# Proof-of-concept for multi-file target make
#
# By itself, make has no mechanism for keeping track of multiple outputs from a single target.
# A target of the type
#   file1 file2:
#     code
# is equivalent to
#   file1:
#     code
#   file2:
#     code
# which leads to "code" being run multiple times.
#
# The example below demonstrates a mechanism for constructing extra targets
# that work around this limitation, with these features:
# 0) Updated prerequisites trigger the code 
# 1) All file outputs missing triggers the code
# 2) Only some output files missing triggers the code
# 3) The dummy timestamp files are missing triggers the code
# 4) Parallel make with "make -j" only runs "code" once.
#
# Usage:
# Run
#   make init
# to clean the directory and setup a basic test environment.
# Run
#   make output
# to trigger the full test.
# Rerun "make init" to reset, or remove or "touch" individual files to
# simulate other test cases.
# Run
#   make file1
# and similar commands for further testing.
#
# The extra file timestamp-run is a dummy file to keep track of the last time the code was run.
# The extra file timestamp-force-run is used to trigger a forced run of the code.
#
# The basic idea is that the PHONY target run-conditionally
# is always run, but only once, doing no actualy work unless needed.
#
# 2018-06-28 Finn Lindgren, finn.lindgren@gmail.com

# These are the multiple outputs from the single target
DEPS=file1 file2 file3
INPUT=input

# High level target that depends on DEPS
# Dependency on run-conditionally needed
# to ensure sequential code running with parallel make.
output: $(INPUT) $(DEPS)
	touch output

# Dependence only, to join each separate DEPS target
# into a single target, run-conditionally
$(DEPS): $(INPUT)

# The key common dependence on run-conditionally:
$(DEPS) output: run-conditionally

# The code that does the work, .PHONY target, always run when invoked:
code:
	touch $(DEPS)

clean:
	rm -f $(INPUT) $(DEPS) output timestamp-run timestamp-force-run

init:
	$(MAKE) clean
	touch $(INPUT)

.PHONY: init clean run-conditionally code

## Nothing below this point should need editing

# Only runs if input changed, or timestamp-force-run is new
timestamp-run: $(INPUT) timestamp-force-run
	$(MAKE) code
	touch timestamp-run

# Create dummy timestamp if it doesn't exist
timestamp-force-run:
	touch timestamp-force-run

# .PHONY target, always run
# Force timestamp-run if a file doesn't exist
run-conditionally:
	@for thefile in $(DEPS); do \
		if [ ! -f $$thefile ]; then \
			touch timestamp-force-run; \
		fi; \
	done
	@$(MAKE) timestamp-run
