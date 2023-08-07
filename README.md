# emsim code m1
emsim code : copy from `/bgfs/tibrahim/hej24/birdcage_P31/freq_120/053023_increase_h/TTT_shield_073123_64ch_nostruct_noACRYLICS_v2_2/head_5out_pos/cap_57_5/trancap_no/code`

## Error
```
jin@Jins-Mac-mini code % make
  CC    auxiliary.o
clang: error: the clang compiler does not support '-march=native'
make: *** [auxiliary.o] Error 1
```

```
jin@Jins-Mac-mini code % clang --version
Apple clang version 12.0.5 (clang-1205.0.22.11)
Target: arm64-apple-darwin20.6.0
Thread model: posix
InstalledDir: /Library/Developer/CommandLineTools/usr/bin
```

## Fix
`clang: error: the clang compiler does not support '-march=native'`
https://stackoverflow.com/questions/65966969/why-does-march-native-not-work-on-apple-m1 

### attempt 1
change ` -march=native` to `-mcpu=apple-a14`
```CFLAGS=-std=gnu11 -g0 -O2 -Wall -Wextra -mcpu=apple-a14 -ftree-vectorize -mcmodel=medium ```
```
jin@Jins-Mac-mini code % make 
  CC    auxiliary.o
fatal error: error in backend: Only small, tiny and large code models are allowed on AArch64
clang: error: clang frontend command failed with exit code 70 (use -v to see invocation)
Apple clang version 12.0.5 (clang-1205.0.22.11)
Target: arm64-apple-darwin20.6.0
Thread model: posix
InstalledDir: /Library/Developer/CommandLineTools/usr/bin
clang: note: diagnostic msg: 
********************

PLEASE ATTACH THE FOLLOWING FILES TO THE BUG REPORT:
Preprocessed source(s) and associated run script(s) are located at:
clang: note: diagnostic msg: /var/folders/6y/q8hmgj_j5zx9_r13csqxvzh80000gn/T/auxiliary-72b666.c
clang: note: diagnostic msg: /var/folders/6y/q8hmgj_j5zx9_r13csqxvzh80000gn/T/auxiliary-72b666.sh
clang: note: diagnostic msg: Crash backtrace is located in
clang: note: diagnostic msg: /Users/jin/Library/Logs/DiagnosticReports/clang_<YYYY-MM-DD-HHMMSS>_<hostname>.crash
clang: note: diagnostic msg: (choose the .crash file that corresponds to your crash)
clang: note: diagnostic msg: 

********************
make: *** [auxiliary.o] Error 70
```

`-mcpu=apple-a14` or `-mcpu=apple-a11` not work. 

### attempt 2
upgrade clang
```
brew install llvm
echo 'export PATH="/opt/homebrew/opt/llvm/bin:$PATH"' >> ~/.zshrc
jin@Jins-Mac-mini code % clang --version                                                  
Apple clang version 12.0.5 (clang-1205.0.22.11)
Target: arm64-apple-darwin20.6.0
Thread model: posix
InstalledDir: /Library/Developer/CommandLineTools/usr/bin
```
upgrade clang failed.


### attempt 3
update OX software BigSur->Ventura
https://apple.stackexchange.com/questions/360009/how-to-install-clang-specifically 
`macOS Ventura Version 13.5`

```
jin@Jins-Mac-mini code % make
xcrun: error: invalid active developer path (/Library/Developer/CommandLineTools), missing xcrun at: /Library/Developer/CommandLineTools/usr/bin/xcrun
```

```
xcode-select --install
```

```
jin@Jins-Mac-mini code % make        
  CC    auxiliary.o
fatal error: error in backend: Only small, tiny and large code models are allowed on AArch64
clang: error: clang frontend command failed with exit code 70 (use -v to see invocation)
Apple clang version 14.0.3 (clang-1403.0.22.14.1)
Target: arm64-apple-darwin22.6.0
Thread model: posix
InstalledDir: /Library/Developer/CommandLineTools/usr/bin
clang: note: diagnostic msg: 
********************

PLEASE ATTACH THE FOLLOWING FILES TO THE BUG REPORT:
Preprocessed source(s) and associated run script(s) are located at:
clang: note: diagnostic msg: /var/folders/6y/q8hmgj_j5zx9_r13csqxvzh80000gn/T/auxiliary-cb3eaf.c
clang: note: diagnostic msg: /var/folders/6y/q8hmgj_j5zx9_r13csqxvzh80000gn/T/auxiliary-cb3eaf.sh
clang: note: diagnostic msg: Crash backtrace is located in
clang: note: diagnostic msg: /Users/jin/Library/Logs/DiagnosticReports/clang_<YYYY-MM-DD-HHMMSS>_<hostname>.crash
clang: note: diagnostic msg: (choose the .crash file that corresponds to your crash)
clang: note: diagnostic msg: 

********************
make: *** [auxiliary.o] Error 70
```

`clang` is `14.0.3`. probably have to check Xcode. 

### attempt 4
change `-mcmodel=medium` to `-mcmodel=large`

```
jin@Jins-Mac-mini code % make
  CC    auxiliary.o
  CC    bounds.o
  CC    calculations.o
  CC    dft_module.o
  CC    emsim.o
emsim.c:20:10: fatal error: 'argp.h' file not found
#include <argp.h>
         ^~~~~~~~
1 error generated.
make: *** [emsim.o] Error 1
```
probably related to https://github.com/yhg926/public_kssd/issues/2 


# run docker in m1
## 1. download docker 
https://docs.docker.com/desktop/install/mac-install/ 

## 2. check installation
```
docker --help
docker run --rm hello-world
docker image ls
```
## 3. pull docker image
```docker pull tsantini/image_processing```

## 4. check docker image
`docker scout quickview tsantini/image_processing` 

## 5. access docker image
```
jin@Jins-Mac-mini test_code % docker run -i -t --entrypoint='/bin/bash' --rm --security-opt=apparmor:unconfined -v /Users/jin/Desktop/emsim_code/test_code:/test_code tsantini/image_processing
root@e3d6f17d0c6a:~# cd test_code/code/
root@e3d6f17d0c6a:/test_code/code# make
root@e3d6f17d0c6a:/test_code/code# cp emsim ..
root@e3d6f17d0c6a:/test_code/code# make clean
root@e3d6f17d0c6a:/test_code# ./emsim --demo -i geometry -o demo_sim
```
