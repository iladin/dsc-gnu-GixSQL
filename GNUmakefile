COBC := /usr/bin/cobc
PSQL := /usr/bin/psql
PSQL_VERSION ?= 12
PSQL_SERVICE := /var/run/postgresql/.s.PGSQL.5432
DB ?= postgres
sudoers_file :=  /etc/sudoers.d/$(USER)



SHELL := bash # Prefer bash shell
MAKEFILE := $(abspath $(lastword $(MAKEFILE_LIST)))
# Recompile all targets when the makefile changes
.EXTRA_PREREQS:= $(MAKEFILE)
# Disable un-needed compile rules
MAKEFLAGS += --no-builtin-rules
.SUFFIXES:
all: $(COBC) $(PSQL) $(PSQL_SERVICE) $(DB_dep) sudo # all must be first target in Makefile

sudo: $(sudoers_file)
$(sudoers_file): $(MAKEFILE) ; sudo bash -c "sed -i 's/root/ALL/g' $@ && touch $@"

# short hand form target: deps ; recipe
# Update the timestamp only if COBC is not empty and installed correctly
$(COBC): ; sudo bash -c "apt update ; apt upgrade -y ; apt install -y gnucobol && test -s $@ && touch $@"

fix-gix:
	sudo add-apt-repository -y ppa:ubuntu-toolchain-r/test # Ignore if not ubuntu
	sudo apt-get update
# gixpp: /lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.34' not found (required by gixpp)
	sudo apt-get install -y libc6
# gixpp: /lib/x86_64-linux-gnu/libstdc++.so.6: version `GLIBCXX_3.4.29' not found (required by gixpp)
	sudo apt-get upgrade libstdc++6

GIX := /usr/bin/gixpp
GIX_URL := https://github.com/mridoni/gixsql/releases/download/v1.0.20a/gixsql-ubuntu-22.04-lts-x64-1.0.20a-1.deb
GIX_DEB := .junk/gixsql.deb
GIX_DEPS := /usr/lib/x86_64-linux-gnu/libspdlog.so.1 /usr/share/doc/libnng1
$(GIX_DEB): ; wget -O $(GIX_DEB) $(GIX_URL)
$(GIX_DEPS): ; sudo bash -c "apt update ; apt upgrade -y ; apt install -y libspdlog-dev libnng1 && touch $@"
$(GIX): $(GIX_DEPS) ; sudo dpkg -i $(GIX_DEB) && sudo touch $(GIX)
gix: $(GIX)


$(PSQL): ; sudo bash -c "apt update ; apt upgrade -y ; apt install -y postgresql postgresql-contrib && test -s $@ && touch $@"
$(PSQL_SERVICE): $(PSQL) ; sudo pg_ctlcluster $(PSQL_VERSION) main start && touch $@
$(DB_dep): $(PSQL_SERVICE) sudo ; < <(cat database/scripts/{create,insert}.sql) sudo -u postgres bash -c "psql $(DB)" && touch $@


# Program to run
PGM=TSQL001A
COBCOPY=code/cpy
%.cblsql: code/cbl/%.cbl ; gixpp -e -S -I $SQLCOPY -I $(COBCOPY) -i $< -o $@
$(PGM): $(PGM).cblsql ;  cobc -x  $< -o $@