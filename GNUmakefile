include GNUmakefile.common

# This Makefile requires the following to be in your PATH
#   - Kaitai Struct (kaitai-struct-compiler)
#   - GNU make      (make.exe)
#   - GNU binutils  (objcopy.exe, e.g. MinGW)

KSC=cmd.exe /c kaitai-struct-compiler
KFLAGS=--debug --target javascript

# Input KSY definitions from Kaitai
ksydefs=$(wildcard formats/*/*.ksy)
formats=$(wildcard parsers/*.js)

all: parsers
	$(MAKE) kiewtai.hem command.exe

parsers: $(ksydefs)
	$(KSC) $(KFLAGS) -I formats -d parsers $^

parsers.h: parsers
	echo // THIS IS A GENERATED FILE > $@
ifeq ($(OS),Windows_NT)
	for %%f in ($(basename $(notdir $(wildcard parsers/*.js)))) do      \
	    echo extern char binary_parsers_%%f_js_start[];        >> $@ &  \
	    echo extern size_t binary_parsers_%%f_js_size[];       >> $@ &  \
	
	echo static KAITAI_PARSER KaitaiParsers[] = {   >> $@
	
	for %%f in ($(basename $(notdir $(wildcard parsers/*.js)))) do      \
	    echo { "%%f",                                          >> $@ &  \
	    echo binary_parsers_%%f_js_start,                      >> $@ &  \
	    echo (ULONG) ^&binary_parsers_%%f_js_size },           >> $@
	
	echo }; >> $@
	
	REM Special Case: Jpeg requires Exif
	copy parsers\\Exif.js + parsers\\Jpeg.js ExifJpegCombined.js
else
	for f in $(basename $(notdir $(wildcard parsers/*.js))); do         \
	    printf "extern char binary_parsers_%s_js_start[];\n" $${f};     \
	    printf "extern size_t binary_parsers_%s_js_size;\n" $${f};      \
	done >> $@
	
	printf "static KAITAI_PARSER KaitaiParsers[] = {\n" >> $@
	for f in $(basename $(notdir $(wildcard parsers/*.js))); do         \
	    printf "{ \"%s\"," $${f};                                       \
	    printf "binary_parsers_%s_js_start," $${f};                     \
	    printf "(ULONG) &binary_parsers_%s_js_size },\n" $${f};         \
	done >> $@
	
	printf "};\n" >> $@
	
	# Special Case: Jpeg requires Exif
	cat parsers/Exif.js parsers/Jpeg.js > ExifJpegCombined.js
endif
	$(MV) ExifJpegCombined.js parsers/Jpeg.js

kaitai.obj:     | parsers.h
kiewtai.obj:    | parsers.h

command.exe: kaitai.obj duktape.obj PolyFill.obj KaitaiStream.obj \
                pako_inflate.obj $(formats:.js=.obj)

kiewtai.dll: kaitai.obj duktape.obj PolyFill.obj KaitaiStream.obj \
                pako_inflate.obj $(formats:.js=.obj) hiewgate.obj

%.obj: %.js
	objcopy -I binary -O pe-i386 $^ $@

%.hem: %.dll
	$(CP) $< $@

clean::
	$(RMDIR) parsers
	$(RM) parsers.h *.hem

