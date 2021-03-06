include lib/nall/Makefile.string
prefix = /usr/local
ui = ui_qt

################
### compiler ###
##########:######

platform=x
compiler=gcc
enable_gzip=false

ifneq ($(findstring gcc,$(compiler)),) # GCC family
  flags = -O2 -fomit-frame-pointer -Ilib
  libcoflags := $(flags) -static
  c = $(compiler)
  cpp = $(subst cc,++,$(compiler))
  obj = o
  rule = -c $< -o $@
  link = 
  mkbin = -o$1
  mkdef = -D$1
  mkincpath = -I$1
  mklib = -l$1
  mklibpath = -L$1
  # profile-guided optimization:
  #flags += -fprofile-generate
  #link += -lgcov
  #flags += -fprofile-use
  flags += -g -ggdb  -export-symbols
else ifeq ($(compiler),cl) # Visual C++
  flags = /nologo /wd4355 /wd4805 /wd4996 /Ox /GL /EHsc /Ilib
  libcoflags = $(flags)
  c = cl
  cpp = cl
  obj = obj
  rule = /c $< /Fo$@
  link = /link
  mkbin = /Fe$1
  mkdef = /D$1
  mkincpath = /I$1
  mklib = $1.lib
  mklibpath = /L$1
else
  unknown_compiler: help;
endif

##########
### os ###
##########


ifeq ($(platform),mac) #Mac
    ruby = video.glx video.xv video.sdl audio.alsa audio.openal audio.oss audio.pulseaudio audio.ao input.sdl input.x
    mkbin = -o $1
    delete = rm -f $1
    link += 	-L/opt/local/lib \
		-lSDL -framework,Cocoa \
		-lX11 \
    		-lm  \
		-framework Foundation\
		-framework Cocoa \
		-framework AGL \
    		-framework OpenAL\
		-framework ApplicationServices \
		-framework Foundation\
    		-framework OpenGL\
    		-framework GLUT \
		-lz \
		-lgl \
		-lao
    	

    flags +=-I/opt/local/include \
	-I/System/Library/Frameworks/OpenAL.framework/Versions/A/Headers/\
	-DVIDEO_SDL \
	-DINPUT_SDL \
	-DVIDEO_GLX \
	-DAUDIO_AO  \
	-DAUDIO_OPENAL 
endif

ifeq ($(platform),x) # X11
  ruby = video.glx video.xv video.sdl audio.alsa audio.openal audio.oss audio.pulseaudio audio.ao input.sdl input.x
  delete = rm -f $1

else ifeq ($(platform),win) # Windows
  mingw_link_flags = -mwindows
  # mingw_links_flags = -mconsole
  # enable static linking to Qt for Windows build
  mingw_link_flags += -enable-stdcall-fixup -Wl,-s -Wl,-enable-auto-import -Wl,-enable-runtime-pseudo-reloc
  ruby = video.direct3d video.wgl video.directdraw video.gdi audio.directsound input.rawinput input.directinput
  delete = $(if $(findstring i586-mingw-gcc,$(compiler)),rm -f $1,del $(subst /,\,$1))
  link += $(if $(findstring mingw,$(compiler)),$(mingw_link_flags))
  link += $(call mklib,uuid)
  link += $(call mklib,kernel32)
  link += $(call mklib,user32)
  link += $(call mklib,gdi32)
  link += $(call mklib,shell32)
endif

############
### ruby ###
############

rubyflags = $(if $(findstring .sdl,$(ruby)),`sdl-config --cflags`)
link += $(if $(findstring .sdl,$(ruby)),`sdl-config --libs`)

link += $(if $(findstring video.direct3d,$(ruby)),$(call mklib,d3d9))
link += $(if $(findstring video.directdraw,$(ruby)),$(call mklib,ddraw))
link += $(if $(findstring video.glx,$(ruby)),$(call mklib,GL))
link += $(if $(findstring video.wgl,$(ruby)),$(call mklib,opengl32))
link += $(if $(findstring video.xv,$(ruby)),$(call mklib,Xv))
link += $(if $(findstring audio.alsa,$(ruby)),$(call mklib,asound))
link += $(if $(findstring audio.ao,$(ruby)),$(call mklib,ao))
link += $(if $(findstring audio.directsound,$(ruby)),$(call mklib,dsound))
link += $(if $(findstring audio.openal,$(ruby)),$(if $(call streq,$(platform),x),$(call mklib,openal),$(call mklib,openal32)))
link += $(if $(findstring audio.pulseaudio,$(ruby)),$(call mklib,pulse-simple))
link += $(if $(findstring input.directinput,$(ruby)),$(call mklib,dinput8) $(call mklib,dxguid))
link += $(if $(findstring input.rawinput,$(ruby)),$(call mklib,xinput) $(call mklib,dinput8) $(call mklib,dxguid))

