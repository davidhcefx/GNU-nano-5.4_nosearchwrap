# GNU nano (5.4) - nosearchwrap
[![build](https://github.com/davidhcefx/GNU_nano_5.4_nosearchwrap/actions/workflows/build.yml/badge.svg)](https://github.com/davidhcefx/GNU_nano_5.4_nosearchwrap/actions/workflows/build.yml)

- While performing textual search, many editors, for instance [Sublime](https://www.sublimetext.com/), support toggling the option **Wrap**, which is to wrap past end-of-file and proceed searching. It will be nice if Nano can have this too!


### New rcfile option:
```
set nosearchwrap
    Don't wrap around to the start or end of the buffer when performing search or replace.
```

### New program option:
```
--nosearchwrap
    Don't wrap around to the start or end of the buffer when performing search or replace.
```

## How to compile

*Please refer to [README.GIT](/README.GIT) for more details.*

1. Install the dependencies: `apt install autoconf automake autopoint gcc gettext git groff make pkg-config texinfo`

2. Run `./autogen.sh`.

3. After that, just do the normal `./configure`, `make` and `make install`.


## Modification Summary
```diff
diff --git a/doc/nano.1 b/doc/nano.1
index a759f359..7e3c5316 100644
--- a/doc/nano.1
+++ b/doc/nano.1
@@ -340,6 +340,9 @@ filename in the center of the title bar.
 .BR \-! ", " \-\-magic
 When neither the file's name nor its first line give a clue,
 try using libmagic to determine the applicable syntax.
+.TP
+.BR \-\-nosearchwrap
+Don't wrap around to the start or end of the buffer when performing search or replace.
 
 .SH TOGGLES
 Several of the above options can be switched on and off also while
diff --git a/doc/nanorc.5 b/doc/nanorc.5
index 5716a4b1..a383d7c4 100644
--- a/doc/nanorc.5
+++ b/doc/nanorc.5
@@ -209,6 +209,9 @@ Don't automatically add a newline when a text does not end with one.
 .B set nopauses
 Obsolete option.  Ignored.
 .TP
+.B set nosearchwrap
+Don't wrap around to the start or end of the buffer when performing search or replace.
+.TP
 .B set nowrap
 Deprecated option since it has become the default setting.
 When needed, use \fBunset breaklonglines\fR instead.
diff --git a/src/definitions.h b/src/definitions.h
index 7924f8ab..f1302ec6 100644
--- a/src/definitions.h
+++ b/src/definitions.h
@@ -348,7 +348,8 @@ enum {
 	INDICATOR,
 	BOOKSTYLE,
 	STATEFLAGS,
-	USE_MAGIC
+	USE_MAGIC,
+	NO_SEARCH_WRAP
 };
 
 /* Structure types. */
diff --git a/src/nano.c b/src/nano.c
index 4c97375a..74c56faf 100644
--- a/src/nano.c
+++ b/src/nano.c
@@ -639,6 +639,7 @@ void usage(void)
 #ifdef HAVE_LIBMAGIC
 	print_opt("-!", "--magic", N_("Also try magic to determine syntax"));
 #endif
+	print_opt("", "--nosearchwrap", N_("Don't wrap past EOF when search/replace"));
 }
 
 /* Display the version number of this nano, a copyright notice, some contact
@@ -1744,6 +1745,7 @@ int main(int argc, char **argv)
 		{"unix", 0, NULL, 'u'},
 		{"afterends", 0, NULL, 'y'},
 		{"stateflags", 0, NULL, '%'},
+		{"nosearchwrap", 0, NULL, '\x01'},
 #endif
 #ifdef HAVE_LIBMAGIC
 		{"magic", 0, NULL, '!'},
@@ -2044,6 +2046,9 @@ int main(int argc, char **argv)
 				SET(USE_MAGIC);
 				break;
 #endif
+			case 0x01:
+				SET(NO_SEARCH_WRAP);
+				break;
 			default:
 				printf(_("Type '%s -h' for a list of available options.\n"), argv[0]);
 				exit(1);
diff --git a/src/nano.exe b/src/nano.exe
new file mode 100755
index 00000000..c70b3f2e
Binary files /dev/null and b/src/nano.exe differ
diff --git a/src/rcfile.c b/src/rcfile.c
index ef410369..dfaa9d43 100644
--- a/src/rcfile.c
+++ b/src/rcfile.c
@@ -111,6 +111,7 @@ static const rcoption rcopts[] = {
 	{"locking", LOCKING},
 	{"matchbrackets", 0},
 	{"noconvert", NO_CONVERT},
+	{"nosearchwrap", NO_SEARCH_WRAP},
 	{"showcursor", SHOW_CURSOR},
 	{"smarthome", SMART_HOME},
 	{"smooth", SMOOTH_SCROLL},  /* Deprecated; remove in 2021. */
diff --git a/src/search.c b/src/search.c
index c0c89804..8ac9452a 100644
--- a/src/search.c
+++ b/src/search.c
@@ -243,7 +243,7 @@ int findnextstr(const char *needle, bool whole_word_only, int modus,
 		/* If we've reached the start or end of the buffer, wrap around;
 		 * but stop when spell-checking or replacing in a region. */
 		if (line == NULL) {
-			if (whole_word_only || modus == INREGION) {
+			if (whole_word_only || modus == INREGION || ISSET(NO_SEARCH_WRAP)) {
 				nodelay(edit, FALSE);
 				return 0;
 			}
diff --git a/syntax/nanorc.nanorc b/syntax/nanorc.nanorc
index 22eb9fcb..191b80fb 100644
--- a/syntax/nanorc.nanorc
+++ b/syntax/nanorc.nanorc
@@ -7,7 +7,7 @@ comment "#"
 color brightred ".*"
 
 # Keywords
-color brightgreen "^[[:space:]]*(set|unset)[[:space:]]+(afterends|allow_insecure_backup|atblanks|autoindent|backup|boldtext|breaklonglines|casesensitive|constantshow|cutfromcursor|emptyline|historylog|indicator|jumpyscrolling|linenumbers|locking|magic|mouse|multibuffer|noconvert|nohelp|nonewlines|positionlog|preserve|quickblank|rawsequences|rebinddelete|regexp|saveonexit|showcursor|smarthome|softwrap|stateflags|suspendable|tabstospaces|trimblanks|unix|wordbounds|zap)\>"
+color brightgreen "^[[:space:]]*(set|unset)[[:space:]]+(afterends|allow_insecure_backup|atblanks|autoindent|backup|boldtext|breaklonglines|casesensitive|constantshow|cutfromcursor|emptyline|historylog|indicator|jumpyscrolling|linenumbers|locking|magic|mouse|multibuffer|noconvert|nohelp|nosearchwrap|nonewlines|positionlog|preserve|quickblank|rawsequences|rebinddelete|regexp|saveonexit|showcursor|smarthome|softwrap|stateflags|suspendable|tabstospaces|trimblanks|unix|wordbounds|zap)\>"
 color yellow "^[[:space:]]*set[[:space:]]+((error|function|key|number|prompt|scroller|selected|status|stripe|title)color)[[:space:]]+(bold,)?(italic,)?(bright|light)?(white|black|red|blue|green|yellow|magenta|cyan|normal|pink|purple|mauve|lagoon|mint|lime|peach|orange|latte)?(,(light)?(white|black|red|blue|green|yellow|magenta|cyan|normal|pink|purple|mauve|lagoon|mint|lime|peach|orange|latte))?\>"
 color brightgreen "^[[:space:]]*set[[:space:]]+(backupdir|brackets|errorcolor|functioncolor|keycolor|matchbrackets|numbercolor|operatingdir|promptcolor|punct|quotestr|scrollercolor|selectedcolor|speller|statuscolor|stripecolor|titlecolor|whitespace|wordchars)[[:space:]]+"
 color brightgreen "^[[:space:]]*set[[:space:]]+(fill[[:space:]]+-?[[:digit:]]+|(guidestripe|tabsize)[[:space:]]+[1-9][0-9]*)\>"
```
