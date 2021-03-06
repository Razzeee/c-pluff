## Process this file with automake to produce Makefile.in.

# Copyright 2007 Johannes Lehtinen
# This Makefile is free software; Johannes Lehtinen gives unlimited
# permission to copy, distribute and modify it.

# This can be defined to a debugging wrapper program such as Valgrind.
TEST_WRAPPER =

# This can be defined to a subset of test cases to be performed
SELECTED_TESTS = all

DIST_SUBDIRS = plugins-source

EXTRA_DIST = tests.txt

CPPFLAGS = @CPPFLAGS@
CPPFLAGS += -DCP_HOST="\"$(host)\""

LIBS = @LIBS_OTHER@ @LIBS_TESTSUITE@ @LIBS@

check_PROGRAMS = testsuite

testsuite_SOURCES = psymbolusage.c extcfg.c pdependencies.c pcallbacks.c pscanning.c pinstallation.c ploading.c loggers.c collections.c ploaders.c initdestroy.c fatalerror.c cpinfo.c testmain.c test.h
testsuite_LDFLAGS = -dlopen self

testsuite_cxx_SOURCES = initdestroy_cxx.cc fatalerror_cxx.cc cpinfo_cxx.cc test_cxx.cc test_cxx.h testmain.c test.h
testsuite_cxx_LDADD = @LIBS_OTHER_XX@
testsuite_cxx_LDFLAGS = -dlopen self

# Enable C++ tests if C++ API enabled
if CPLUFFXX
TEST_CPLUFFXX = yes
check_PROGRAMS += testsuite_cxx
else
TEST_CPLUFFXX = no
endif

tmpinstalldir = $(CURDIR)/tmp/install

check-local: install-plugins
	@numf=0; numt=0; \
	run_test () { \
		local testsuite="$$1"; \
		local test="$$2"; \
		if test -x "$(srcdir)/test-$$test"; then \
			srcdir='$(srcdir)' TEST_WRAPPER='$(TEST_WRAPPER)' "$(srcdir)/test-$$test"; \
			rc=$$?; \
		else \
			eval srcdir='$(srcdir)' libtool --mode=execute $(TEST_WRAPPER) "\$$testsuite" "\$$test"; \
			rc=$$?; \
		fi; \
		case $$rc in \
			77) \
				echo "SKIPPED: $$test"; \
				;; \
			0) \
				echo "OK: $$test"; \
				numt=$$(($$numt + 1)); \
				;; \
			*) \
				echo "FAIL: $$test"; \
				numt=$$(($$numt + 1)); \
				numf=$$(($$numf + 1)); \
				;; \
		esac; \
	}; \
	echo; \
	echo '===================================================================='; \
	echo 'C-Pluff Test Suite'; \
	echo '===================================================================='; \
	if test '$(SELECTED_TESTS)' = all; then \
		while read test; do \
			run_test ./testsuite "$$test"; \
		done < '$(srcdir)/tests.txt'; \
		if test '$(TEST_CPLUFFXX)' = yes; then \
			while read test; do \
				run_test ./testsuite_cxx "$$test"; \
			done < '$(srcdir)/tests_cxx.txt'; \
		fi; \
	else \
		for test in $(SELECTED_TESTS); do \
			run_test "$$test"; \
		done; \
	fi; \
	echo '===================================================================='; \
	if test $$numf -gt 0; then \
		echo "FAILED: $$numf/$$numt"; \
	else \
		echo 'ALL OK!'; \
	fi; \
	echo '===================================================================='; \
	echo; \
	test $$numf -eq 0

clean-local:
	rm -rf tmp
	test ! -f plugins-source/Makefile || (cd plugins-source && $(MAKE) $(AM_MAKEFLAGS) clean)

dist-hook:
	$(MKDIR_P) '$(distdir)'
	cp -p '$(srcdir)'/test-* '$(distdir)'
	$(MKDIR_P) '$(distdir)/expected'
	cp -p '$(srcdir)'/expected/*.txt '$(distdir)'/expected
	$(MKDIR_P) '$(distdir)'/plugins
	for d in $$(ls '$(srcdir)'/plugins); do \
		( $(MKDIR_P) '$(distdir)'/plugins/"$$d" && \
			cp -p '$(srcdir)'"/plugins/$$d/plugin.xml" '$(distdir)'/plugins/"$$d" ) \
		|| exit 1; \
	done
	$(MKDIR_P) '$(distdir)'/pcollections
	for d in $$(ls '$(srcdir)'/pcollections); do \
		for d2 in $$(ls '$(srcdir)'"/pcollections/$$d"); do \
			( $(MKDIR_P) '$(distdir)'/pcollections/"$$d/$$d2" && \
				cp -p '$(srcdir)'"/pcollections/$$d/$$d2/plugin.xml" '$(distdir)'/pcollections/"$$d/$$d2" ) \
			|| exit 1; \
		done; \
	done

install-plugins: build-plugins install-libcpluff
	cd plugins-source && $(MAKE) $(AM_MAKEFLAGS) DESTDIR='$(tmpinstalldir)' install

build-plugins:
	cd plugins-source && $(MAKE)

install-libcpluff:
	cd ../libcpluff && $(MAKE) $(AM_MAKEFLAGS) DESTDIR='$(tmpinstalldir)' install

install-libcpluffxx:
	cd ../libcpluffxx && $(MAKE) $(AM_MAKEFLAGS) DESTDIR='$(tmpinstalldir)' install

.PHONY: build-plugins install-plugins install-libcpluff install-libcpluffxx