####################
### core objects ###
####################

objects = libco ruby libfilter string \
  reader cart cheat \
  memory smemory cpu scpu smp ssmp sdsp ppu bppu snes \
  bsx srtc sdd1 spc7110 cx4 dsp1 dsp2 dsp3 dsp4 obc1 st010 cmmio fatfs diskio

ifeq ($(enable_gzip),true)
  #objects += adler32 compress crc32 deflate gzio inffast inflate inftrees ioapi trees unzip zip zutil
  #flags += $(call mkdef,GZIP_SUPPORT)
endif

ifeq ($(enable_jma),true)
  #objects += jma jcrc32 lzmadec 7zlzma iiostrm inbyte lzma winout
  #flags += $(call mkdef,JMA_SUPPORT)
endif

######################
### implicit rules ###
######################

compile = \
  $(strip \
    $(if $(filter %.c,$<), \
      $(c) $(flags) $1 $(rule), \
      $(if $(filter %.cpp,$<), \
        $(cpp) $(flags) $1 $(rule) \
      ) \
    ) \
  )

%.$(obj): $<;	$(call compile)

all: build;

include $(ui)/Makefile

objects := $(patsubst %,obj/%.$(obj),$(objects))
rubydef := $(foreach c,$(subst .,_,$(call strupper,$(ruby))),$(call mkdef,$c))

#################
### libraries ###
#################

