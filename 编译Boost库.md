### 编译Boost库

#### 一 生成Boost编译工具

​	多数库是不需要预先编译的，include hpp文件就能用。如果出现链接失败，那就是可能需要编译库了。

boost自带一套编译工具bjam，bjam本身是跨平台的，并且也要自行编译出来。在boost目录下有bootstrap.sh和bootstrap.bat两个脚本分别用来编译Unix和windows下的bjam。bootstrap脚本可以传入参数，以在编译bjam过程中生成特定的编译boost的配置。这些配置保存在新生成的project-config.jam里，但还可以在运行bjam的时候再传入参数来覆盖。同时生成的b2是bjam的代理，运行哪个的效果都差不多。 
在终端下运行 
`bjam --show-libraries` 
会列出所有要编译的库。 
真正编译时，可以传入–with-xxx来选择编译哪些库，或者传入–without-xxx来选择不编译哪些库。如果不传则会读取project-config.jam的设置，如果也没有则是编译全部的库。 
更多的参数可以用 
`bjam --help` 
来查看。例如编译成静态库还是动态库，运行时库是静态的还是动态的，编译完后要不要安装等。

​	打开vs自带的cmd，切换目录到Boost根目录并执行命令

```
>cd .../boost_1_63_0
>booststrap.bat
```

​	执行成功后会生成b2.exe和bjam.exe两个编译程序。

#### 二 使用bjam.exe编译Boost库

```
>bjam.exe stage --stagedir="vc12" --toolset=msvc-12.0 link=static runtime-link=shared threading=multi debug address-model=64 --with-graph --without-wave
```

​	参数介绍：

- **stage/install：** stage表示只生成库（lib和dll），install还会生成包含头文件的include目录。一般使用stage。

- **--toolset**：指定编译器。msvc-12.0为vs2013编译器。

- **--without/with**：选择编译指定的库。

- **--build-type=complete**：编译所有版本。一般不需要，只编译自己需要的版本。

- **--stagedir/prefix**：stage时使用stagedir，install时使用prefix，表示编译生成文件的路径。推荐给不同的IDE指定不同的目录。如VS2008对应的是E:\SDK\boost\bin\vc9，VC6对应的是E:\SDK\boost\bin\vc6，否则都生成到一个目录下面，难以管理。如果使用了install参数，那么还将生成头文件目录，vc9对应的就是E:\SDK\boost\bin\vc9\include\boost-1_46\boost,vc6类似（光这路径都这样累赘，还是使用stage好）。 

- **build-dir**：编译生成的中间文件的路径。这个本人这里没用到，默认就在根目录（E:\SDK\boost）下，目录名为bin.v2，等编译完成后可将这个目录全部删除（没用了），所以不需要去设置。 

- **link**：生成动态/静态链接库。static：静态。shared：动态。一般使用static编译，免得还需要boost的dll。

- **runtime-link**：动态/静态链接C/C++运行时库。同样有shared和static两种方式，这样runtime-link和link一共可以产生4种组合方式，各人可以根据自己的需要选择编译。一般link只选static的话，只需要编译2种组合即可，即link=static runtime-link=shared和link=static runtime-link=static，本人一般就编这两种组合。 

- **threading**：单/多线程编译。一般都写多线程程序，当然要指定multi方式了；如果需要编写单线程程序，那么还需要编译单线程库，可以使用single方式。 

- **debug/release**：编译debug/release版本。一般都是程序的debug版本对应库的debug版本，所以两个都编译。 

- **address-mode：**address-model=64，如果没有这个属性的话，会默认生成32位的平台库，加入这个选项才能生成64位的DLL。如果运行在VS32位的命令行下需要添加” architecture=x86”，笔者使用x64 Native Tools Command Prompt for VS 2017没有x86与x64之间的矛盾，所以未设置。 

  

#### 三 Linux下编译

`>./booststrap.sh`

```
./bjam stage --toolset=gcc --with-date_time --with-thread --with-filesystem --with-program_options --stagedir="/mnt/hgfs/sdk/boost/bin/gcc" link=static runtime-link=static threading=multi debug release
```

​	这些参数的意义与Windows下完全一样，只不过编译器改成gcc，其他选项根据自己的需要进行设置。

生成的库文件就在bin/gcc/lib目录下，与vc9编译的一样。

如果将来不会再升级boost版本，那么可以把中间文件全部删掉，包括bin.v2目录和tools/jam/stage目录。

​	