TARGET = async-kqueue-idris

INSTALLDIR = ../lib

CPPFLAGS = -fPIC

MACHINE := $(shell $(CC) -dumpmachine)

ifneq (,$(findstring darwin,$(MACHINE)))
OS := darwin
else ifneq (,$(findstring netbsd,$(MACHINE)))
OS := netbsd
else ifneq (,$(findstring freebsd,$(MACHINE)))
OS := freebsd
else
$(error async-kqueue-idris is only supported on macOS and BSD systems)
endif

ifeq ($(OS),darwin)
LLD := $(shell command -v ld64.lld 2>/dev/null)
else
LLD := $(shell command -v ld.lld 2>/dev/null)
endif

ifeq ($(LLD),)
$(error LLD is required but was not found in PATH)
endif

LDFLAGS = --ld-path=$(LLD)

ifeq ($(OS),darwin)
DYLIB_WORKAROUND = cp "${INSTALLDIR}/${TARGET}" "${INSTALLDIR}/${TARGET}.dylib"
else
DYLIB_WORKAROUND = cp "${INSTALLDIR}/${TARGET}" "${INSTALLDIR}/${TARGET}.so"
endif

SRCS = $(wildcard *.c)
OBJS = $(SRCS:.c=.o)

all: $(TARGET)

$(TARGET): $(OBJS)
	$(CC) -shared $(LDFLAGS) -o $@ $^

%.o: %.c
	$(CC) $(CPPFLAGS) -c $< -o $@

.PHONY: clean
clean:
	rm -f $(OBJS) $(TARGET)
	rm -rf $(INSTALLDIR)

.PHONY: install
install: $(TARGET)
	mkdir -p $(INSTALLDIR)
	install $(TARGET) $(INSTALLDIR)
	$(DYLIB_WORKAROUND)
