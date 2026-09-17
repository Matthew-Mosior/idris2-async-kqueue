TARGET = async-kqueue-idris

INSTALLDIR = ../lib

LDFLAGS =
CPPFLAGS = -fPIC

CC_VERSION = $(shell $(CC) --version)
MACHINE := $(shell $(CC) -dumpmachine)

ifneq (,$(findstring darwin,$(MACHINE)))
	OS := darwin
else ifneq (,$(findstring bsd,$(MACHINE)))
	OS := bsd
else
	$(error async-kqueue-idris is only supported on macOS and BSD systems)
endif

ifeq ($(findstring Apple clang,$(CC_VERSION)),Apple clang)
	# Use Apple's system linker on macOS.
else
	LDFLAGS += -fuse-ld=lld
endif

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
