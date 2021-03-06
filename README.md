# Il2CppParser  
libil2cpp runtime symbol parser for ida,  
do not need global-metadata.dat.  
  
![example](./example.png)  

# Useage  

## prepare  

### find libil2cpp  
firstly you should find target's internal libil2cpp headerfile version,  
by analsys libil2cpp.so or try every finded version by github or etc.  
you can run `scripts/find_il2cpp_addrs.py` without libil2cpp to determine internal structs and it's size,  
which will be helpful for find right libil2cpp version.  
once you find it, replace libil2cpp.  
if your finded version use `class-internals.h` and `object-internals.h`  
instead of `il2cpp-class-internals.h` and `il2cpp-object-internals.h`,  
you can just replace them in `src/parser.cpp`.  

### fix libil2cpp for ida  
copy libil2cpp to another path for ida load only;  
add `__ANDROID__=1;__arm__=1;` to `Options->Compiler... Predefined macros`;  
add `{your_ndk_toolchain_path}/sysroot/usr/include` to `Options->Compiler... include`;  
add libil2cpp for ida to `Options->Compiler... include`;  
remove all include <...> from il2cpp-class-internals.h and il2cpp-object-internals.h,  
try `File->Load file->Parse C header file...` to load il2cpp-class-internals.h and il2cpp-object-internals.h,  
if error happend, just locate and fix or remove it.  
il2cpp_fix_diff is a example diff for fix.  

### install sark  
see https://github.com/tmr232/Sark   
  
## make
edit CMakeLists.txt, setup ndk toolchain path;  
run `mkdir build; cd build; cmake ..;make`;  
push build/libparser.so on your device;  
  
## use  
open libil2cpp.so(arm) with ida;  
Click `View->Open subviews->Strings` to generate strings's xref info;  
wait for ida's thinking is done(sometimes it take a while)  
as find_il2cpp_addrs.py will use xref info of some function/global_addr;  
Click `File->Script file...` or press `Alt+F7`, run `scripts/find_il2cpp_addrs.py`,  
`find_il2cpp_addrs.py` will generate `il2cpp_addrs.json` at same path with libil2cpp.so;  
push `il2cpp_addrs.json` on your device;  
  
open app, let it load libparser.so, and run these after unity has started:  

    libparser.init(il2cpp_addrs.json);  
    libparser.dumpAll(jsonoutpath, headeroutpath);  
for frida, you can use `scripts/frida_il2cpp.js` at this step.  
if app crashed at this step, the libil2cpp's version may be wrong, try another.  

pull headerfile from headeroutpath, put it in libil2cpp for ida,  
Click File->Load file->Parse C header file... or press Ctrl+F9, load headerfile.  

pull jsonfile from jsonoutpath, put it at same path with libil2cpp.so,rename it to `output.json`;  
run `scripts/load_symbols.py`, it may take a long time.  

as ida's function and strings search may be very slow after loaded lot's of symbols,  
you can copy `scripts/symbols_search.py` to same path with `output.json`, open python shell there,  
run `from symbols_search import *`, then  
use `sf("kword1 kword2 ...")` to search function or class's name,  
use `ss("kword1 kword2 ...")` to search in strings.  

then enjoy your reversing with full structs and symbols info.  
