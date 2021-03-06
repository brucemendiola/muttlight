include config

## Options

ARCH:=x86_64
BUILD:=build
MACOSX_DEPLOYMENT_TARGET:=10.8
BUILD_MACHINE_OS_BUILD:=$(shell sw_vers -buildVersion)

CC:=$(XCODE_DEVELOPER_PATH)/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang
CFLAGS+=-Wall -Werror

CC_LDFLAGS+=-fobjc-link-runtime \
	 -Xlinker -export_dynamic \
	 -Xlinker -no_deduplicate \
	 -L$(BUILD)

LD:=$(XCODE_DEVELOPER_PATH)/Toolchains/XcodeDefault.xctoolchain/usr/bin/ld

OBJC:=-x objective-c -fobjc-arc -DOBJC_OLD_DISPATCH_PROTOTYPES=0

MACOSX:=-isysroot $(XCODE_DEVELOPER_PATH)/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.12.sdk \
       -mmacosx-version-min=$(MACOSX_DEPLOYMENT_TARGET)

IBTOOL:=$(XCODE_DEVELOPER_PATH)/usr/bin/ibtool
IBFLAGS+=--errors --warnings --notices

ACTOOL:=$(XCODE_DEVELOPER_PATH)/usr/bin/actool
ACFLAGS+=--notices --warnings

LSREGISTER:=/System/Library/Frameworks/CoreServices.framework/Frameworks/LaunchServices.framework/Support/lsregister

MUTT_LAUNCHER:=../mutt-launcher-app

EXECUTABLE_NAME:=Muttlight
PRODUCT_BUNDLE_IDENTIFIER:=org.tbrk.muttlight
PRODUCT_NAME:=Muttlight

BUNDLE_SHORT_VERSION:=1.0.0
BUNDLE_VERSION:=1

QL_EXECUTABLE_NAME:=muttlight-quicklook
QL_PRODUCT_BUNDLE_IDENTIFIER:=org.tbrk.muttlight.quicklook
QL_PRODUCT_NAME:=muttlight-quicklook

SL_EXECUTABLE_NAME:=muttlight-spotlight
SL_PRODUCT_BUNDLE_IDENTIFIER:=org.tbrk.muttlight.spotlight
SL_PRODUCT_NAME:=muttlight-spotlight

DSTDIR:=package/$(EXECUTABLE_NAME).app

## Mutt object files

OBJEXT:=o

am_mutt_OBJECTS := addrbook.$(OBJEXT) alias.$(OBJEXT) attach.$(OBJEXT) \
	base64.$(OBJEXT) browser.$(OBJEXT) buffy.$(OBJEXT) \
	color.$(OBJEXT) crypt.$(OBJEXT) cryptglue.$(OBJEXT) \
	commands.$(OBJEXT) complete.$(OBJEXT) compose.$(OBJEXT) \
	copy.$(OBJEXT) curs_lib.$(OBJEXT) curs_main.$(OBJEXT) \
	date.$(OBJEXT) edit.$(OBJEXT) enter.$(OBJEXT) flags.$(OBJEXT) \
	init.$(OBJEXT) filter.$(OBJEXT) from.$(OBJEXT) \
	getdomain.$(OBJEXT) group.$(OBJEXT) handler.$(OBJEXT) \
	hash.$(OBJEXT) hdrline.$(OBJEXT) headers.$(OBJEXT) \
	help.$(OBJEXT) hook.$(OBJEXT) keymap.$(OBJEXT) \
	mbox.$(OBJEXT) menu.$(OBJEXT) mh.$(OBJEXT) mx.$(OBJEXT) \
	pager.$(OBJEXT) parse.$(OBJEXT) pattern.$(OBJEXT) \
	postpone.$(OBJEXT) query.$(OBJEXT) recvattach.$(OBJEXT) \
	recvcmd.$(OBJEXT) rfc822.$(OBJEXT) rfc1524.$(OBJEXT) \
	rfc2047.$(OBJEXT) rfc2231.$(OBJEXT) rfc3676.$(OBJEXT) \
	score.$(OBJEXT) send.$(OBJEXT) sendlib.$(OBJEXT) \
	signal.$(OBJEXT) sort.$(OBJEXT) status.$(OBJEXT) \
	system.$(OBJEXT) thread.$(OBJEXT) charset.$(OBJEXT) \
	history.$(OBJEXT) lib.$(OBJEXT) muttlib.$(OBJEXT) \
	editmsg.$(OBJEXT) mbyte.$(OBJEXT) url.$(OBJEXT) \
	ascii.$(OBJEXT) crypt-mod.$(OBJEXT) safe_asprintf.$(OBJEXT)
