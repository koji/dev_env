```
macOS: Big Sur version 11.2.3
CPU: Intel Core i9
package manager: Homebrew
```
# python

- Crash so many times
- Cannot install some software as well as macOS 10
In this article, I will show you how I could install python via `pyenv` on BigSur.
The following was the issue I had.

```zsh
$ pyenv install 3.7.6
python-build: use openssl@1.1 from homebrew
python-build: use readline from homebrew
Downloading Python-3.7.6.tar.xz...
-> https://www.python.org/ftp/python/3.7.6/Python-3.7.6.tar.xz
Installing Python-3.7.6...
python-build: use readline from homebrew
python-build: use zlib from xcode sdk

BUILD FAILED (OS X 11.0 using python-build 20180424)

Inspect or clean up the working tree at /var/folders/gj/x6v5vwdx1v7741fdfcxwmr100000gn/T/python-build.20200830033458.15319
Results logged to /var/folders/gj/x6v5vwdx1v7741fdfcxwmr100000gn/T/python-build.20200830033458.15319.log

Last 10 log lines:
    struct sf_hdtr sf;
           ^
./Modules/posixmodule.c:8401:15: error: implicit declaration of function 'sendfile' is invalid in C99 [-Werror,-Wimplicit-function-declaration]
        ret = sendfile(in, out, offset, &sbytes, &sf, flags);
              ^
clang -Wno-unused-result -Wsign-compare -Wunreachable-code -DNDEBUG -g -fwrapv -O3 -Wall -I/Applications/Xcode-beta.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk/usr/include   -I/Applications/Xcode-beta.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk/usr/include   -std=c99 -Wextra -Wno-unused-result -Wno-unused-parameter -Wno-missing-field-initializers -Wstrict-prototypes -Werror=implicit-function-declaration   -I. -I./Include -I/usr/local/opt/readline/include -I/usr/local/opt/readline/include -I/Users/koji/.pyenv/versions/3.7.6/include  -I/usr/local/opt/readline/include -I/usr/local/opt/readline/include -I/Users/koji/.pyenv/versions/3.7.6/include    -c ./Modules/pwdmodule.c -o Modules/pwdmodule.o
2 errors generated.
make: *** [Modules/posixmodule.o] Error 1
make: *** Waiting for unfinished jobs....
1 warning generated.
```

According to the wiki on GitHub, basically this issue will be solved by installing `zlib`.
So, I installed zlib via Homebrew

```zsh
$ brew install zlib
$ export LDFLAGS="-L/usr/local/opt/zlib/lib" 
$ export CPPFLAGS="-I/usr/local/opt/zlib/include
```
Maybe some of you will need to install the following.
```zsh
$ brew install readline xz
```

However, I got the same error again… I did some research on this issue.
I also tried reinstalling Xcode command-line tools, but it didn’t solve the issue…
Then finally, I figured out the solution.

There are 2 steps to resolve this issue.

## Step 1 Align command-line tools
In my case, I use Xcode-beta.app
I needed to check the version of command-line tools matches Xcode.

1. Open Xcode-beta.app
2. Go to Preference > Locations
3. Select the right version of command-line tools
(* I use Xcode-beta.app version 5 and command-line tools version 5)

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/74geagjrje1v6u346ua5.png)

## Step 2 Install python
In this case, I installed 3.8.0. If you want to install a different version, you will need to change the version in the following command.

```zsh
$ CFLAGS="-I$(brew --prefix openssl)/include -I$(brew --prefix bzip2)/include -I$(brew --prefix readline)/include -I$(xcrun --show-sdk-path)/usr/include" LDFLAGS="-L$(brew --prefix openssl)/lib -L$(brew --prefix readline)/lib -L$(brew --prefix zlib)/lib -L$(brew --prefix bzip2)/lib" pyenv install --patch 3.8.0 < <(curl -sSL https://github.com/python/cpython/commit/8ea6353.patch\?full_index\=1)
```

```zsh
$ pyenv versions
* system (set by /Users/koji/.pyenv/version)
  3.8.0
```


### precondition
- Installed Homebrew

### install packages with brew
```zsh
brew install zlib
brew install sqlite
brew install bzip2
brew install libiconv
brew install libzip
```

### Install python 3.4.10
```zsh
$ LDFLAGS="-L$(brew --prefix zlib)/lib -L$(brew --prefix bzip2)/lib" pyenv install --patch 3.4.10 < <(curl -sSL https://github.com/python/cpython/commit/8ea6353.patch\?full_index\=1)
```

#### Install python 3.5.7
```zsh
$ LDFLAGS="-L$(brew --prefix zlib)/lib -L$(brew --prefix bzip2)/lib" pyenv install --patch 3.5.7 < <(curl -sSL https://github.com/python/cpython/commit/8ea6353.patch\?full_index\=1)
```

#### Install python 3.6.5
```zsh
$ LDFLAGS="-L$(brew --prefix zlib)/lib -L$(brew --prefix bzip2)/lib" pyenv install --patch 3.6.5 < <(curl -sSL https://github.com/python/cpython/commit/8ea6353.patch\?full_index\=1)
```
I also tried to install 3.6.9 and worked well(just changed the version).


### Install python 3.7.3
I tried to install 3.7.6, 3.7.7, 3.7.8, and 3.7.9, but couldn't install them. Only 3.7.3 worked with the following.

```zsh
$ CFLAGS="-I$(brew --prefix openssl)/include -I$(brew --prefix bzip2)/include -I$(brew --prefix readline)/include -I$(xcrun --show-sdk-path)/usr/include" LDFLAGS="-L$(brew --prefix openssl)/lib -L$(brew --prefix readline)/lib -L$(brew --prefix zlib)/lib -L$(brew --prefix bzip2)/lib" pyenv install --patch 3.7.3 < <(curl -sSL https://github.com/python/cpython/commit/8ea6353.patch\?full_index\=1)
```

```zsh
$ pyenv versions
  system
  3.4.10
  3.5.7
  3.6.5
  3.7.3
  3.8.3
```


