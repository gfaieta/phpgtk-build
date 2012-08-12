PHP 5.3
=======

PHP build steps taken from [https://wiki.php.net/internals/windows/stepbystepbuild](https://wiki.php.net/internals/windows/stepbystepbuild)

## PHP

#### 00)
Install Windows7 32bit

#### 01)
Install Microsoft VisualC++ 2008 Express (`VS2008ExpressWithSP1ENUX1504728.iso`)

#### 02)
Install Microsoft Windows SDK 6.1 (`6.0.6001.18000.367-KRMSDK_EN.iso`)

#### 03)
create the directory `c:\php-sdk`

#### 04)
unpack the binary-tools.zip archive (`php-sdk-binary-tools-20110915.zip`) from 
[http://windows.php.net/downloads/php-sdk/](http://windows.php.net/downloads/php-sdk/)
into this directory, there should be one sub-directory called `bin` and one 
called `script`

#### 05)
open the windows sdk 6.1 shell (Start -> All Programs -> Microsoft Windows SDK v6.1
-> CMD Shell) and execute the following commands in it:

    setenv /x86 /xp /release
    cd c:\php-sdk\
    bin\phpsdk_setvars.bat
    bin\phpsdk_buildtree.bat php53

#### 06)
download and extract php sources (currently version is `php-5.3.15-src.zip`) to
`C:\php-sdk\php53\vc9\x86` so that the following directory gets created:
`C:\php-sdk\php53\vc9\x86\php-5.3.15-src`

#### 07)
download the prepackaged deps library `deps-5.4-vc9-x86.7z` from 
[http://windows.php.net/downloads/php-sdk/](http://windows.php.net/downloads/php-sdk/)
and extract to `C:\php-sdk\php5\vc9\x86` (merge with existing `deps` directory)
The deps directory in `C:\php-sdk\php53\vc9\x86` should contain all required libraries
(see [http://wiki.php.net/internals/windows/libs](http://wiki.php.net/internals/windows/libs)).

#### 08)
run in the windows-sdk-shell:

    cd C:\php-sdk\php53\vc9\x86\php-5.3.15-src
    buildconf
    configure --with-gd --enable-cli --disable-zts --enable-cli-win32
    nmake
    nmake build-devel

#### 09)
test the newly built PHP

    cd C:\php-sdk\php53\vc9\x86\php-5.3.15-src\Release
    php -v
    php -m


## php_cairo

#### 10)
extract `gtk+-bundle_2.24.10-20120208_win32.zip` in `c:\gtk` (download from 
[http://ftp.gnome.org/pub/gnome/binaries/win32/gtk+/2.24/](http://ftp.gnome.org/pub/gnome/binaries/win32/gtk+/2.24/))

#### 11)
Copy overwriting existing files:

    c:\gtk\include\cairo               to  C:\php-sdk\php53\vc9\x86\deps\include\cairo
    c:\gtk\include\fontconfig          to  C:\php-sdk\php53\vc9\x86\deps\include\fontconfig
    c:\gtk\include\freetype2\freetype  to  C:\php-sdk\php53\vc9\x86\deps\include\freetype
    c:\gtk\include\ft2build.h          to  C:\php-sdk\php53\vc9\x86\deps\include\ft2build.h

copy the following files from `c:\gtk\lib\`  into  `C:\php-sdk\php53\vc9\x86\deps\lib\`

    cairo.def
    cairo.lib
    fontconfig.def
    fontconfig.lib
    freetype.def
    freetype.lib
    libcairo.dll.a
    libfontconfig.dll.a
    libfreetype.dll.a
 
#### 12)
download php_cairo source (I used 
[https://github.com/gtkforphp/cairo](https://github.com/gtkforphp/cairo) 
commit: df45aa1418) and extract to `C:\php-sdk\php53\vc9\x86\` and rename 
extracted directory to `cairo`

`C:\php-sdk\php53\vc9\x86\` should now contain 3 subdirs: `cairo`, `deps` 
and `php-5.3.15-src`

#### 13)
run in the windows-sdk-shell:

    cd C:\php-sdk\php53\vc9\x86\cairo       
    ..\php-5.3.15-src\Release\php-5.3.15-devel-VC9-x86\phpize.bat
    configure --with-cairo

#### 14)
Open the generated Makefile (`C:\php-sdk\php53\vc9\x86\cairo\Makefile`)
with a text-editor.  Find the first line beginning with `CFLAGS=` (should be at 
line 49) and append the following definitions:

    /D HAVE_FREETYPE=1 /D HAVE_WIN32_FONT=1 /D HAVE_CAIRO=1 /D HAVE_STRNLEN=1

Find the second line beginning with `CFLAGS=` (should be at line 81) and
delete the string (it's near the end of the line):

    /D ZTS=1

Then append:
    
    /I "C:\php-sdk\php53\vc9\x86\deps\include" /D HAVE_FREETYPE=1 /D HAVE_WIN32_FONT=1 /D HAVE_CAIRO=1 /D HAVE_STRNLEN=1
    

#### 15)
Open `C:\php-sdk\php53\vc9\x86\deps\include\fontconfig\fontconfig.h`
with a text-editor. Find the line that says:

    #include <unistd.h>

and replace with:
    
    #ifdef PHP_WIN32
    # include "win32/unistd.h"
    #else
    # include <unistd.h>
    #endif    

#### 16)
run in the windows-sdk-shell:

    nmake
 
#### 17)
copy `C:\php-sdk\php53\vc9\x86\cairo\Release\php_cairo.dll` to
`C:\php-sdk\php53\vc9\x86\php-5.3.15-src\Release\ext\php_cairo.dll`

copy the following files from `C:\gtk\bin` to `C:\php-sdk\php53\vc9\x86\php-5.3.15-src\Release`

    libcairo-2.dll
    libexpat-1.dll
    libfontconfig-1.dll
    freetype6.dll
    libpng14-14.dll
    zlib1.dll
       

#### 18)
Create a `php.ini` in `C:\php-sdk\php53\vc9\x86\php-5.3.15-src\Release` with 
the following content:

    extension_dir=.\ext
    extension=php_cairo.dll
       
#### 19)
verify that the extension is loaded

    cd C:\php-sdk\php53\vc9\x86\php-5.3.15-src\Release
    php -m


## php_gtk2


#### 20)
get `grep` and `sed` for windows (for example: 
[http://unxutils.sourceforge.net/UnxUtils.zip](http://unxutils.sourceforge.net/UnxUtils.zip))
and put them in the `PATH` (ex. `c:\windows`)

#### 21)
extract php-gtk sources to `c:\php-gtk` (I used 
[https://github.com/auroraeosrose/php-gtk-src](https://github.com/auroraeosrose/php-gtk-src)
commit: 4cda109cf9)
                
#### 22)
We need a `php.exe`, so add `C:\php-sdk\php53\vc9\x86\php-5.3.15-src\Release` to 
`PATH` in `C:\Program Files\Microsoft Visual Studio 9.0\Common7\Tools\vsvars32.bat`

#### 23)
In  `C:\Program Files\Microsoft Visual Studio 9.0\Common7\Tools\vsvars32.bat`

    add to INCLUDE  C:\php-sdk\php53\vc9\x86\php-5.3.15-src
    add to INCLUDE  C:\php-sdk\php53\vc9\x86\deps\include
    add to LIB      C:\php-sdk\php53\vc9\x86\php-5.3.15-src\Release
    add to LIB      C:\php-sdk\php53\vc9\x86\deps\lib

#### 24)
    mkdir c:\php_build

#### 25)
extract
[http://www.php.net/extra/win32build.zip](http://www.php.net/extra/win32build.zip)
into `c:\php_build`
                              
#### 26)
copy from `c:\gtk` to `c:\php_build`

    include\cairo                     -> include\cairo
    include\gtk-2.0\*                 -> include\*
    include\glib-2.0\*                -> include\*
    include\atk-1.0\atk               -> include\atk
    include\pango-1.0\pango           -> include\pango
    include\gdk-pixbuf-2.0\gdk-pixbuf -> include\gdk-pixbuf
    include\fontconfig                -> include\fontconfig
    include\freetype2\freetype        -> include\freetype

    lib\*  -> lib\*

Overwrite conflicting files

#### 27)
copy 

    c:\php_build\lib\glib-2.0\include\*   -> c:\php_build\include\*
    c:\php_build\lib\gtk-2.0\include\*    -> c:\php_build\include\*

#### 28)
In  `C:\Program Files\Microsoft Visual Studio 9.0\Common7\Tools\vsvars32.bat`

    add to INCLUDE  C:\php_build\include
    add to LIB      C:\php_build\lib
and execute the batch, in the windows sdk shell, to activate the changes


#### 29) ***** HACK *****
copy `C:\Program Files\Microsoft SDKs\Windows\v6.1\Samples\winui\TSF\tsfapp\winres.h`
to `c:\php_build\include`

#### 30)
copy `php_cairo_api.h` and  `php_cairo.h` 
from `C:\php-sdk\php53\vc9\x86\cairo` to `c:\php_build\include`

#### 31)
copy `php_cairo.dll.res`,  `php_cairo.exp` and `php_cairo.lib`
from `C:\php-sdk\php53\vc9\x86\cairo\Release` to `c:\php_build\lib`

#### 32)
run in the windows-sdk-shell:

    buildconf --with-phpize=c:\php-sdk\php53\vc9\x86\php-5.3.15-src\Release\php-5.3.15-devel-VC9-x86\phpize.bat
    configure --with-php-build=..\php_build --disable-zts --enable-gd
    nmake

#### 33)
copy `C:\php-gtk\Release\php_gtk2.dll` to
`C:\php-sdk\php53\vc9\x86\php-5.3.15-src\Release\ext\php_gtk2.dll`

copy the following files from `C:\gtk\bin`
to `C:\php-sdk\php53\vc9\x86\php-5.3.15-src\Release`

    libgtk-win32-2.0-0.dll
    libgdk-win32-2.0-0.dll
    libgdk_pixbuf-2.0-0.dll
    intl.dll
    libgio-2.0-0.dll
    libglib-2.0-0.dll
    libgmodule-2.0-0.dll
    libgobject-2.0-0.dll
    libgthread-2.0-0.dll
    libpango-1.0-0.dll
    libpangocairo-1.0-0.dll
    libpangoft2-1.0-0.dll
    libpangowin32-1.0-0.dll
    libatk-1.0-0.dll

#### 34)
add the following line to `C:\php-sdk\php53\vc9\x86\php-5.3.15-src\Release\php.ini`

    extension=php_gtk2.dll 

#### 35)
verify that the extension is loaded

    cd C:\php-sdk\php53\vc9\x86\php-5.3.15-src\Release
    php -m


## DONE!