MUTT_LIB_OBJECTS := pgp.$(OBJEXT) pgpinvoke.$(OBJEXT) pgpkey.$(OBJEXT) \
		   pgplib.$(OBJEXT) gnupgparse.$(OBJEXT) pgpmicalg.$(OBJEXT) \
		   pgppacket.$(OBJEXT) crypt-mod-pgp-classic.$(OBJEXT) \
		   smime.$(OBJEXT) crypt-mod-smime-classic.$(OBJEXT) \
		   resize.$(OBJEXT) mutt_idna.$(OBJEXT)

MUTT_LIBS:=-lintl -liconv

## Targets

.PHONY : app mainpackage qlpackage slpackage dmg

app: package qlpackage slpackage

APP_FILES=$(DSTDIR)/Contents/MacOS/Muttlight \
	  $(DSTDIR)/Contents/Resources/Base.lproj/MainMenu.nib \
	  $(DSTDIR)/Contents/Resources/AppIcon.icns \
	  $(DSTDIR)/Contents/Info.plist \
	  $(DSTDIR)/Contents/PkgInfo \
	  $(DSTDIR)/Contents/MacOS/Mutt\ Launcher.app

package: $(APP_FILES)
	@touch $(DSTDIR)/Contents/MacOS/Muttlight

$(BUILD)/fwrapdata.o: fwrapdata.c fwrapdata.h | $(BUILD)
	$(CC) -c $< -o $@ $(MACOSX)

$(BUILD)/entities.o: entities.c entities.h | $(BUILD)
	$(CC) -c $< -o $@ $(MACOSX)

# Install package

dmg: Muttlight-$(BUNDLE_SHORT_VERSION).dmg
Muttlight-$(BUNDLE_SHORT_VERSION).dmg: \
    		$(APP_FILES) $(QL_FILES) $(SL_FILES) \
		package/LICENSE package/Applications
	rm -f $@
	hdiutil create -fs HFS+ -volname Muttlight -srcfolder package $@

package/LICENSE:
	cp ../LICENSE $@

package/Applications:
	ln -s /Applications $@

# main app and user interface

MUTTLIGHT_OBJS:=$(BUILD)/AppDelegate.o \
    		$(BUILD)/main.o

$(DSTDIR)/Contents/MacOS/Muttlight: $(MUTTLIGHT_OBJS) \
    		| $(DSTDIR)/Contents/MacOS
	MACOSX_DEPLOYMENT_TARGET=$(MACOSX_DEPLOYMENT_TARGET) \
	$(CC) -fobjc-arc -arch $(ARCH) $(CFLAGS) $(MACOSX) \
		$(CC_LDFLAGS) -framework Cocoa $(MUTTLIGHT_OBJS) -o $@

$(DSTDIR)/Contents/Resources/Base.lproj/MainMenu.nib: Base.lproj/MainMenu.xib \
    		| $(DSTDIR)/Contents/Resources/Base.lproj
	XCODE_DEVELOPER_USR_PATH=$(XCODE_DEVELOPER_PATH)/usr \
	$(IBTOOL) $(IBFLAGS) \
	    --target-device mac \
	    --minimum-deployment-target $(MACOSX_DEPLOYMENT_TARGET) \
	    --output-partial-info-plist \
	    	$(BUILD)/MainMenu-PartialInfo.plist \
	    --output-format human-readable-text \
	    --compile $@ $<