obj/ruby.$(obj): lib/ruby/ruby.cpp lib/ruby/* lib/ruby/video/* lib/ruby/audio/* lib/ruby/input/*
	$(call compile,$(rubydef) $(rubyflags))
obj/libco.$(obj): lib/libco/libco.c lib/libco/*
	$(c) $(libcoflags) $(rule)
obj/libfilter.$(obj): lib/libfilter/libfilter.cpp lib/libfilter/*
obj/string.$(obj): lib/nall/string.cpp lib/nall/*

#################
### utilities ###
#################

obj/reader.$(obj): reader/reader.cpp reader/*
obj/cart.$(obj)  : cart/cart.cpp cart/*
obj/cheat.$(obj) : cheat/cheat.cpp cheat/*

##############
### memory ###
##############

obj/memory.$(obj) : memory/memory.cpp memory/*
obj/smemory.$(obj): memory/smemory/smemory.cpp memory/smemory/* memory/smemory/mapper/*

###########
### cpu ###
###########

obj/cpu.$(obj) : cpu/cpu.cpp cpu/*
obj/scpu.$(obj): cpu/scpu/scpu.cpp cpu/scpu/* cpu/scpu/core/* cpu/scpu/dma/* cpu/scpu/memory/* cpu/scpu/mmio/* cpu/scpu/timing/*

###########
### smp ###
###########

obj/smp.$(obj) : smp/smp.cpp smp/*
obj/ssmp.$(obj): smp/ssmp/ssmp.cpp smp/ssmp/* smp/ssmp/core/* smp/ssmp/memory/* smp/ssmp/timing/*

###########
### dsp ###
###########

obj/adsp.$(obj): dsp/adsp/adsp.cpp dsp/adsp/*
obj/sdsp.$(obj): dsp/sdsp/sdsp.cpp dsp/sdsp/*

###########
### ppu ###
###########

obj/ppu.$(obj) : ppu/ppu.cpp ppu/*
obj/bppu.$(obj): ppu/bppu/bppu.cpp ppu/bppu/*

############
### snes ###
############

obj/snes.$(obj): snes/snes.cpp snes/* snes/scheduler/* snes/video/* snes/audio/* snes/input/*

#####################
### special chips ###
#####################

obj/bsx.$(obj)    : chip/bsx/bsx.cpp chip/bsx/*
obj/srtc.$(obj)   : chip/srtc/srtc.cpp chip/srtc/*
obj/sdd1.$(obj)   : chip/sdd1/sdd1.cpp chip/sdd1/*
obj/spc7110.$(obj): chip/spc7110/spc7110.cpp chip/spc7110/*
obj/cx4.$(obj)    : chip/cx4/cx4.cpp chip/cx4/*
obj/dsp1.$(obj)   : chip/dsp1/dsp1.cpp chip/dsp1/*
obj/dsp2.$(obj)   : chip/dsp2/dsp2.cpp chip/dsp2/*
obj/dsp3.$(obj)   : chip/dsp3/dsp3.cpp chip/dsp3/*
obj/dsp4.$(obj)   : chip/dsp4/dsp4.cpp chip/dsp4/*
obj/obc1.$(obj)   : chip/obc1/obc1.cpp chip/obc1/*
obj/st010.$(obj)  : chip/st010/st010.cpp chip/st010/*
obj/cmmio.$(obj)  : chip/cmmio/cmmio.cpp chip/cmmio/*
obj/fatfs.$(obj)  : chip/fatfs/fatfs.cpp chip/fatfs/*
obj/diskio.$(obj)  : chip/fatfs/diskio.cpp chip/fatfs/*




############
### zlib ###
############

obj/adler32.$(obj) : reader/zlib/adler32.c reader/zlib/*
obj/compress.$(obj): reader/zlib/compress.c reader/zlib/*
obj/crc32.$(obj)   : reader/zlib/crc32.c reader/zlib/*
obj/deflate.$(obj) : reader/zlib/deflate.c reader/zlib/*
obj/gzio.$(obj)    : reader/zlib/gzio.c reader/zlib/*
obj/inffast.$(obj) : reader/zlib/inffast.c reader/zlib/*
obj/inflate.$(obj) : reader/zlib/inflate.c reader/zlib/*
obj/inftrees.$(obj): reader/zlib/inftrees.c reader/zlib/*
obj/ioapi.$(obj)   : reader/zlib/ioapi.c reader/zlib/*
obj/trees.$(obj)   : reader/zlib/trees.c reader/zlib/*
obj/unzip.$(obj)   : reader/zlib/unzip.c reader/zlib/*
obj/zip.$(obj)     : reader/zlib/zip.c reader/zlib/*
obj/zutil.$(obj)   : reader/zlib/zutil.c reader/zlib/*

###########
### jma ###
###########

obj/jma.$(obj)    : reader/jma/jma.cpp reader/jma/*
obj/jcrc32.$(obj) : reader/jma/jcrc32.cpp reader/jma/*
obj/lzmadec.$(obj): reader/jma/lzmadec.cpp reader/jma/*
obj/7zlzma.$(obj) : reader/jma/7zlzma.cpp reader/jma/*
obj/iiostrm.$(obj): reader/jma/iiostrm.cpp reader/jma/*
obj/inbyte.$(obj) : reader/jma/inbyte.cpp reader/jma/*
obj/lzma.$(obj)   : reader/jma/lzma.cpp reader/jma/*
obj/winout.$(obj) : reader/jma/winout.cpp reader/jma/*

###############
### targets ###
###############

build: ui_build $(objects)
	g++ $(call mkbin,bsnes) $(objects) $(link)

install:
	install -D -m 755 ../bsnes $(DESTDIR)$(prefix)/bin/bsnes
	install -D -m 644 data/bsnes.png $(DESTDIR)$(prefix)/share/pixmaps/bsnes.png
	install -D -m 644 data/bsnes.desktop $(DESTDIR)$(prefix)/share/applications/bsnes.desktop

clean: ui_clean
	-@$(call delete,obj/*.$(obj))
	-@$(call delete,*.res)
	-@$(call delete,*.pgd)
	-@$(call delete,*.pgc)
	-@$(call delete,*.ilk)
	-@$(call delete,*.pdb)
	-@$(call delete,*.manifest)

help:
	@echo "Usage: $(MAKE) platform=(os) compiler=(cc) [options]"
	@echo ""
	@echo "Supported platforms:"
	@echo "  x   - Linux / BSD (x86, x86-64)"
	@echo "  win - Windows (x86, x86-64)"
	@echo ""
	@echo "Supported compilers:"
	@echo "  gcc              - GCC compiler"
	@echo "  mingw32-gcc      - MinGW compiler"
	@echo "  i586-mingw32-gcc - MinGW cross compiler"
	@echo "  cl               - Visual C++"
	@echo ""
	@echo "Available options:"
	@echo "  enable_gzip=[true|false] - Enable ZIP / GZ support (default=false)"
	@echo "  enable_jma=[true|false]  - Enable JMA support (default=false)"
	@echo ""
	@echo "Example: $(MAKE) platform=x compiler=gcc enable_gzip=true"
	@echo ""
