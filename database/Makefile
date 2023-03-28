DB=gixsql
BUILD=$(CURDIR)/build.sql
SCRIPTS=$(CURDIR)/scripts
MASTER=$(SCRIPTS)/create.sql
INSERT = $(SCRIPTS)/insert.sql
DROP = $(SCRIPTS)/drop.sql

.PHONY: drop clean

all: insert
	psql $(DB) -f $(BUILD)

create:
	@cat $(MASTER) >> $(BUILD)

insert: create
	@cat $(INSERT) >> $(BUILD)

drop:
	psql $(DB) -f $(DROP)

clean:
	@rm -rf $(BUILD)