$(DSTDIR)/Contents/Resources/AppIcon.icns: Assets.xcassets
	$(ACTOOL) \
	    --output-format human-readable-text \
	    --output-partial-info-plist \
	    	$(BUILD)/assetcatalog_generated_info.plist \
	    --app-icon AppIcon \
	    --enable-on-demand-resources NO \
	    --target-device mac \
	    --minimum-deployment-target $(MACOSX_DEPLOYMENT_TARGET) \
	    --platform macosx \
	    --product-type com.apple.product-type.application \
	    --compile $(DSTDIR)/Contents/Resources $<

$(DSTDIR)/Contents/Info.plist: Info.plist | $(DSTDIR)/Contents
	sed \
	  -e 's/$$(EXECUTABLE_NAME)/$(EXECUTABLE_NAME)/g' \
	  -e 's/$$(PRODUCT_BUNDLE_IDENTIFIER)/$(PRODUCT_BUNDLE_IDENTIFIER)/g' \
	  -e 's/$$(PRODUCT_NAME)/$(PRODUCT_NAME)/g' \
	  -e 's/$$(MACOSX_DEPLOYMENT_TARGET)/$(MACOSX_DEPLOYMENT_TARGET)/g' \
	  -e 's/$$(BUILD_MACHINE_OS_BUILD)/$(BUILD_MACHINE_OS_BUILD)/g' \
	  -e 's/$$(BUNDLE_SHORT_VERSION)/$(BUNDLE_SHORT_VERSION)/g' \
	  -e 's/$$(BUNDLE_VERSION)/$(BUNDLE_VERSION)/g' \
	  $< >$@

$(DSTDIR)/Contents/PkgInfo:
	printf 'APPLmutl' > $@

$(DSTDIR)/Contents/MacOS/Mutt\ Launcher.app: $(MUTT_LAUNCHER) \
	    	$(DSTDIR)/Contents/Resources/AppIcon.icns \
		| $(DSTDIR)/Contents/MacOS
	rm -rf "$@"
	cp -r "$<" "$@"
	cp $(DSTDIR)/Contents/Resources/AppIcon.icns \
	   $(DSTDIR)/Contents/MacOS/Mutt\ Launcher.app/Contents/Resources/appIcon.icns

$(BUILD)/main.o: main.m | $(BUILD)
$(BUILD)/AppDelegate.o: AppDelegate.m | $(BUILD)

# Mutt objects

$(BUILD)/mutt.o: | $(BUILD)
	$(LD) -r $(am_mutt_OBJECTS:%=$(MUTT_SRC)/%)  \
	      	 $(MUTT_LIB_OBJECTS:%=$(MUTT_SRC)/%) \
	      -o $@

## Quicklook plugin

QL:=./quicklook
QL_DSTDIR:=$(DSTDIR)/Contents/Library/QuickLook/muttlight.qlgenerator

QL_OBJS:=$(BUILD)/GenerateThumbnailForURL.o \
	 $(BUILD)/GeneratePreviewForURL.o \
	 $(BUILD)/qlmain.o \
	 $(BUILD)/muttiface.o \
	 $(BUILD)/fwrapdata.o \
	 $(BUILD)/entities.o \
	 $(BUILD)/mutt.o

QL_FILES=$(QL_DSTDIR)/Contents/MacOS/muttlight-quicklook \
	 $(QL_DSTDIR)/Contents/Info.plist \
	 $(QL_DSTDIR)/Contents/Resources/email.css

qlpackage: $(QL_FILES)
	@touch $(QL_DSTDIR)/Contents/MacOS/muttlight-quicklook

$(QL_DSTDIR)/Contents/MacOS/muttlight-quicklook: $(QL_OBJS) \
		| $(QL_DSTDIR)/Contents/MacOS
	MACOSX_DEPLOYMENT_TARGET=$(MACOSX_DEPLOYMENT_TARGET) \
	$(CC) -bundle -arch $(ARCH) $(CFLAGS) $(MACOSX) \
		$(CC_LDFLAGS) $(QL_OBJS) \
		-L$(ULOCAL)/lib $(MUTT_LIBS) -lncurses \
		-framework CoreServices -framework QuickLook \
		-o $@

