ACLOCAL_AMFLAGS = -I m4

AM_CPPFLAGS = -D_GNU_SOURCE -D_FILE_OFFSET_BITS=64 $(GLIB_VER_DEFINES) \
	-DVMNETFS_SCHEMA_PATH=\"$(pkgpythondir)/schema/vmnetfs.xsd\"
AM_CFLAGS = -std=gnu99 -W -Wall -Wstrict-prototypes -pthread \
	$(libcurl_CFLAGS) $(glib_CFLAGS) $(gthread_CFLAGS) $(fuse_CFLAGS) \
	$(libxml2_CFLAGS)
AM_LDFLAGS = -pthread $(libcurl_LIBS) $(glib_LIBS) $(gthread_LIBS) \
	$(fuse_LIBS) $(libxml2_LIBS)

dist_bin_SCRIPTS = tools/vmnetx \
	tools/vmnetx-server \
	tools/vmnetx-generate

pkglibexec_PROGRAMS = vmnetfs/vmnetfs
vmnetfs_vmnetfs_SOURCES = vmnetfs/bitmap.c \
	vmnetfs/cond.c \
	vmnetfs/fuse.c \
	vmnetfs/fuse-image.c \
	vmnetfs/fuse-stats.c \
	vmnetfs/fuse-stream.c \
	vmnetfs/io.c \
	vmnetfs/ll-modified.c \
	vmnetfs/ll-pristine.c \
	vmnetfs/log.c \
	vmnetfs/pollable.c \
	vmnetfs/stats.c \
	vmnetfs/stream.c \
	vmnetfs/transport.c \
	vmnetfs/util.c \
	vmnetfs/vmnetfs.c \
	vmnetfs/vmnetfs-private.h

nobase_python_PYTHON = vmnetx/__init__.py \
	vmnetx/define.py \
	vmnetx/domain.py \
	vmnetx/generate.py \
	vmnetx/package.py \
	vmnetx/protocol.py \
	vmnetx/reference.py \
	vmnetx/util.py \
	vmnetx/controller/__init__.py \
	vmnetx/controller/local/__init__.py \
	vmnetx/controller/local/monitor.py \
	vmnetx/controller/local/virtevent.py \
	vmnetx/controller/local/vmnetfs.py \
	vmnetx/controller/remote.py \
	vmnetx/server/__init__.py \
	vmnetx/ui/__init__.py \
	vmnetx/ui/view.py
nobase_nodist_python_PYTHON = vmnetx/system.py
CLEANFILES = vmnetx/system.py
EXTRA_DIST = vmnetx/system.py.in pylintrc vmnetx.te README.rst

nobase_dist_pkgpython_DATA = schema/domain.xsd \
	schema/domain-metadata.xsd \
	schema/package.xsd \
	schema/reference.xsd \
	schema/vmnetfs.xsd \
	schema/libvirt/basictypes.rng \
	schema/libvirt/domaincommon.rng \
	schema/libvirt/domain.rng \
	schema/libvirt/networkcommon.rng \
	schema/libvirt/storageencryption.rng

applicationsdir = $(datarootdir)/applications
mimedir = $(datarootdir)/mime/packages
dist_applications_DATA = desktop/vmnetx.desktop
dist_mime_DATA = desktop/vmnetx.xml

man_MANS = man/vmnetx.1 man/vmnetx-generate.1
CLEANFILES += man/vmnetx.1 man/vmnetx-generate.1
EXTRA_DIST += man/vmnetx.1.in man/vmnetx-generate.1.in

dbusconfdir = $(sysconfdir)/dbus-1/system.d
dbusservicedir = $(datarootdir)/dbus-1/system-services
polkitactiondir = $(datarootdir)/polkit-1/actions
dist_pkglibexec_SCRIPTS = authorizer/authorizer
dist_dbusconf_DATA = authorizer/org.olivearchive.VMNetX.Authorizer.conf
dbusservice_DATA = authorizer/org.olivearchive.VMNetX.Authorizer.service
dist_polkitaction_DATA = authorizer/org.olivearchive.VMNetX.Authorizer.policy
CLEANFILES += authorizer/org.olivearchive.VMNetX.Authorizer.service
EXTRA_DIST += authorizer/org.olivearchive.VMNetX.Authorizer.service.in

vmnetx/system.py: vmnetx/system.py.in Makefile
	$(AM_V_at)$(MKDIR_P) vmnetx
	$(AM_V_GEN) sed -e "s:@pkglibexecdir@:$(pkglibexecdir):g" \
		-e "s:@version@:$(VERSION):g" $< > $@

%.1: %.1.in Makefile
	$(AM_V_at)$(MKDIR_P) man
	$(AM_V_GEN) sed -e "s:@version@:$(VERSION):g" $< > $@

%.service: %.service.in Makefile
	$(AM_V_at)$(MKDIR_P) authorizer
	$(AM_V_GEN) sed -e "s:@pkglibexecdir@:$(pkglibexecdir):g" $< > $@

update-databases:
	-update-mime-database $(datarootdir)/mime
	-update-desktop-database $(applicationsdir)

install-data-hook: update-databases
uninstall-hook: update-databases

check-local: $(nobase_nodist_python_PYTHON)
	@# Combine package sources and generated system.py into one directory
	rm -rf lint
	mkdir lint
	cp -r $(srcdir)/vmnetx lint/vmnetx
	@# "make distcheck" sets directories read-only; cp follows suit
	chmod -R u+w lint
	cp -r $(builddir)/vmnetx/* lint/vmnetx
	find lint -type f ! -name \*.py -exec rm {} \;
	@# Fail on serious pylint errors
	cd lint && PYTHONPATH=. \
		$(PYLINT) --rcfile=$(abs_srcdir)/pylintrc vmnetx || \
		[ $$(($$? & 3)) -eq 0 ]
	rm -rf lint

# When checking "make uninstall", ignore files generated by update-databases
distuninstallcheck_listfiles = find . -path ./share/mime -prune -type f -o \
	-type f ! -name mimeinfo.cache