$(QL_DSTDIR)/Contents/Info.plist: $(QL)/Info.plist \
    		| $(QL_DSTDIR)/Contents
	sed \
	  -e 's/$$(EXECUTABLE_NAME)/$(QL_EXECUTABLE_NAME)/g' \
	  -e 's/$$(PRODUCT_BUNDLE_IDENTIFIER)/$(QL_PRODUCT_BUNDLE_IDENTIFIER)/g' \
	  -e 's/$$(PRODUCT_NAME)/$(QL_PRODUCT_NAME)/g' \
	  -e 's/$$(MACOSX_DEPLOYMENT_TARGET)/$(MACOSX_DEPLOYMENT_TARGET)/g' \
	  -e 's/$$(BUILD_MACHINE_OS_BUILD)/$(BUILD_MACHINE_OS_BUILD)/g' \
	  -e 's/$$(BUNDLE_SHORT_VERSION)/$(BUNDLE_SHORT_VERSION)/g' \
	  -e 's/$$(BUNDLE_VERSION)/$(BUNDLE_VERSION)/g' \
	  $< >$@

$(QL_DSTDIR)/Contents/Resources/email.css: $(QL)/email.css \
	    	| $(QL_DSTDIR)/Contents/Resources
	cp $< $@

# for testing
$(BUILD)/muttiface: muttiface.c muttiface.h $(BUILD)/mutt.o \
    		$(BUILD)/fwrapdata.o $(BUILD)/entities.o
	$(CC) $(CFLAGS) -o $@ -DTESTING=1 -I $(MUTT_SRC) \
	    $< $(BUILD)/mutt.o $(BUILD)/fwrapdata.o $(BUILD)/entities.o \
	    -Wl,-dead_strip $(MUTT_LIBS) -lncurses \
	    -framework CoreFoundation

$(BUILD)/GenerateThumbnailForURL.o: $(QL)/GenerateThumbnailForURL.c | $(BUILD)
$(BUILD)/GeneratePreviewForURL.o: $(QL)/GeneratePreviewForURL.c | $(BUILD)
$(BUILD)/muttiface.o: CINCLUDE += -I $(MUTT_SRC) -I $(ULOCAL)/include
$(BUILD)/muttiface.o: muttiface.c muttiface.h | $(BUILD)
$(BUILD)/qlmain.o: $(QL)/qlmain.c | $(BUILD)

$(QL_DSTDIR): | $(DSTDIR)/Contents/Library
	mkdir -p $@
$(QL_DSTDIR)/Contents: | $(QL_DSTDIR)
	mkdir -p $@
$(QL_DSTDIR)/Contents/Resources: | $(QL_DSTDIR)/Contents
	mkdir -p $@
$(QL_DSTDIR)/Contents/MacOS: | $(QL_DSTDIR)/Contents
	mkdir -p $@

## Spotlight plugin

SL:=./spotlight
SL_DSTDIR:=$(DSTDIR)/Contents/Library/Spotlight/muttlight.mdimporter

SL_OBJS=$(BUILD)/GetMetadataForFile.o \
    	$(BUILD)/slmain.o \
	$(BUILD)/muttiface.o \
	$(BUILD)/fwrapdata.o \
	$(BUILD)/entities.o \
	$(BUILD)/mutt.o

SL_FILES=$(SL_DSTDIR)/Contents/MacOS/muttlight-spotlight \
	 $(SL_DSTDIR)/Contents/Info.plist

slpackage: $(SL_FILES)
	@touch $(SL_DSTDIR)/Contents/MacOS/muttlight-spotlight

$(SL_DSTDIR)/Contents/MacOS/muttlight-spotlight: $(SL_OBJS) \
    		$(BUILD)/mutt.o \
		| $(SL_DSTDIR)/Contents/MacOS
	MACOSX_DEPLOYMENT_TARGET=$(MACOSX_DEPLOYMENT_TARGET) \
	$(CC) -fobjc-arc -bundle -arch $(ARCH) $(CFLAGS) $(MACOSX) \
		$(CC_LDFLAGS) $(SL_OBJS) \
		-L$(ULOCAL)/lib $(MUTT_LIBS) -lncurses \
		-framework CoreServices \
		-o $@

$(SL_DSTDIR)/Contents/Info.plist: $(SL)/Info.plist \
    		| $(SL_DSTDIR)/Contents
	sed \
	  -e 's/$$(EXECUTABLE_NAME)/$(SL_EXECUTABLE_NAME)/g' \
	  -e 's/$$(PRODUCT_BUNDLE_IDENTIFIER)/$(SL_PRODUCT_BUNDLE_IDENTIFIER)/g' \
	  -e 's/$$(PRODUCT_NAME)/$(SL_PRODUCT_NAME)/g' \
	  -e 's/$$(MACOSX_DEPLOYMENT_TARGET)/$(MACOSX_DEPLOYMENT_TARGET)/g' \
	  -e 's/$$(BUILD_MACHINE_OS_BUILD)/$(BUILD_MACHINE_OS_BUILD)/g' \
	  -e 's/$$(BUNDLE_SHORT_VERSION)/$(BUNDLE_SHORT_VERSION)/g' \
	  -e 's/$$(BUNDLE_VERSION)/$(BUNDLE_VERSION)/g' \
	  $< >$@

$(BUILD)/GetMetadataForFile.o: CINCLUDE += -I $(MUTT_SRC) -I $(ULOCAL)/include
$(BUILD)/GetMetadataForFile.o: $(SL)/GetMetadataForFile.m | $(BUILD)
$(BUILD)/slmain.o: $(SL)/slmain.c | $(BUILD)

$(SL_DSTDIR): | $(DSTDIR)/Contents/Library
	mkdir -p $@
$(SL_DSTDIR)/Contents: | $(SL_DSTDIR)
	mkdir -p $@
$(SL_DSTDIR)/Contents/Resources: | $(SL_DSTDIR)/Contents
	mkdir -p $@
$(SL_DSTDIR)/Contents/MacOS: | $(SL_DSTDIR)/Contents
	mkdir -p $@

## Directories

$(BUILD):
	mkdir -p $@
$(DSTDIR):
	mkdir -p $@
$(DSTDIR)/Contents: | $(DSTDIR)
	mkdir -p $@
$(DSTDIR)/Contents/MacOS: | $(DSTDIR)/Contents
	mkdir -p $@
$(DSTDIR)/Contents/Library: | $(DSTDIR)/Contents
	mkdir -p $@
$(DSTDIR)/Contents/Resources: | $(DSTDIR)/Contents
	mkdir -p $@
$(DSTDIR)/Contents/Resources/Base.lproj: | $(DSTDIR)/Contents/Resources
	mkdir -p $@

## Administration

clean:
	@rm -rf $(BUILD)

realclean: distclean
distclean: clean
	@rm -rf $(DSTDIR) Muttlight-$(BUNDLE_SHORT_VERSION).dmg package

## Rules

.SUFFIXES : .m .o

$(BUILD)/%.o: %.m
	$(CC) $(OBJC) -arch $(ARCH) $(CFLAGS) $(CINCLUDE) $(MACOSX) -c $< -o $@

$(BUILD)/%.o: $(SL)/%.m
	$(CC) $(OBJC) -arch $(ARCH) $(CFLAGS) $(CINCLUDE) $(MACOSX) -c $< -o $@

$(BUILD)/%.o: %.c
	$(CC) -arch $(ARCH) $(CFLAGS) $(CINCLUDE) $(MACOSX) -c $< -o $@

$(BUILD)/%.o: $(QL)/%.c
	$(CC) -arch $(ARCH) $(CFLAGS) $(CINCLUDE) $(MACOSX) -c $< -o $@

$(BUILD)/%.o: $(SL)/%.c
	$(CC) -arch $(ARCH) $(CFLAGS) $(CINCLUDE) $(MACOSX) -c $< -o $@

