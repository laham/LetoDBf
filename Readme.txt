

                              __         __        ____  ____  __
                             / /   ___  / /_____  / __ \/ __ )/ _|
                            / /   / _ \/ __/ __ \/ / / / __  | |_
                           / /___/  __/ /_/ /_/ / /_/ / /_/ /|  _|
                          /_____/\___/\__/\____/_____/_____/ |_| ork


# Welcome to LetoDBf

Contents
--------

0. tl;dr
1. Directory structure
2. Building binaries
   2.1 via hbmk2
   2.2 Borland Win32 C compiler
   2.3 MS Visual C compiler
   2.4 Old Harbour 3.0
3. Running and stopping server
   3.1 the classic way for all OS
   3.2 Run as Windows service
4. Server configuration
   4.1 letodb.ini
   4.2 Different Server setups
   4.3 Authentication
5. How to work with the letodb server
   5.1 Connecting to the server from client programs
   5.2 Filters and Relations
   5.3 Database driver
   5.4 Special Data Files in RAM
6. Variables management
7. Functions list
   7.1 Connection management functions
   7.2 Transaction functions
   7.3 Additional functions for current workarea
   7.4 Additional rdd functions
   7.5 Setting client paramenter
   7.6 File functions
   7.7 Management functions
   7.8 User account management functions
   7.9 Server variable management functions
   7.10 Calling udf-functions on the server
   7.11 Functions for bitmap filters
8. Utils
   8.1 Server Management utility
   8.2 Uhura
9. Server-side functions
10 Abbreviations and remarks
A. Internals



      0. tl;dr

 In following chapters with many words is described the extended use of the pair:
 <client> and <server>.
 The <server> is an executable running in a network,
 and <client> communicating with the server is your project linked with this library,
 So you get a R-eplaceable D-atabase D-river ("LETO"), selected as default driver if not explicitely
 given in functions like DbUseArea().
 Databases and index orders are then used by the server, the client requests the server about
 records and timed caches them in client RAM for further access.

 For a quick early test, Harbour users need to read/ make: 
 # 2.1 --> building server executable // client lib:
   hbmk2 letodb[ svc ] // hbmk2 rddletoaddon
 # adapt server data- & log- path in bin/letodb.ini
   3. --> start executable[ or service ]
 # 5.1 --> add Leto_Connect( "//DNSname|IP/" ) in main() before using workareas,
   build your project: hbmk2 yourapp[.prg|.hbp] letodb.hbc


      1. Directory structure

      bin/          -    server executable file
      include/      -    source header files
      lib/          -    rdd library
      source/
          client/   -    client RDD lib sources
          common/   -    some common source files to server and client
          server/   -    server sources
          3rd/      -    third party source
          3rd/lz4   -    LZ4 compression library
      tests/        -    test programs, samples
      utils/
          manager/  -    server management utilities
          backup/   -    demo of backup-ing a running server
          uhura/    -    automatic server IP detection



      2. Building binaries

 Get and build the fantastic Harbour:
    The letodb server and client library can be compiled only by the Harbour compiler >= V3.0.
    It is strong recommended to download and build Harbour from the fresh 3.2 source:
       git clone https://github.com/harbour/core.git
    For this you need your C-Compiler used for Harbour in your OS search path.
    Or to use latest binary package:
       https://sourceforge.net/projects/harbour-project/files/
    Follow the instructions found with Harbour.

 Get latest source of LetoDBf
    with GIT:
       git clone https://github.com/elchs/LetoDBf.git
    or as packed package at:
       https://github.com/elchs/LetoDBf
       ZIP: https://github.com/elchs/LetoDBf/zipball/master
       TAR: https://github.com/elchs/LetoDBf/tarball/master
    and change in command window into the the root directory of download package.


      2.1 building letodb with hbmk2, for all C compilers

 Server itself:
    letodb.hbp is ready configured server for Windows and Linux daemon,
    letodbsvc.hbp is ready configured server for use as Windows service.
    -- Windows service  hbmk2 letodbsvc
    -- all other OS     hbmk2 letodb

 Client library:
    -- all OS:          hbmk2 rddleto
 Recommended is to integrate LetoDbf client library into your Harbour environment as an 'addon':
    -- all OS:          hbmk2 rddletoaddon

    If Linux user have 'installed' Harbour, you need root rights to also install LetoDBf as 'addon':
    -- Linux:           [ sudo ] hbmk2 rddletoaddon


 Resulting server executable will be found in the "bin" directory, library will be in "lib".
 In the "bin" directory is also the "letodb.ini" file to configure the server.

 After successful build as 'addon', you can compile *at any place* your applications with:
    hbmk2 your_application letodb.hbc
 else you have to point to the "letodb.hbc", example out of a sub-directory in LetoDBf:
    hbmk2 your_application ../letodb.hbc

 For first testing purpose it is recommended to let the server executable remain in the "bin"
 directory of your LetoDBf package. The following will just copy the executable and letodb.ini
 to another place, you may know a better place for them and do that manually.
 To install LetoDBf server into your OS system search paths:
 -- Linux with Harbour 'installed':
                        sudo hbmk2 letodbaddon.hbp
 -- all OS:             hbmk2 letodbaddon.hbp
 Then the server executable goes into the place, where the Harbour executable directory is.
 In Windows the letodb.ini goes also into same place, in Linux it goes into: "/etc",
 where you need root rights to change config options.
 ! Installing LetoDBf needs to outcomment and adjust the <LogPath> in letodb.ini !
 Use e.g. temporary directory of your OS, where normal users have write rights, e.g.: "/tmp".


      2.2 Borland Win32 C++ compiler
      2.3 MS Visual C compiler

 If the above described way to compile with ".hbp" files does not work ( wrong setup ?, no hbmk2 ),
 for BCC and old older MsVc exists a make_b32.bat and a make_vc.bat. Look into, adapt OS search
 paths to point to Harbour and your C-compiler executable. Further important is to set:
 "HB_PATH" to point to the base! directory of Harbour, e.g. "C:\harbour"
 You will know what to do, are on your own. I use them only for sporadic compile tests.

 BCC55 and maybe also newer ones have a problem with compiling LZ4 compression library, you will
 get this case slower ZLib compression. This must fit together for client lib and server when you
 want to use network traffic compression. It is configured by this "{!bcc}" at top in the ".hbp" files.


      2.4 Old Harbour 3.0

 Basically it is possible to compile and use LetoDBf with older Harbour version 3.0.
 For this you have to search in above named HBP files for the line with: "#-cflag=-D__HARBOUR30__=1",
 and there to remove the single character '#' at line start, which means the line is outcommented.
 This should be the last solution, as you will miss some fantastic new features of Harbour 3.2 and
 instead get some left and meanwhile fixed bugs.

 As the hbmk2 make tool v 3.0 does not know about the "-env:" option in the HBP files, you have to set
 these as environment variables. So to set Environment variables: __LZ4=yes   and   __PMURHASH=yes
 to get defaults to use LZ4 compression and PMurHash algorithm. This is done in your terminal for
 Windows with: SET ...=...   and in Linux with: export ...=...
 You have to manually '#include rddleto.ch' into your projects, hbmk2 3.0 does that not automatic
 by using letodb.hbc.


      3. Running and stopping server

 Before you do so, adapt the "DataPath" in letodb.ini, the most important setting, see 4.1
 If this path does not exist or is invalid, the server will not start !
 If you 'installed' the server or use it as service, also outcomment and adapt the LogPath,
 where the log files will go. If LogPath is not set, they go into directory of server executable.
 For both directories user needs write rights granted by the OS.


      3.1 the classic way for all OS

 Start it, in default mode __WIN_DAEMON__ or both modes __LINUX_DAEMON__ and __CONSOLE__
      start /B letodb.exe           ( Windows )
      ./letodb                      ( Linux )

 To shutdown the server, run the executable with the 'stop' parameter:
      letodb.exe stop               ( Windows )
      ./letodb stop                 ( Linux )

 To reload the "letoudf.hrb" module, containing UDF server side functions, it must be in same
 directory same as server executable, use the 'reload' parameter:
      letodb.exe reload             ( Windows )
      ./letodb reload               ( Linux )

 Linux: it needs a pause of 1-2 minute before you can restart server after a shutdown.
 To automate that, use bash script: 'leto.sh' in "bin" directory, it will start the server
 when it is again possible. It is about the time that must elapse before TCP/IP can release a
 closed connection and reuse its resources. This is known as TIME_WAIT state.


      3.2  Run as Windows@ service

 For use as "Windows service" server executable must be compiled for this task, see 2.1
 To install LetoDbf as service, the executable must be placed in a directory covered by the OS
 system search paths to be found from any place. Then run letodb with 'install' parameter:
      letodb.exe install

 Verify in letodbf.log that the service was successful installed and started.
 To check the state of a Windows service use the GUI management for services.
 Alternatively at command line can be used to start/ stop the service:
      net start LetoDBf_Service
      net stop LetoDBf_Service

 To deinstall service again, run letodb with 'uninstall' parameter:
      letodb.exe uninstall
 It possible may need to restart your Windows machine. You can check beforehand if the service is
 still listed in the GUI management for services, else it is not needed.



      4. Server configuration

      4.1 letodb.ini

 In Windows environment the letodb.ini config file must be placed in directory of server executable.
 In Linux the server looks for it in directory "/etc", if not found there then in the directory of
 server executable.
 This file is only read once with starting LetoDBf, after changes therein you have to restart the server
 to let it get active.

 Really important options commonly only are: DataPath, LogPath, Share_Tables, No_Save_WA,
 LetoDBf newbies then continue reading with section: 5. How to work with the LetoDBf server,
 and come some days later experienced back to look in 4.x sections what else all is possible.

 Currently following parameters exists ( default values are designated ).

      [MAIN]
      IP =                     -    ! Leave it empty for Windows. !
                                    IP address, where Letodb server listens for connections;
                                    If not set, all net interfaces ( all IP addresses ) available on the
                                    computer, are used.
      Server =                 -    IP address used by tools like management console to find the server.
                                    This can be, but must not be, the same as used for config option 'IP'
                                    and is just for convenience.
      Port = 2812              -    Server port number, default is 2812 [ then 2813 used for second socket ]
                                    There are two! ports used by server, this and the following number.
                                    ! You ever connect to first port number !
      DataPath =               -    PATH to a base directory on a server with your databases,
                                    may include also a drive letter for poor Windows systems
      LogPath =                -    if config option <DEBUG> level is greater zero [ 0 ],     
                                    PATH to a directory (with write access) for all log files.
                                    File letodbf.log for the main server will contain some info from settings
                                    at server starttime, plus info about new connected and disconneted clients
      No_Save_WA = 1           -    server mode of internally handling database tables 
                                    1  each dbUseArea() will cause a real file open operation by the OS,
                                       identical to what client requested, so workareas at the server are same as
                                       at client side. [ WA number, alias, filter conditions, relations ]
                                    0  each table is opened only one time, this workarea 'exchanged' in between client
                                       requests. so only one connection will have access to the table at a time.
                                       No relations active at server, Alias names at server are different from 
                                       the client.
                                    Recommend '1' if you plan to execute functions at server side ( UDF ).
      Share_Tables = 0         -    other software simultanous access tables used by server,
                                    which changes logical or physical locking -- in dependance:
                                    # No_Save_WA = 0    
                                    0  server open all tables in exclusive mode, what leads to
                                       performance increase as e.g. record-/ file- locks are not applied by OS.
                                    1  tables are opened in the same mode [shared/exclusive] as client
                                       applications opened them, what allows LetoDB to work in coexistence with
                                       other applications [ non LetoDB users ] simultanous on the same DBF tables.
                                    # No_Save_WA = 1
                                    1  physical record-/ file- locks set with the OS are viewable for other
                                    0  only logical internally locking
      Default_Driver = CDX     -    default RDD to open DBF tables in at server, if not given explicitely in
                                    your sourcecode. Possible values: CDX NTX
                                    If the server is linked with rushmore index support, CDX gets BMCDX and
                                    NTX will became BMNTX
                                    If not set, the server by default uses CDX [ BMCDX ]
                                    Can on demand changed by client with function: leto_DbDriver().
      Cache_Records            -    The default number of records to be read into the client read cache,
                                    used for skipping etc without new requesting the server.
                                    Records are valid at client as long as the hotbuffer timeout.
                                    Default is 10, minimum is 1 (slow performance), good values are 10 - 50,
                                    theoretical ! maximum 65535. Adapt for performance in your environment.
                                    Can be set for specific tables and occasions with leto_SetSkipBuffer().
      Lock_Scheme = 0          -    If > 0, extended locking scheme will be used by server.
                                    * This is only needed, if your DBF will be greater in size as 1 GB. *
                                    Then DB_DBFLOCK_HB32 will be used for NTX/CDX;
                                    _or_ if set to 6, DB_DBFLOCK_CLIPPER2 for NTX, HB32 for other
                                    _or_ if set to 2, DB_DBFLOCK_COMIX for CDX, HB32 for other.
      Memo_Type =              -    LEAVE IT EMPTY, to get the default for the choosen option Default_Driver.
                                    Default: FPT for DBFCDX, DBT for DBFNTX, SMT for others.
      Memo_BSize =             -    for !expert! users !, this will change default memo blocksize
                                    for *NEW* created DBF data tables. Before doing so, you need a lesson about.
      Lower_Path = 0           -    if 1, convert all paths and filenames to lower case;
                                    This is useful if: all files at disk are in lower case, in your
                                    application they are named mixed case, and the [Linux] OS for server
                                    is case sensitive for filenames.
      EnableFileFunc = 0       -    if 1, using of file functions ( leto_file(), leto_ferase(),leto_frename() etc ..
                                    is allowed. Else these functions do nothing or return .F.
      EnableAnyExt = 0         -    if 1, *creating* of data tables and indexes with any extention, other than
                                    standard ( dbf,cdx,ntx ) is allowed. Else these would be rejected.
      Pass_for_Login = 0       -    Lowest level of password verification: after login all is allowed to all
                                    if 1, user authentication is necessary to login to the server;
      Pass_for_Manage = 0      -    if 1, user authentication is necessary to use management functions,
                                    e.g. run the monitor console [ Leto_mggetinfo() ]
      Pass_for_Data = 0        -    if 1, user authentication is necessary to have write access to the data;
      Server_User =                 The Unix/ Linux username from whom UID and GUI are fetched for __LINUX_DAEMON__.
                                    Have precedence over following two options, if username is given and exists.
      Server_UID = 0                The User-ID and Group-ID for the Linux server to run as daemon.
      Server_GID = 0                Your DBF tables will get this IDs, important for choosing the correct access rights.
                                    Default is empty, then this will be the U-ID and G-ID who started the server.
      Pass_File = "leto_users" -    the path and name of users info file;
      Max_Vars_Number = 1000   -    Maximum number of shared variables
      Max_Var_Size = 67108864  -    Maximim size in sum of all text/ array variables, default 64 MB.
                                    A single text/ array variable is allowed to be a quarter of that ( 16 MB )
                                    Be very carefull with thoughtless increasing this value to much bigger sizes,
                                    as the server will need at least 4 times of that value as RAM.
                                    Theoretical! maximum for a single! item is ~ 4 GB, then your server will need
                                    to have 64! GB!! RAM. [ NOT tested ! :-) ]
      Trigger = cFuncName      -    Server side trigger function for *every* table. ! USE WITH SPECIAL CARE !
                                    If given, this trigger function is executed for *all* opened WAs for specific
                                    actions like record append, update, ...
                                    Function <cFuncName> must be known at server, and can be a HRB loaded UDF
                                    function. It receives 4 params: nEvent, nArea, nFieldPos, xTrigVa;
                                    where nFieldPos and xTrigVal are only filled for events EVENT_PUT and EVENT_GET.
                                    ( see an example of "Leto_Trigger" in: tests/letoudf.prg )
      Tables_Max = 999         -    Number of *MAXIMUM* designated DBF tables handled by server,
                                    for server mode No_Save_WA == 0 this are physical DBF tables,
                                    for server mode No_Save_WA == 1 this are DBF tables opened by all users.
                                    This number can *not* be increased during runtime of server.
                                    Theoretically maximum value: 1000000, minimum: 100.
                                    Increase default value big enough to your needs,
                                    Example for No_Save_WA == 0: 2 * physical existing DBF
                                    Example for No_Save_WA == 1: Users_Max * physical existing DBF
                                    ( Maximum limit per one single user connection is about ~ 60000. )
      Users_Max = 99           -    Number of *MAXIMUM* designated users. do not set too low.
                                    This number can *not* be increased during runtime of server.
                                    Theoretically maximum value: 65534.
                                    Increase default value big enough to your needs, example two times as actually
                                    users, but do not exaggerate.
      Debug = 1                -    Debug level, default: 1 --> only minor information about login/ logout of
                                    connections is written into letodbf.log
                                    0 = none debugging messages -- notice that real errors are always logged.
                                    The greater the the value, the more information is written.
                                    A value >= 15 will include partly communication traffic to server,
                                    with a value > 20 the full communication protocol is logged.
                                    !! USE WITH CARE !!, the log files can get very quick VERY BIG.
                                    It can be changed 'on the fly' for critcal sections with new
                                    RDDI_DEBUGLEVEL -- see 7.5
                                    ONLY increase value in case of problems, to trace what happened at server,
                                    and the actions from client. Each connection will get an own log file with
                                    connection-ID as file extension; new created when a connection starts. 
      HardCommit = 0           -    if 0, SET HARDCOMMIT OFF, this is now DEFAULT.
                                    It is recommended for UNSTABLE running server to set it to <1>,
                                    which means that each change at data tables are immedeate written to
                                    harddrive bypassing the OS cache.
                                    Expect significant reduced performance with setting '1'.
      Optimize = 1             -    _SET_OPTIMIZE setting
      ForceOpt = 0             -    _SET_FORCEOPT setting
      Allow_Udf = 0            -    security setting, DEFAULT is ! NOT ! to allow the use of
                                    loaded UserDefinedFunction for remote execution at server.
                                    With value 0 even a Leto_UDFExist() will deny to answer.
                                    Set to 1 to use UDF functionality on LetoDB server.
                                    0 will disable that possibility.
      TimeOut = -1             -    Connection timeout in seconds, -1 means infinite wait.
                                    This timeout determine, how long a write to network/ wait for requested workarea
                                    will wait to succeed, before the thread for the connection give up.
                                    If used: Zombie_Check, this value shell be shorter than that.
      Zombie_Check = 0         -    Time in seconds, that a client must be quiet ( no activity ), before
                                    a 'are you healthy' query (ping) is send from server, to verify it's not a
                                    dead/ unplugged connection. ! Application must be linked multi-thread ( '-mt' ) !,
                                    else these checks cannot be done.
                                    As 3 times for a given interval a check is done, a zombie can be 1/3 time
                                    longer 'dead', e.g. 60 ==> max. 80 seconds 'dead' before detected.
                                    Such connection will be shut down, opened files and locks are reset-ed.
                                    If set to 0 [ default ], these checks are diabled.


      4.2  Different Server compile setups/ extensions

      4.2.1 UDF support

 Aside calling single Harbour command with leto_UDF( "cCommand"[, xParam] ),
 you can load your own PRG-level functions with a <HRB> file also during the server is running.
 A very basic example is found in: tests/letoudf.prg.
 How to compile a PRG to a HRB, look into letoudf.hbp. This is called with: hbmk2 letoudf.
 Place the resulting <HRB> file in same directory as the server executable.
 After the "reload" command or together with server start you have an entry in letodbf.log if they
 were successful loaded. In case of error you shell also find a short text what have failed.
 See further at Leto_Udf() ... 

 For the execution of single Harbour functions at server side, or when your functions in the HRB file
 need Harbour commands ( like "STR", "DTOC" ), these Harbour functions must during compile process linked
 into the executable.
 There are already very !many! available. If really one is missing, it must be added at top in:
 source/server/server.prg, done like the others there with a: REQUEST <cFunction>

 You can enable also the full set of all basic Harbour commands to be available at server runtime.
 Herefore comment out 2 lines in source/server/server.prg.
 Or if you need the full set of the Harbour Cl*pper tools contrib [CT], another two lines must be
 outcommented.


 * In server mode No_Save_WA=0: for all tables; and for all HbMemIO tables in any server mode: *
 at server side the ALIAS name for a workarea is *different* to that used in your application.
 Client ALIAS is *automatically* translated if part of <cCommand> string e.g. in codeblocks.
 If you want to access at multiple workareas simultanous in your UDF, and such WA is not the active one,
 UDFs functions in HRB are needed. Here then to work with Leto_Select() for above specified WA and
 Leto_Alias() for all other tables to query for the WA ALIAS name at server.
 The ALIAS at server for these tables is not predictable because dynmically created,
 ( FYI: scheme is: "Exxxxxxx", where x is a number of sequential occurance to create such an ALIAS global
   to server for all connections -- it is re-used after a workarea is lastly closed. )


      4.2.2 Codepage support

 This topic relates mainly to index files.
 Each connection can use a different codepage.
 Each connection is actually limited to use same codepage for all its tables.
 You should avoid to open the *same* DBF aka its index files with different codepages.
 These limits i may extend, if possible, in future if its' really needed.
 It is important, with which CP setting the index was created and then later DBF data is modified.

 If your questions now are: what is a codepage ? -- how do i determine the used codepage in my sourcecode ?
 -- what is the command to change in my sourcecode the codepage ? --  and similar questions ...
 indicate, that you are using default settings. Then you are nearly done about this section.

 In the file: source/include/letocdp.ch
 you may adapt the list of available codepages. These can then be enabled/ loaded for a client connection.
 ! After changing content in that include file, you have to rebuild the server !
 The names in that file are the same you use in your sourcecode, but with prefix: HB_CODEPAGE_MyCodepage

 Above shell be the recommended way to adapt the list of possible codepages.
 Alternatively, you can enable *all* by Harbour known codepages by outcommenting in:
 letodb.hbp ( ledodbaddon/ letodbsvc ) the line with: "__HB_EXT_CDP__" [ remove the '#' at beginning ]


      4.2.3 Rushmore bitmap index support

 Before you believe, this will be the mother of filtering performance problems, let be told she isn't.
 Better to invest a bit time about how to get optimized *server-side* filters with help of the Leto_Var*()
 system, see section 5.2 and search for the Leto_VarGetCached() idea ...
 Because to get the array with record numbers to be set as valid, it must be skipped one time through the
 whole database, in opposite to first suggest where this is done for so many records as just needed.
 To update this 'fixed set', again a full run is needed, where above suggest just actualizes the expression.

 The server and the client library can be built with support of the driver BMDBFCDX/ BMDBFNTX/ BMDBFNSX.
 In this case, these RDD will be used by default, aka e,g. BMDBFCDX if "CDX" is found in letodb.ini
 Basically they support the same functionality as classic CDX/ NTX, but there are five functions to set
 bitmap filters, see section 7.11
 To build server for this, you need to uncomment in the hbp file / letodb.hbp, rddleto.hbp ) a macro:
 "__BM", done with removing of the '#' at beginning of lines with that "__BM".
 Then re-compile *both* sides sides of LetoDBf, aka server executable and client library.
 As they are implemented as an UDF call, you must also set in letodb.ini Allow_UDF = 1.


      4.2.4 Compression LZ4 / ZLib

 Default is to use highspeed realtime LZ4 compression algorithm, but stoneage BCC 5.5 compiler cannot use it !
 This can be changed to classic ( slower! ) ZLib compression by outcommenting with set a < # > into first position
 of the corresponding .hbp files, found at very top the line with: "#-env:__LZ4=yes"
 I would very recommend lightning fast LZ4.
 Re-compile both!! server executable and client library. This must fit together, a server with LZ4 won't understand
 a client application with ZLib.

 Additional remark: at least stoneage old BCC 5.5 had problems with using LZ4, so it is outcommented for all BCC
 versions. If you want to try it with a newer BCC version you have to remove that: "{!bcc}" in the HBP files.


      4.3 Authentication

 To turn authentication system on, aka to log into server with required username/ password,
 you need to set one of the following letodb.ini parameters to 1:
 Pass_for_Login, Pass_for_Manage, Pass_for_Data.
 ! Beforehand, you need to create at least one user with admin rights, because when authentication
 system is active, only authenticated users with admin rights are able to add/ change users
 and passwords.

 To add a first user, you need to execute LETO_USERADD() one time, for example:
      LETO_USERADD( "admin", "secret:", "YYY" )
 where "secret" is the password, and "YYY" is a 3 letter Y_es and N_o string,
 which grants rights to; 'admin' 'manage' 'write access'.

 You can also use the console program in utils/manager/console.prg to add/ delete users.
 Look for section 8.1

 To connect to a server with an authentication active with username and password, you must use
 the LETO_CONNECT() function.



      5. How to work with the LetoDBf server

      5.1 Connecting to the server

 Look for example given in tests/basic.prg.

 It will basically need 2 lines modification to your existing source, the rest of your
 application will stay as it was without LetoDBf usage.
 Hopefully you did not set the second param of DbUseArea( , cDriver, ... )
 or used the "VIA" option of the "USE command". In that case remove them all.

 Most easy and highly recommended is to build your app for LetoDBf by using "letodb.hbc" for hbmk2.
 You can include it into your project HBP by adding it in an extra line, or use it at command line:

      hbmk2 yourapp[.prg|.hbp] letodb.hbc

 It automates for you the following steps:
 a) request for the LetoDBf RDD driver, else manually must be set outside the main() procedure:
      REQUEST LETO
 b) includes the header file: "rddleto.ch", else it must be done manually in every of your PRG:
      #include "rddleto.ch"
 c) set the switch "-mt" to link your application with LetoDBf client library with multi-thread support.
    Further an internal, additive idletask is activated, if your application is build with '-mt'.

 Then there are two ways to open a DBF table at the server,
 THE **very recommended** because portable way is using leto_Connect().

      IF leto_Connect( "//192.168.5.22:2812/" ) < 0
         Alert( "Can't connect to server ..." )
         QUIT
      ENDIF

 For detailed parameters info of leto_Connect() see: 7.1

 This will set the default RDD driver to "LETO" after connecting similar done with: RddSetDefault( "LETO" ).
 Also the server is informed about four SET settings: DELETED/ SOFTSEEK/ AUTOPEN/ AUTORDER.
 With connection to the server, and later with opening or creating a table, information about codepage
 and dateformat settings are sent to server. This is important for creating index orders containing
 national special characters or index keys containing a date value.
 The last applied dateformat setting will be from then on the active one at server.

 All filenames and paths are now relative to the root DataPath in letodb.ini.
 It may look alike: ( to this example i refer in the following explanations )
      DataPath = [drive:]\path\to\data_diretory

 If none DataPath is given ( ! NOT ! recommended ), it will be the root directory with the server
 executable.

 Example: DbUseArea( .T.,, "test\customer.dbf" ) will open DBF in:
      [drive:]\path\to\data_diretory\test\customer.dbf.

 A drive letter in your filenames will be cut away, and only one "..\" directory step up higher than
 the <DataPath> in letodb.ini is allowed. This means: DbUseArea( .T.,, "..\data_other\customer.dbf" )
 will point to:
      [drive:]\path\to\data_other\customer.dbf.

 If your filenames contain at least one path seperator '/' or '\', they will be treated as relative to
 <DataPath>.  All path separators in your filenames are converted by LetoDBf server internal to the needed
 one, no need to take care about as "\" or "/" is equal.

 This root path is equal a SET DEFAULT TO setting, so if your filenames contain *no path separator,
 all new files will be created directly in <DataPath>.
 The filenames can have optional OS dependent leading '\' or '/', example: /mydbf.dbf, that will force
 them into the <DatPath> directory, despite a given "SET DEFAULT" to a subdirectory.

 For blank, pure filename the "SET DEFAULT TO ... " setting, when set *before* Leto_Connect() will name
 an additive sub-directory to <DataPath>. Example: "SET DEFAULT TO data" will put all NEW files without
 path separator into:
      [drive:]\path\to\data_diretory\data
 !! Allowed for "DEFAULT" is only one single path, NOT ENDING WITH ';' OR ":" !!
 The example setting of "SET PATH TO system;tmp" will lead to search for files in:
      [drive:]\path\to\data_diretory\system  and [drive:]\path\to\data_diretory\tmp
 If you use: DbUseArea( ,, cFile, .. ) and it is not in the DEFAULT directory, but in one directory given
 by SET PATH, it will be opened without further needed action from you.
 Both DEFAULT and PATH recognize the <DataPath>, what means a "SET DEFAULT TO to\data_diretory\data" will
 lead to a DEFAULT directory: '[drive:]\path\to\data_diretory\data' as given as "SET DEFAULT TO data".
 !! Again, both settings of "DEFAULT" and "PATH" apply to filenames without any path seperator '\' or '/'.

 To check for some first examples, look into the "tests" directory. Build them all at once with: "buildall"
 resp. "-/buildall.sh" for Linux. Add for execution as first param the IP address of the server,
 if unknown but at same machine running as the application use loopback network: 127.0.0.1
 At any case you shell try the "test_file" executable, as about a not working "Leto_File() functions exist
 multiple! miles long mysthic threads at Harbour Google group about its inner magic :-) 8-) !!

 The other, not recommended way because of not being portable, is to open a DBF table with the
 'on the fly' method by adding IP-address[:port] plus relative path to Harbours 'USE' command,
 example:
       USE "//192.168.5.22:2812/mydir/test"
 will open file in:
       [drive:]\path\to\data_diretory\mydir\test.dbf.
 After doing this an initial time, you are also connected to the server and would need for the next calls
 no more IPaddress:port prefix more. This is a hint for experienced LetoDB users to think about.


      5.2 Filters and Relations

      5.2,1 Filters

 The filter is established usually: by the SET FILTER TO command or by calling DbSetFilter() function.
 Most important param of DbSetFilter() is the expression, the second param: DbSetFilter( bBlock, cExpression )
 as only this can be transmitted to the server. ( codeblocks are not exchangeable between applications )
 The filter expression which can be executed at the server is called "optimized".
 If the filter must be executed locally at client, it is called: "not optimized". Such filter is slow as all
 records must be received from the server, and the client have to discard all the invalid records.
 In case of an optimized filter, only valid records are send to the client application.

 To get an optimized filter, it is necessary, that the expression is solely executable for the server.
 So all the functions therein, like standard Str(), Upper(), DToS() etc must be known to the server.
 If your expression contains an own function from yourself, this can be loaded with a HRB file any time during
 a running server, see therefore 4.2.1 UDF supprt.
 If your expression contains a variable only known to you application, the mighty Leto_Var*() system comes into
 action. With this you can share the content of a variable at client side with the server, and vice versa.
 See therefore section 7.9 Server variables.

 So with the help of leto_Var*() functions plus UDF loadable functions plus a rich set of classic Harbour
 commands, it is possible to turn any non-optimized filter into an optimized.
 These are lightning fast and a pleasure to work with. To test, whether a filter is an optimized, just check
 for with the LETO_ISFLTOPTIM() function, if it returns TRUE.

 Following two settings influence the use of optimized, server side filters:

 SET( _SET_OPTIMIZE, .F. ) will disable any filter evalution at server, so every filter will become a
 non-optimized filter to be executed by client [ the default is .T. == allow them ].
 Only in server mode: No_save_WA = 1 and then setting:

 SET( _SET_FORCEOPT, .T. ) will enable filter expressions at server with ALIAS names, maybe of an relationed
 workarea. [ default is .F. == ALIAS not allowed ].
 Server mode No_Save_WA = 1 is needed with ALIAS names other than the active WA, because in mode '0' LetoDBf
 server uses WA detaching/ requesting technics and other WA are not available/ detached, and so there is no
 relation active at server.
 With ForceOpt = TRUE a filter expression MUST be valid at server, else a RTE is thrown -- it will NOT be
 evaluated alernatively at client as without ForceOptimize.
 [NEW] If the filter expression contains a PRIVATE/ PUBLIC memvar, they will be synced between client and server
 with help of the LetoVar() system. A unique Leto_VarCreate() with value of the memvar is executed, the filter
 string expression is automatically modified to use a Leto_VarGet() instead of the memvar.
 With each move ( GoTo[p|bottom], Skip, Seek ) in the WA, a possible changed memvar value is automatic synced
 with a new Leto_VarSet() call. Clearing the filter, also done with closing the table, will Leto_VarDel() all
 created LetoVars.
 Example: 'SET FILTER TO table->field > xMemvar' will work as without LeotDBf, but as optimized filter.
 See also Leto_VarExpr*() functions if you want to do it manually, or for other occasions.


      5.2,2 Relations

 In server mode: No_Save_WA = 1 relations are additional active at server side. This is not possible for server
 mode: No_Save_WA = 0, here the relations are only active at client side.
 To transmit the relation expression to the server, the second param of DbSetRelation( bBlock, cExpression ) is
 needed, as only a string can be transfered to the server, no codeblock.
 By using the command: "SET RELATION TO ... INTO ..." this second param is filled automatically through the
 header file: <std.ch>, which is anytime included for any Harbour application. This second param have to be set
 manually if the function DbSetFilter() is used. Be careful that bBlock and cExpression mean the same.
 If cExpression contains an error, or is not executable at server side because of a function known only in
 your application ( see 4.2.1 UDF support ) or client application variables ( see 7.9 Server variables ), you will
 get a RTE with a description what failed at server.
 Setting a 'cyclic relation', aka a set of relations where one relationed child area refer back to an parent area,
 lead to an RTE informing you about. The most simple 'cyclic relation' is a relation pointing to itself workarea.


      5.3 Database driver

 If nowhere explicitely set, the default database driver for the server is DBFCDX.
 This default driver at server can be changed in the letodb.ini with setting of Default_Driver.
 The active driver for your connection you can query and !SET! with:

       leto_DbDriver( [ <cNewDriver> ], [ <cMemoType> ], [ <nBlocksize> ] )
                                                               ==> aInfo

 This will return a 3 dimensional array, in the order of the parameters, so aInfo[ 1 ] is
 the active used driver. With no arguments the active settings are returned.
 You can change that default by using "DBFNTX", "DBFCDX", "DBFFPT", "DBFNSX" or "SIXDBF".

 Each driver can be aside their defaults combined with MemoType "DBT", "FPT" or "SMT".
 Further for expert ! users the blocksize used for the memofield can be changed:
 minimum is 32 Bytes, maximum 64KB == 65535 Bytes and always must be a multiple of 32 Bytes.
 ! This must be done before creating a new DBF, else the setting active with creation is used for
 that DBF.

 !! Use leto_DbDriver() function before open or create new files !!
 This way you can even mix different drivers for a single connection. Sure you can not mix different
 drivers for the same database, so e.g. a DBFNTX table must be used for all connections as NTX type.


      5.4 Special Data Files in RAM

 LetoDB can create ( and share between connections ) data tables in RAM, not on harddrive.
 These HbMemIO called tables are especially useful for temporary tables. They are limited to
 available RAM at your server and only valid during one server run, all lost after LetoDB shutdown.

 Filenames for these special data tables optional can have *optional* a leading path seperator
 [ '/' or '\' ], follwed mandatory by a 'mem:' prefix. As example:
 "/mem:speeddata.dbf" will be a valid filename. These data tables work like 'real' data tables,
 so they are created using the active database driver ( see 5.3 ).
 When index orders are created, the bagname must also have this prefix: "mem:", else it will be
 created on harddrive.
 You also can combine a data table on harddisk with an ( temporary ) index in RAM.
 Or a data table in RAM with index on hard drive, but this combination makes not much sense ;-)
 These HbMemIO files have especially at slower harddrive some performance advantages.

 !! Don't forget to drop/ delete no more used hbMemIO files, else the server may quickly run out
 of available RAM. !!



      6. Variables management

 Letodb allows to manage variables, which are shared between applications, connected users
 to the server and with the server itself. All operations on variables are performed consecutively by
 one thread, so the variables may work as semaphores.

 Scroll to section: 7.9 Server variable functions, and check a first example in tests/test_var.prg.



      7. Functions list

      7.1 Connection management functions

 Below is a full ( at least, for the moment I write it ) list of functions,
 available for using in client applications with RDD LETO linked.

 In these functions <cAddress> means the IP-address in format: "//IP:port/".
 The ":port" part got now optional, if not given it will use ":2812" as default.

      LETO_CONNECT( cAddress, [ cUserName ], [ cPassword ],
                    [ nTimeOut ], [ nBufRefreshTime ], [ lZombieCheck ] )
                                                               ==> nConnection, -1 if failed
 <cAddress> can be given as raw IP-address "127.0.0.1" or "//127.0.0.1/".
 If the port is omitted ( aka: "//127.0.0.1:2812/" ), the default port number: 2812 is choosen.
 Alternatively DNS names can be used instead of an IP number.
 <nTimeOut> defines, how log for an answer from server application will wait, in 0.001 seconds.
 This timeout value is valid for each request to the server, not only for the initial connect.
 Default is 120000 aka 2 minutes. '-1' means infinite wait. After that timespan, application will
 break with an error if no answer from server had been send.
 <nBufRefreshTime> defines the time interval in 0.01 second units. After this time is up,
 the records skipbuffer will be refreshed, 100 by default ( 100/100 == 1 sec ).
 Value zero (0) means infinite! caching, -1 will disable using the skip buffer. These both extreme
 values should be applied only at very special! occasion and need.
 lZombieCheck = .F. disable check for dead connection and also the second socket
 for faster communication with the server. Default is .T.
 If you use in letodb.ini configuration point: Pass_for_Data = 1, it is advised to
 disable lZombieCheck, aka to set it explicitely to .F.


      LETO_CONNECT_ERR( [ lAsText ] )                          ==> nError [ cError ]

 Retrieves the last occured error for active connection, not only after connect.
 With <lAsText> TRUE given a string with an description.

      LETO_DISCONNECT( [ cAddress ] )                          ==> lDisconnect
 Dis-connnect current connection, returns boolean if a connection is disconnected.
 With optional param <cAddress> that connection is tried to disconnect.

      LETO_SETCURRENTCONNECTION( cAddress )                    ==> cAddress
 Returns the <cAddress> of the active connection after a try to change the active one.
 It is an empty string "" if IP was wrong/ not given.

      LETO_GETCURRENTCONNECTION()                              ==> cAddress
 Return the <cAddress> of the active connection, EMPTY string "" in case of no active
 connection. This function can be used together with Leto_SetCurrentConnection() to
 save/ restore the active connection.

      LETO_GETSERVERVERSION( [ lHarbourVersion ] )             ==> cVersion
 Returns version of LetoDBf server, with given .T. boolean parameter the version of Harbour at
 compile time.

      LETO_GETLOCALIP( [ lLocal ] )                            ==> IP address of client [ server ]
 Returns IP address of first found interface for subnet, which may be the wrong one,
 if you have multiple ( pgysical or logical like bridges ) NICs for that subnet.
 With optional <lLocal> == FALSE ( .F. ) returns IP address of server.

      LETO_ADDCDPTRANSLATE( cClientCdp, cServerCdp )           ==> nil
 For ugly ;-) xHarbour hackers with different CP names, no comment.

      LETO_SET( nOption [, xNewSet ] )                         ==> xActiveSetting
 Recommended is to use the commands: SET xxx [ TO ] or the SET() function instead, to keep your
 sourcecode portable. A translation is done in "rddleto.ch" by define macros, to inform server about
 changes of four settings: SOFTSEEK, DELETED, AUTORDER, AUTOPEN. All other requests/ settings are
 simply forwarded to the SET() function.
 With no given <xNewSet>, you get the active setting without changing it.


      LETO_RECONNECT( [ cAddress ], [ cUserName ], [ cPassword ],
                    [ nTimeOut ], [ nBufRefreshTime ], [ lZombieCheck ], [ nDelay ] )
                                                               ==> nConnection, -1 if failed
 All param are optional! and same as LETO_CONNECT(), plus optional 7th param <nDelay> in unit seconds.
 This will close a possible still alive or dead connection to server, and re-establish a new connection
 to the same or even different server if <cAddress> is given. Except <cUserName> and <cPassword> all
 params are filled in by setting of the old connection.
 The full WA environment ( tables, index orders, filters, scope, relations, R|F-locks ) is restored
 for the new connection.
 If it is the same server, <nDelay> will be 1.0 second to let the server close the tables and remove
 existing locks before try to establish a new connection, for different server no delay is needed.
 <nDelay> should only needed to be manually set, if a connection is made to same server, but over
 different network.


      7.2 Transaction functions

 A transaction is a series of data changes, which is guaranteed to be applied in a sequence alike
 'one block'. In practical life, only record/ file locks are set at server ( no DbAppend() lock ),
 and all data changes are buffered at client side. If you DbSeek()/ DbSkip() to a record with changed
 data, you will see at client your changes still not applied at server side.
 When committing the transaction, one single request about all changes is send to server, which will
 start processing data changes firstly after complete receiving the request and further pre-checks.
 Up to this point is ensured *all or nothing* of a transaction is processed.
 If the server experience hardware problems during processing the sequence, like a power loss, parts
 of a transaction will miss ...

 As a side effect, transactions are nice for Flock()ed or non-shared, exclusive opened tables, because
 for these it leads to a !drastical! improved performance to append 10000 records in a fraction of a second.
 For RLock() an insane tremendous overhead is needed, if really **many** records need to be
 changed. Example: to Rlock() the 1000th record at server, it must search through 999 existing locks.
 In sum a compare have been done 499500 times for the 1000th record. When the transaction is applied at
 server, these Rlocks are again verified, then are 1000 * 1000 = 1 million checks needed.
 Only 1000 Rlock() record changes of above example are taken just on the fly, but if there are 10th
 of thousands ...

 !! Important !! -- during an active transaction, aka after leto_BeginTransaction():
   # it is explicitly forbidden to mix RLock() and FLock() FOR THE SAME WORKAREA
   # you can not unlock any record or whole table ( e.g. NOT use DbUnlock() )
   # transactions must start and end with a LETO RDD workarea active, and this must be further
     from the same LetoDBf server.
   # you can modify same field of same record multiple times, and in between changing to different
     record, but will get as value always first modification. In other words: you cannot add up a sum.
     Later at server these changes are applied as consecutive changes in a row, so last change makes
     the result.
   # Recno() will be '0' after 'appending' a blank record with e.g. DbAppend()
   # as server does the job for auto-incrementing fields, they will be empty at client side for a
     transactioned DbAppend(). Leave them unchanged, server will fill them later during real append.


      LETO_BEGINTRANSACTION( [ [ lUnlockAll ] )

 NEW: with <lUnlockAll> param, only here default is false ( .F. )
 By default, all locks ( R-locks and F-locks ) remain,
 with .T. it's very convenient to remove them for all LETO workareas at once, which will give
 you a fresh start ( aka must not think about to continue for this or that workarea with Rlock()
 or FLock().

      LETO_ROLLBACK( [ lUnlockAll ] )                          ==> nil

 This will discard the collected field changes and records to be appended.
 By default <lUnlockAll> is true (.T.), then all locks ( R-locks and F-locks ) are removed,


      LETO_COMMITTRANSACTION( [ lUnlockAll ] )                 ==> lSuccess

 default <lUnlockAll> == .T. will automatic unlock *all* locks ( Rlock() and Flocks() ) after
 the transaction is processed at server.
 Default (.T.) recommended to do, as it is faster. Also the client will have no information
 about what records are appended & R-locked at server, it must query for that again.

 ! ATTENTION !
 If you want to also *append* records in a transaction, you need for a shared opened table at
 least Flock() *or* one single Rlock() done during transaction for this workarea.
 If none is locked, as example a transaction containing only records to append,
 LetoDBf client will try internally to set itself a Flock().
 This will fail if any other connection have a record locked [Rlock()] for this workarea.
 Then the transaction will fail with a runtime error.
 Above is needed to ensure, when transaction sequence runs at server, nobody can jump in by
 occasion to Flock() the table, which then would block the wanted DbAppend() at server.
 Rely on this automatic, or ensure explicitely a Flock() or at least one Rlock() active for
 each workarea with records to append. This Rlock()ed record must even not contain changed data,
 only the lock counts.

      LETO_INTRANSACTION()                                     ==> lTransactionActive

 Just returns a boolean if inside or outside heaven.


      7.3 Additional functions for current workarea

      LETO_COMMIT()

 ! Deprecated !
 But functionality is still there, it is what a common DbCommit() does. Both commands are allowed to be
 used during transactions.

      LETO_DBEVAL( [ <cBlock> ], [ <cFor> ], [ <cWhile> ], [ nNext ], [ nRecord ], [ lRest ] )
                                                               ==>aResults
 ! The optional codeblocks: cBlock, cFor, cWhile must be given in literal form !
 This works alike Harbours' DbEval() command, but like an UDF directly at the server: this way only the
 resulting array is transferred over network, not all the records [ to be ] processed.
 Codeblock expressions may, but not mandatory, start/ end with '{' and '}' chars -- if these are missing,
 in front a '{||' and at end a '}' will be added.
 Codeblock receives as first argument a boolean, set to true if *first* valid record is processed, e.g.:
 "{ | lFirst | IIF( lFirst, DoThis(), DoThat() ), DoEver() }"
 ToDo : You can make e.g. sums in the codeblock by using THIS thread global var: xCB [ x means any content and CB codeblock ],
 "{ | lFirst | IIF( lFirst, xCB := 0, ), xCB += 1 }"

      LETO_SUM( <cFieldNames>|<cExpr>, [ cFilter ], [xScopeTop], [xScopeBottom] )
                                                               ==> nSumma if one field or expression passed, or
                                                               {nSumma1, nSumma2, ...} for several fields
 The first parameter of leto_sum is a comma separated list of fields or expressions,
 optional cFilter is a filter condition to be taken instead a possible active filter [ DbSetFilter() ],
 optional xScope[Top|Bottom] are scope values for an active index order,
 Example:
    leto_sum("NumField1,numField2,#", "CharField $ 'elch'", cScopeTop, cScopeBottom )
 returns an array with values of sum fields NumField1 and NumField2.

 If "#" symbol passed as field name, leto_sum returns a count of
 evaluated records, f.e:
 leto_sum("Sum1,Sum2,Sum1+Sum2,#", cFilter, cScopeTop, cScopeBottom)
                                                               ==> {nSum1, nSum2, nSum3, nCount}
 If only one field name or expression is passed, leto_sum() returns a numeric value


      LETO_GROUPBY( cGroup, <cFields>|<cExpr>, [cFilter], [xScopeTop], [xScopeBottom]) ==> aValues
                                                               {{xGroup1, nSumma1, nSumma2, ...}, ...}

 This function return two-dimensional array. The first element of each row is a value of <cGroup> field,
 elements from 2 - sum of comma separated fields or expressions, represented in <cFields>.
 If "#" symbol passed as field name in cFields, leto_groupby return a count of evaluated records in each group

      LETO_ISFLTOPTIM()                                        ==> lFilterOptimized

 To determine if an active filter in selected workarea is optimized [ aka executed only at server side ]
 or non-opimized [ server send all records to client, which then must decide itself for valid records. ]
 See 5.2 for more info.

      LETO_MEMOISEMPTY( cnField [, cnAlias ] )                 ==> lEmpty ( TRUE for not a memofield )

 This is an optimzed function to very fast test, if a memofield of the current record is empty or not.
 The check will be done only at client side, so no network traffic to the server will occure.
 As it else would happen with a test like: EMPTY( FIELD->memofield ), for which the server will send the
 whole content of a memofield to the client, before client can decide if empty or not.
 <cnField> cFIELDNAME or nFIELDPOS, <cnAlias> cALIAS or nSELECT or active WA if empty.

      DbInfo( DBI_BUFREFRESHTIME[, nNewSetting  ] )            ==> nOldVal

 This returns the timeout value for the skipbuffer valid for the this table, before an optional
 new setting is applied with <nNewSetting>.
 Default is no specific timeout for a table, aka to use the general connection timeout value.
 With optional <nNewSetting> it can be applied a new setting only guilty for this specific table.
 "-1 " == skipbuffer disabled, "0" == infinite skipbuffer, nHotBuffer > 0 == nHotBuffer / 100 seconds.
 Above is also the possible range for <nNewSetting> -- plus a value < -1 will disable again a specific
 setting for this table. For the global timeout value see: Leto_Connect() function, 5th param.

      DbInfo( DBI_AUTOREFRESH[, lNewSetting  ] )               ==> lSet

 This returns and sets the behaviour, when the hotbuffer timeout value is elapsed. Default timeout
 is one second and can be changed with Leto_Connect(). If the table have a specific timeout set, this
 have precedence over the global value. Then with next access to a field after elapsed timeout,
 an internal DbSkip( 0 ) is executed to refresh the data from server, if data at server is accessible
 to others, aka the record or the table not locked, table shared opened and not readonly.
 ! Please note, that disabled timeout ( -1 ) will lead to a DbSkip( 0 ) for each field access.
 So if you want to access multiple fields, better set minimum value of 1 == 0.01 second as table
 specific timeout -- else much network traffic will occure.

      DbInfo( DBI_CLEARBUFFER )

  This command clears the skip buffer, to forces to get fresh data with next Dbskip( [ 0 ] ), DbGoTo().


      7.4 Additional rdd functions

      leto_DbDriver( [ "cNewDriver" ], [ "cMemoType" ], [ nBlocksize ] )
                                                               ==> aInfo
 return an array with and in order of the three params, new values are optional.
 New value for <cNewDriver> is then valid for the next opened/ created data table.
 New values for <cMemoType> and <nBlockSize> only get into action for new created DBF tables.
 Only changing <cMemoType> without given <cNewDriver> will also change <cNewDriver>.
 cNewDriver: "DBFNTX", "DBFCDX", "DBFNSX", "SIXCDX", "DBFFPT" ( no index! )
 cMemoType : "DBT", "FPT", "SMT"
 nBlockSize: default for DBT is 512, for FPT 64 and SMT 32 Bytes.
             minimum is 32 Bytes, maximum 65535 == 64 KB; new values as multiple of 32 Bytes.
 Only if server and client library are linked with rushmore bitmap index support ( 4.2.3 ),
 additional possible for < cNewDriver >: "BMDBFCDX", "BMDBFNTX", "BMDBFNSX"

      leto_CloseAll( [ cConnString ] )                         ==> nil

 Somehow obsolete: closing all workareas for a specified or the default connection.
 This function is only interesting for users with multiple server connections and does the same as:
 DbCloseAll() with setting and restoring current connection with Leto_[Set|Get]CurrentConnetion()


      7.5 Setting client paramenter

      LETO_SETSKIPBUFFER( nSkip )                              ==> nSet (buffer statistic using)

 This buffer is intended for optimization of multiple calls of skip.
 This function set size of cached records in a skip buffer for current workarea.
 By default, the size of skip buffer is CACHE_RECORDS server config value. Skip buffer is bidirectional.
 Skip buffer is refreshed after BUFF_REFRESH_TIME ( default: 1 sec )
 Minimum value is 1.
 If parameter <nSkip> is absent, function returns buffer statistic ( number of buffer hits )
 with given numeric value effective set size or 0 if no workarea was selected.
 Related to the skipbuffer timeout see also 7.3: DbInfo( DBI_BUFREFRESHTIME ).

      RddInfo( RDDI_REFRESHCOUNT[, <lSet> ] )                  ==> lOldSet

 By default, the RDDI_REFRESHCOUNT flag is set to true.
 If this flag is set, function: RecCount() retrieve amount of records from server.
 If not set, with record data transmitted values are used and are maybe slightly out-timed.
 If other applications are appending records to the table, new records count won't be immediately seen.

      RddInfo( RDDI_BUFKEYNO[, <lSet> ] )                      ==> lOldSet

 Only use this if you need to very often query for OrdKeyNo(), e.g. during browsing to actualize the scrolbar.
 Default value is .F., which means that the function: OrdKeyNo() will send an extra request to the server.
 If <lSet> is set to TRUE (.T.), the values for OrdKeyNo() are transmitted with the record data and no extra
 request is send. Generally this is very performance decreasing as counting need time at server, so immediate
 deactivate (.F.) it if no more needed.

      RddInfo( RDDI_DEBUGLEVEL [, nNewLevel ] )                ==> nOldLevel

 Reports [ and changes ] the debug level at server, responsible for amount of feedback in the log files.
 Use with care, log files will grow at a busy server in only some seconds MB stepwise ...
 For possible values look 4.1 :letodb.ini ...
 With <nNewLevel> this can be changed on the fly, no server restart is needed. This then applies to all
 active and new server connections.

      LETO_SETSEEKBUFFER( nRecsInBuf )                         ==> 0
 ! DEPRECATED !

      LETO_SETFASTAPPEND( lFastAppend )                        ==> .F.
 ! DEPRECATED !
 because of ugly design problems. It is left as dummy function doing nothing.
 If such former functionality is really needed, encapsulate your request in a transaction.


      7.6 File functions

 IMPORTANT: all file commands start at server with the DataPath, so <cFileName> is a relative path
 to the root path defined in letodb.ini with DataPath.  Only ONE ( 1 ) single ".." is allowed.
 Exception: filenames starting with "mem:" redirects into the RAM of server (virtual HbNetIO FS )

 Recommended is to use plain filenames, after the use of Leto_Connect().

 Alternatively in the OLD style, the <cFileName> parameter of all file functions *can* contain a
 connection string to the letodb server in a format:
 //IP_address:port/[mem:][\]file_name
 where IP_address can be also a DNS name like example "localhost".

 If the whole connection prefix "//..:../" is omitted, the currently active connection is used.
 Such an connection is established with recommended: Leto_Connect() -or- after a aingle LetoDbf command
 by using correct connection prefix. After connection is established, can leave away all this IP:port ...

      Leto_FError( [ lAskServer ] )                            ==> nError
 Returns an error code set by ( some, not for all ) file functions at client.
 NEW: with optional <lAskServer> set to TRUE ( .T. ) a query is send to the server.

      Leto_File( cFileName )                                   ==> lFileExists
 Determine if file exist at the server, analog of File() function.
 <cFileName> is ever relative to the <DataPath> of letodb.ini.

      Leto_FCopy( cFilename, cFileNewName )                    ==> -1 if failed
 Copy a file at the server with a new name at server

      Leto_FErase( cFileName )                                 ==> -1 if failed
 Delete a file at the server.

      Leto_FRename( cFileName, cFileNewName )                  ==> -1 if failed
 Rename a file: <cFileName> ==> <cFileNewName>. <cFileNewName> should be without
 connection string.

      Leto_MemoRead( cFileName )                               ==> cString
 Returns the contents of file at the server as character string, analog of
 MemoRead() function.
 ! FIXED ! If last character in file is character 26 == strg-z, it is removed.

      Leto_MemoWrite( cFileName, cBuf )                        ==> lSuccess
 Writes a character string in <cBuf> into a file at the server, analog of
 MemoWrit() function.
 ! FIXED ! Notice, that character '26' == 'strg-z' is appened to <cBuf>.

      Leto_Directory( [ cDir ] [, cAttr] )                     ==> aDirectory
 Returns a content of directory at the server in the same format as Directory() function.
 With no given <cDir> the DataPath root directory is used.

      Leto_DirExist( cPath )                                   ==> lDirExists
 Determine if cirectory exist at the server, analog of Leto_File() function, but
 for directories

      Leto_DirMake( cPath )                                    ==> -1 if failed
 Creates a directory at the server. [ renamed, formerly: Leto_MakeDir ]

      Leto_DirRemove( cPath )                                  ==> -1 if failed
 Deletes a directory at the server

      Leto_FileSize( cFileName )                               ==> -1 if failed
 Returns a length of file at the server

      Leto_FileAttr( cFileName [, cNewAttr] )                  ==> cAttr
 Get ( without given cNewAttr ) or set <cNewAttr> file attributes, where returned value
 <cAttr> are the active attributes ( after an optional change with <cNewAttr> )
 <cNewAttr> can contain at first place a "-", to revert the following attribute(s),
 e.g.; "-A" will remove the 'archive' attribute; "-" will remove all attributes.
 File attributes are only valid for FileSystem which support them !

      Leto_FileRead( cFileName, [ nStart ], [ nLen ], @cBuf )  ==> -1 if failed
 Read a content of file at the server from <nStart> offset and max <nLen> length.
 Defaults for <nStart> and <nLen> are 0, and nLen == 0 means the whole file.

      Leto_FileWrite( cFileName, [ nStart ], cBuf )            ==> lSuccess
 Write <cBuf> character string to a file at the server from <nStart> offset.
 Default for <nStart> is 0, means at beginning of file.

      Leto_FCopyToSrv( cLocalFileName, sServerFileName[, nStepSize ] )
                                                               ==> lSuccess
      Leto_FCopyFromSrv( cLocalFileName, sServerFileName[, nStepSize ] )
                                                               ==> lSuccess
 Copy a file from/ to client to/ from server, where:
 <cLocalFileName> is filename at client side,
 <sServerFileName> is filename at server which can contain connection info "//IP:port/".
 Optional <nStepSize> determine the size of bytes to be copied with one step, default if
 not given is 1 MB.
 <sServerFileName> can only contain prefix: "mem:" for files in RAM,
 <cLocalFileName> can contain any redirector prefix known by Harbour.
 A simple backup:
    aArr := Leto_Directory( "*" )
    AEval( aArr, { |aItem| Leto_FCopyFromSrv( aItem[1], aItem[1] } )
 Copy from a logged into HbNetIO server a file to LeoDBf located in RAM:
    Leto_FCopyToSrv( "net:hbnetio.txt", "mem:RAMfile.txt" )


      Leto_FOpen( cFile [, nMode ] )                           ==> nHandle
      Leto_FCreate( cFile [, nMode ] )                         ==> nHandle
      Leto_FSeek( nHandle, nBytes [, nOffset ] )               ==> nPos
      Leto_FRead( nHandle, @cBuffer, nLen )                    ==> nRead
      Leto_FWrite( nHandle, cBuffer [, nLen ] )                ==> nWritten
      Leto_FClose( nHandle )                                   ==> lSuccess
      Leto_FEof( nHandle )                                     ==> lEndOfFile

      Leto_FReadStr( nHandle, nLen )                           ==> cBytes
 Stops reading at CHR( 0 )

      Leto_FReadLen( nHandle, nLen )                           ==> cBytes
 Binary version of Leto_FReadStr() including any! char to read

 Above functions do the same and with same params as the Harbour pendants without 'Leto_'
 prefix, aka Leto_FOpen() == FOpen(), but they act on files at server. File names <cFile>
 respect the server datapath, are relative to it.
 It is ensured, that all opened/ created files are closed with connection end,
 and Leto_FClose() will close only files opened/ created with Leto_FOpen/ Leto_FCreate.



      7.7 Management functions

      LETO_MGGETINFO()                                         ==> aInfo[17]
 This function returns parameters of current connection as 17-element array
 of char type values:
 aInfo[ 1]  - count of active users
 aInfo[ 2]  - max count of users
 aInfo[ 3]  - opened tables
 aInfo[ 4]  - max opened tables
 aInfo[ 5]  - 0 [ server up time moved into LETO_MGGETTIME()
 aInfo[ 6]  - count of operations
 aInfo[ 7]  - bytes sent
 aInfo[ 8]  - bytes read
 aInfo[ 9]  - opened indexes
 aInfo[10]  - max opened indexes
 aInfo[11]  - data path
 aInfo[12]  - server CPU(s) time in sum for all connections
 aInfo[13]  - ulWait
 aInfo[14]  - count of transactions
 aInfo[15]  - count successfully of transactions
 aInfo[16]  - 0 [ current memory used ] -- moved into LETO_MGSYSINFO()
 aInfo[17]  - 0 [ max memory used ] -- moved into LETO_MGSYSINFO()

      LETO_MGGETUSERS( [nTable] )                              ==> aInfo[x,5]
 Function returns two-dimensional array, each row is info about user:
 aInfo[i,1] - user number
 aInfo[i,2] - ip address
 aInfo[i,3] - net name of client
 aInfo[i,4] - program name
 aInfo[i,5] - timeout

      LETO_MGGETTABLES( [nUser] )                              ==> aInfo[x,3]
 aInfo[i,1] - filename of data table ( without root data path of server ).
 aInfo[i,2] - 0 if asked for all connections ( nUser < 0 ), else client workarea number.
 aInfo[i,3] - empty "" if asked for all users, else ALIAS name of client workarea .

      LETO_MGGETINDEX( [nUser], [nTable]                       ==> aInfo[x]
 array with filenames of opened index files, global for all connections ( nUser <  0 ),
 or only for a given user and then also only for a specific data table.
 Data table numbers must be retrieved before with above function.

      LETO_MGSYSINFO()                                         ==> aInfo[8]
 aInfo[1]   - free diskspace
 aInfo[2]   - data path of server
 aInfo[3]   - number of CPU cores of server
 aInfo[4]   - free available RAM of server
 aInfo[5]   - server mode ( 1 == No_Save_WA=1, 2 == Share_Tables=1, 3 == Share_Tables=0 )
 next following four items need special build of Harbour with memory statistics ( else all: 0 ),
 and are for debugging purpose only -- for constants see: hbmemory.ch
 aInfo[6]   - HB_MEM_USEDMAX     this and following three need special build Harbour
 aInfo[7]   - HB_MEM_STACKITEMS  with memory statistics-- for constants see: hbmemory.ch
 aInfo[8]   - HB_MEM_STACK
 aInfo[9]   - HB_MEM_STACK_TOP

      LETO_MGGETTIME()                                         ==> aInfo[3]
 Function returns array {<dDate>, <nSeconds>, <nServerUp>}:
 aInfo[1]   - server dDate;
 aInfo[2]   - server time in nSeconds after midnight.
 aInfo[3]   - server UP time in nSeconds
 Convert first tho values to a datetime variable (Harbour):
 hb_DTOT( aDateTime[1], aDateTime[2] )

      LETO_MGKILL( nConnection | IPAddress )                   ==> nConnectionClosed
 Kill user number given by <nConnection> or cIPAddress,
 returns number of killed connection or -1 in case of not found

      LETO_LOCKCONN( lOnOff )                                  ==> lSuccess
 After leto_lockconn( .t. ) request new connections are blocked by server, until
 leto_lockconn( .f. ) called

      LETO_LOCKLOCK( [ lOnOff ] [, nSecs ] )                   ==> lSuccess
 This function wait until any updates/ locks are closed, then commit all changes.
 Afterwards to lock server from any updates from clients, by e.g. to lock a record.
 It returns True, if server is succesfull locked, no user with open locks active.
 Default timeout are 30 seconds.
 Without <lOnOff> you get answer if server is locked() for you [ false for the one who locked server ].

      LETO_TOGGLEZIP( [ nCompessLevel [, cPassword ] ] )       ==> nValueBefore
 This function on demand [de-]activates network traffic compression between server and client,
 where nCompessLevel is:
 -1: == compression off, regularly optimized data traffic,
 [ 0 = none compression, used only by developer for debugging purpose ]
 1 : LZ4 [ or zlib ] compression level.
 If you have compiled LetoDBf server and client with LZ4 highspeed realtime compression [default],
 then 1 is recommended value for compression.
 If LZ4 is disabled during LeoDBf build, then zlib compression is used by Harbour. Also here '1' is
 recommended value, but zLib knows of values 1 to 9.
 With nCompression >= 0, also a cPassword can be given for addtional traffic encryption,
 Initial connect to server ( e.g. LETO_CONNECT() ) is ever done in regularly traffic [ -1 ] mode.
 Without any function param, the active compression setting is returned without changing it.


      7.8 User account management functions

      LETO_USERADD( cUserName, cPass [, cRights ] )            ==> lSuccess
      LETO_USERDELETE( cUserName )                             ==> lSuccess
  New function ;-), now a user can be deleted
      LETO_USERPASSWD( cUserName, cPass )                      ==> lSuccess
      LETO_USERRIGHTS( cUserName, cRights )                    ==> lSuccess

      LETO_USERFLUSH()                                         ==> lSuccess
  Writes the changes in memory into file on storage, will be done else when server shuts down.

      LETO_USERGETRIGHTS()                                     ==> cRights
  Return the rights of the active user.

  Generally the user who want to change/ add/ delete user properties must have admin rights in case
  in letodb.ini any Pass_For_* is set to '1'.
  cRights are three letters: Y == allow, N == deny, in sequence for:
  admin == change/ add users; manage == console user; write == allow to change data in DBF tables.

  The whole password file on storage is now encrypted, and also after loading the content during
  server start into memory, the passwords are kept encrypted. They will be de-crypted only
  'on the fly' to countercheck the password for the one user who asks for access.


      7.9 Server variable functions

 Like a PUBLIC variable is for an application, this is something alike a public variable to all
 the connections/ applications/ and the server itself to exchange/ access/ modify content of that.
 The variables can be 'group-ed' into groups of variables, as example a group of variables used by
 one specific application, or a group of variables only for the server itself.

      LETO_VARSET( cGroupName, cVarName, xValue [, nFlags [, @xRetValue]] )
                                                               ==> lSuccess

 This function assign value <xValue> to variable <cVarName> from group <cGroupName>.
 <xValue> can be:
   boolean ( .T., .F. )
   integer ( without decimals, can be incremented and decremented )
   decimal ( integer with decimals, e.g. 4.321 )
   limited in size !:
   date    ( [NEW] date format, will be internally handled as YYYYMMDD
   string  ( [NEW] also binary string, means containg any char like e.g. CHR( 0 ) )
   array   ( [NEW] { ... } with any above item type, limited in size )

 String/ array are limited by default to be all in sum max 64 MB, maximum can be changed in
 letodb.ini with config option "Max_Var_Size". A single string/ array limits to 1/4 of total
 maximum ( default 16 MB ).
 This is for security reasons, so that crazy users cannot fill up the whole server memory.
 It is only allowed to assign a new value of same type to an existing variable.
 Group- and Var- names are NOT trimmed of white spaces, but char: ';' is an invalid char.
 IF <@xRetValue> is given by reference ( @ ), it will hold up the old value before the new
 one was set. This will only be done for boolean and numeric types, not for string etc. 

 Optional parameter <nFlags> defines the variable create mode/ limitations.
 These flags can be combined by aggregating ( + ) the constants:
 LETO_VCREAT    - [NEW as default] create variable if it doesn't exist;
                  doesn't hurt if variable already exists, this case without further effect
 LETO_VNOCREAT  - [NEW] set this if you do not want above automatic create
 LETO_VOWN      - own user variable (deleted after user disconnect)
 LETO_VDENYWR   - write deny for other users
 LETO_VDENYRD   - read deny for other users
 LETO_VPREVIOUS - only a valid flag for Leto_VarIncr() and Leto_VarDecr():
                  return the value before variable increment/ decrement

      LETO_VARGET( cGroupName, cVarName )                      ==> xValue

 Function return value of variable <cVarName> from group <cGroupName>

      LETO_VARINCR( cGroupName, cVarName, nFlags )             ==> nValue
      LETO_VARDECR( cGroupName, cVarName, nFlags )             ==> nValue

 Function increment/ decrement integer value of variable <cVarName> of group <cGroupName>.
 ! Only allowed for integer [ INT() ] variables without decimals!
 Remark that e.g. a result of a division: 'x / y' is ever a decimal value.
 So if in doubt use INT( value ) when you want to create/ update such a variable.

      LETO_VARDEL( cGroupName[, cVarName ] )                   ==> lSuccess

 Without <cVarName>, all variable members of <cGroupName> and the group itself are deleted.
 With given <cVarName>, variable <cVarName> is deleted from group <cGroupName>.

      LETO_VARGETLIST( [cGroupName [, nMaxLen]] )              ==> aList

 Function returns
 # without a string param for <cGroupName>:
   array with group names { <cGroupName1>, ... }
 # with valid <cGroupName>, without <nMaxLen> param or < 0:
   array with variable names in the group
 # with valid <cGroupName>, and with <nMaxLen> >= 0:
   two-dimensional array with variables: { {<cVarName>, <value>}, ...}
   <nMaxLen> == 0 means unlimited string values, else these are limited in size
   and cut off if longer as <nMaxLen>.
   IF <nMaxLen> > 0, array variables have a symbolic string value: "{ ... }"

      LETO_VARGETCACHED()                                      ==> xValue [NIL]

 It is an optimized form of LETO_VARGET( ... ) and like this can be used excellent in filter
 expressions. It returns the *last changed* LETO_VAR[SET|INCR|DECR]() variable *value*.
 ! To be used with care !
 When one specific connection set a variable, it will change the last cached value.
 If then used in a filter condition, and type of value [ string, numeric, .. ] changed,
 the filter gets invalid !
 If none variable was set before by this connection, NIL value is returned.
 Leto_VarGet[List]() have no influence, so you can query for other variables.
 Leto_VarDel() done by *same user* who set variable cache, will clear the value to NIL.
 Cached value is personal for one user/thread, so *others* can simultanous use same variable,
 or even delete it, without interfering the personally cached value.

 The tricky idea is following: first you create at client application a variable with:
    'leto_VarSet( "MY_GROUP", "MY_VAR", xValue, LETO_VCREAT )'
 then you define a filter condition for your workarea, example for a numeric:
    DbSetFilter( NIL, "field_var < leto_VarGetCached()" )
 Avoid 'ALIAS->' names in filter condition for mode: No_Save_WA=0: they are invalid.
 The only param what really matters is the filter text value, codeblock is optional.
 Function names requested ONLY in filter condition must be explicitely requested by a:
 REQUEST statement to be linked to your application.

 This gives the filter condition at client very! fast access to the value WITHOUT asking the
 server, also the server have very fast access to the cached last value.
 The connection which set filter can interactive change the content of this variable
 with a new leto_Var[Set|Incr|Decr]() to update the filter condition, as that will set
 a new cached value.
 Tip: if there are millions of records, and filter condition reduces them to just a few,
 it may increase performance lowering default amount of returned records with
 leto_setSkipBuffer() if not as many [ default: 10 ]records with one DbSkip() request needed.


 Leto_VarExpr*() function family dealing with memvars ( PRIVATE/ PUBLIC ) in EXPRession
 strings.  The expression <cExpression> itself must *not* be a valid executable expression,
 it is basically enough that a memvar name appears in it.
 So when you want to sync variables with server, do:
    cExpr := Leto_VarExprCreate( "Memvar1 [, MemvarX ]", @aArr )
 Repeatedly call after each occasion of changed value:
    Leto_VarExprSync( aArr )
 Clean up with:
    Leto_VarExprClear( aArr )

      LETO_VAREXPRTEST( cExpression )                          ==> lContainMemvar

 Return TRUE ( .T. ) if <cExpression> contains memvars

      LETO_VAREXPRCREATE( cExpression [, @aLetoVar ] )         ==> cModifiedExpression

 Return a modified <cExpression>, where the name of memvars are replaces by a connection
 unique Leto_VarGet(). Each of these Leto_Vars are created during the call with a
 Leto_VarCreate() containing the value of the memvar, and have the LETO_VOWN flag,
 so they will be automatic deleted with application end.
 If optional 2nd param <@aLetoVar> is given by reference ( @ ), a 3-dim array with
 LetoVars for related memvars is assigned to the param variable, wich will spare an extra
 call of Leto_VarExprVars().

      LETO_VAREXPRVARS( cModifiedExpression [, lOnlyMemvar ] ) ==> aLetoVar

 Scans <cModifiedExpression> result of Leto_VarExprCreate() for Leto_Vars, and return a
 3-dim array to be used with LetoVarExprSync() and Leto_VarExprClear().
 Optional param <lOnlyMemvar> have default TRUE ( .T. ): then only LetoVars with a
 related memvar are added into the result array.

      LETO_VAREXPRCLEAR( cModifiedExpression | aLetoVar[, lOnlyMemvar ] )
                                                               ==> lVarDeleted
 Scans <cModifiedExpression> for LetoVars with default TRUE ( .T. ), which have
 a releated Memvar. For each found LetoVar a Leto_VarDel() will be executed.
 Instead the string expression can be the result <aLetoVar> of LetoVarExprVars() used.

      LETO_VAREXPRSYNC( aLetoVar [, fSyncLetoToMemvar ] )      ==> lVarSynced
 For all LetoVars in <aLetoVar> a LetoVarSet() will be excuted, if the value of the
 related memvar have meanwhile changed ( but the type must be the same ).
 TRUE ( .t. ) is returned only for case of a sync was executed, in all other cases:
 LetoVar equal to memvar, no connection or empty/ invalid aLetoVar array FALSE (.F. ).
 Changing <fSyncLetoToMemvar> default TRUE ( .T. ) to FALSE will sync the reverse way:
 then a changed LetoVar value, maybe at server by an UDF or by other connection, will
 be applied to the local memvar.


      7.10 Calling udf-functions on the server

      LETO_UDF( cSeverFunc [, xParam1, ... ] )                 ==> xResult
 This function is called from client application. The string <cServerFunc> can
 optional contains a server connection string, minimum is udf function name:
 [ //ip_address:port/ ]funcname
 A <funcname> function should be defined on the letodb server.
 Udf function can return result (any type except pointers) to client.
 Examples of udf-functions are in the tests/letoudf.prg

      LETO_UDF( cCodeBlock [, xParam1, ... ] )                 ==> xResult
 If <cCodeBlock> contains a valid codeblock string, it is evaluated at server.
 Maximum length of <cCodeBlock> is 255 characters.
 Example: leto_Udf( "{|x| UPPER( x ) }", "to_upper" )

      LETO_RPC( cSeverFunc, xParam1, ... )                     ==> NIL !
 ! Use with care !, it needs well designed UDF functions.
 Function name is actually limited to 20 char length.
 It is a bit similar as for LETO_UDF, but when the function starts it will have NO workareas
 opened: it is like a fresh connection, independent from the one who started it.
 After such job is started, no waiting for the result will occure and only a boolean ( .T. )
 is returned to your application if successful started. You can start multiple jobs this way.
 This needs special care not to block others in the network from accessing their workareas,
 so it is advised to be used only in server mode 3 aka No_save_Wa = 1.

 This is higly interesting for tasks, where you need no feedback, e.g. some 'cleaning'
 or background calculating jobs.
 You can write a result of something to a DBF/ TXT and retrieve this way the result.
 Or you can exchange info with any connection with help of LETO_VAR*() function, or even manage
 the activity of your UDF with that server variable system -- an area left for many ideas ...

 But such UDF functions should be *VERY* careful designed, as they can NOT be stopped by the
 connection which started them, so better to test them intensive beforehand with LETO_UDF.
 Commonly it must end by itself, as such 'headless' UDF at server can else only be
 stopped by the management console, if the running UDF is designed to repeatedly check for:
 leto_UDFmustQuit(). If it ignores that, it is unstoppable until server shutdown ...

      LETO_UDFEXIST( cSeverFunc )                              ==> lExist
 leto_udfExist check the existance of udf-function at the letodb server.
 <cSeverFunc> parameter is the same as for leto_udf().

      LETO_PARSEREC( cRecBuf )
 ! deprecated - removed !

      LETO_PARSERECORDS( cRecBuf )
 ! deprecated - removed !
 see new LETO_DBEVAL() as powerful alternative, or sample of UDF_dbEval() in tests/letoudf.prg


      7.11 Functions for bitmap filters

 If letodb compiled with rdd BMDBFCDX/BMDBFNTX, then there is support
 the following functions:

      LBM_DbGetFilterArray()                                   ==> aFilterRec
      LBM_DbSetFilterArray( aFilterRec )                       ==> bFilterActive
      LBM_DbSetFilterArrayAdd( aFilterRec )                    ==> bFilterActive
      LBM_DbSetFilterArrayDel( aFilterRec )                    ==> bFilterActive

 Purpose and the parameters of these functions is the same as for the
 corresponding BM_*() functions.
 < bFilterActive > indicates that there is a filter active at server side after the call.
 This can be also verified with LETO_ISFLTOPTIM() returning TRUE.
 An active filter can be cleared by a common DbClearFilter()/ SET FILTER TO.

      LBM_DbSetFilter( [<xScope>], [<xScopeBottom>], [<cFilter>] )
                                                               ==> nil
 This function set bitmap filter by current index order and for condition,
 defined in <xScope>, <xScopeBottom>, <cFilter> parameters.
 The current record after LBM_DbSetFilter() is the first record satisfying filter condition.


      7.12 Miscellaneous Functions

      leto_Hash( cText )                                       ==> nHashValue
 Returns a 32bit hash value of the given <cText>, useful to fast search for a string.
 Depending on how LetoDBF is compiled, MurMur3 hashing algorithm [default] or a homebrew
 algorithm is used. There may be rare collisions, means that it is possible that two <cText>
 resulting into same <nHashValue>. To be at safe side, ever compare the text char-by-char,
 after a valid hash-value is found.



      8. Utils

      8.1 Server Management utility

 There are two management utilities, Windows GUI and all OS console.
 Sources lay in the utils/manage directory.

      8.1.1 All OS console

 For all OS, also Windows, there is a ! NEW ! console utility to be found in:
 utils/manager/console.prg. Easily build it with just a:
    hbmk2 console
 The executable will be found afterwards in the "bin" directory.

 Full parameter set is: console[.exe] [ IP[:port ] ] [ username ] [ password ]
 These params can be set in letodb.ini with option: <Server> for running it local at server.
 But as this is a OS independent remote console, you can watch the server from any station,
 so you commonly will give IP address as first param.

 ! Note the resizeable window, the browses will dynamical stretch into given screen size.
 Displayed information refreshes automatically, as longer you typed no key the greater this
 interval will get, to less interfere server activity.
 If no more needed, end it and restart it by occasion. It will end automatic with shutting down
 LetoDBf server.
 Surfing in the four Browses' is done with mouse-clicks/-wheel or keyboard ( TAB, cursor key )
 Also note this "Menu" button top-left on screen for further actions, like killing connections,
 adding users to the authentication system. For some actions you have to scroll to the wanted
 connection, then to change into that menu. Positioned at console connection shows a summary in
 the other browses, positioned at a specific connection gives detailed informantion about which
 WA is just selected and what index order is active.

      8.1.2 Window GUI management console

 The GUI version builds with help of HwGUI library, so you need that package ready compiled.
 Adapt in the manage.hbp file at top the path to your HwGUI package root directory.
 Then build the executable with:
    hbmk2 manage
 The executable will be found afterwards in the "bin" directory.


      8.2 Uhura

 This is for automatic detection of the server, very helpful in networks with dynamical assigned
 IP addresses. Build the executable in utils/uhura/uhura.prg with a: hbmk2 uhura
 The executable will be found afterwards in the "bin" directory.
 Start it at the same machine, where the LetoDBf server is running.
 Linux! user can give with first param an interface name like "eth1" at which uhura will listen.
 To stop Uhura, call: uhura [ interface ] STOP

 Then look in utils/uhura/detect.prg for example of function use: detectServer( "letodb" ).
 It sends out a broadcast request, and returns the answer of Uhura with the IP address of the
 machine with Uhura, aka the IP address of LetoDBf server.
 All you need for that is to be found in detect.prg, you need to link the helper functions to
 your application, as i have not integrated this functionality into LetoDBf. [ ToDO ? ]

 The call of: uhura help
 will display info about available network interfaces at this machine.
 This help screen will also pop up in case of problems, example when try to stop Uhura and she
 is not running, or in case of invalid interface name ...



      9. Server-side functions

 !!! CHANGED: nUserStru deprecated !!!
 The first parameter for such UDF functions formerly was numeric type, it is now obsolete.
 As result the places for your params in your older defined functions rise up one place in order.

 These functions can be run from the client by function leto_udf,
 and includes also functions defined in the file letoudf.hrb, loaded with server start.

      leto_SetEnv( xTopScope, xScopeBottom, xOrder, cFilter, lDeleted )
      leto_ClearEnv( xTopScope, xScopeBottom, xOrder, cFilter )
 These functions save with setting and later restore possible changed conditions by your UDF.
 Just fill in all param that are needed to be set, and call leto_ClearEnv() with the same params set.
 Scope with filter example: leto_SetEnv( "1000", "2000" ) => leto_ClearEnv( "1000", "2000" )

      leto_Alias( cClientAlias )                               ==> cServerAlias
 This function return the ALIAS name used at server for a given client side alias <cClientAlias>.
 Side effect is, if ALIAS name is valid, it will change the active selected workarea at server.
 The returned server ALIAS then can be used in usual RDD-operations.
 This function is mainly needed for mode 'No_Save_WA=0', here to use additional different workareas
 than the one which was actve when an UDF was initalially called.
 In server mode: 'No_Save_WA=0', ALIAS names at server and ALIAS name at client are different.
 All workareas used in an UDF are not available for other connections as long as the UDF is working.
 In server mode: 'No_Save_WA=1', ALIAS names at server and client side are the same.
 Also in this mode there is no exclusive restriction of workarea use only for the UDF connection,
 and using leto_Alias() become somehow obsolete.

      leto_RecLock( [ nRecord ], [ nSecs ], [ bAppend ] )      ==> lSuccess
  leto_Reclock() function locks record with <nRecord> number, or the current record if nRecord is
  left empty, for data change access.
  <nSecs> param is only available in server file open mode: No_save_WA = 1.
  With given optional <nSecs> [ can be decimal: 1.5 ] it will wait for success if not
  immediate succesfull.
  These locks are internal known for your connection, as if done locally.
  If you append a record in your UDF, which is locked after a successful append, you have to only
  register it for your connection, done by third param set to TRUE ( .T. ) == append lock.

      leto_RecUnLock( [ nRecord ] )                            ==> Nil
  leto_RecUnlock function unlocks a locked record with <nRecord> number, or the current record.

      leto_RecLockList( aRecNo )                               ==> lSuccess
  leto_ReclockList function locked records with number in the <aRecNo> array.
  If any record isn't locked, all records are unlocked, and function returns
  .F. result.
  This function can be used at the server from letoodf.prg module, or from
  client by a call leto_UDF( "leto_RecLockList", aRecNo ).

      leto_TableLock( [ nSecs ] )                              ==> lSuccess
      leto_TableUnLock()
  [ deprecated: no more <nFlags> param available ]
  This will lock the whole DBF table, not only a single record, for data change access.
  <nSecs> param is only available in server file open mode: No_save_WA = 1.
  With given optional <nSecs> [ can be decimal: 1.5 ] it will wait for success if not
  immediate succesfull.

      leto_SelectArea( nAreaId )                               ==> lSuccess

      leto_Select( [ ncClientAlias ] )                         ==> nWorkareaID
 Selects to the workarea given by workarea ID oder ALIAS ( if valid )

      leto_AreaID( [ cAlias ] )                                ==> nAreaId
 Function return internal workarea-ID of current/ by ALIAS name given workarea,
 0 if not found

 [ Following four functions have been renamed for LetoDBf -- to be the same like in Harbour,
   but with prefix: 'leto_' ]

      leto_DbUseArea( [ cDriver ], cFileName, [cAlias, lShared, lReadOnly, cdp ] )
                                                               ==> nAreaId
 Parmeters order like DbUseArea(), EXCEPT the omitted first: table is ever in a new workarea,
 you can NOT preselect the Workarea-ID.
 If not given, cAlias is created from filename, cDriver is the last used setting,
 lShared is the default of _SET_EXCLUSIVE from LetoDBf client!,
 lReadonly is default .F. and codepage is settings used at client side.

      leto_DbCreate( cFilename, aStruct, [ cRDD ] , [ lKeepOpen ], [ cAlias ], [ cCodePage ] )
                                                               ==> lSuccess
 Creates a new database, some param are optional --  cFilename can be prefixed with 'mem:'.
 cRDD can be one of: DBFNTX, DBFCDX, DBFFPT, SIXCDX, DBFNSX -- if not given the last used RDD.
 lKeepOpen default is close file after create.
 cAlias is only needed, if the fresh created table should stay open.
 cCodepage will be the last used -- best to leave ever empty, especially if you not exactly know
 the cause why you want to set it.

      leto_OrdListAdd( cBagName [, ncAreaID ] )      ==> lSuccess
 It is recommended, to use the second param ncAreaID, best as cALIAS or as workarea number to on the
 safe side. LetoDBf will add all index keys in one [ multi tag ] index file at once.
 nAreaID is optional, if not given the actual selected workarea is used.

      leto_OrdCreate( [ ncAreaID ], cBagName, cKey, cTagName,
                      lUnique, cFor, cWhile, lAll, nRecNo,
                      nNext, lRest, lDesc, lCustom, lAdditive )
                                                               ==> lSuccess
 It is recommended, to use the first param ncAreaID, best as cALIAS or as workarea number.
 But the ID at first place can also be omitted, then the order will be created for that WA.
 cBagName is in any case the second param, cKey the index expression as string.
 For the other values see Harbour documentation for OrdCondSet().

      leto_DbCloseArea( [ ncAreaID ] )                         ==> lSuccess
 Close the active or by numeric or by string ALIAS given workarea.

      leto_DbEval( cbBlock, cbFor, cbWhile, nNext, nRecord, lRest )
                                                               ==> aResults

 This UDF function is same kind as DbEval() of Harbour. The three codeblocks may be given
 instead as a codeBlock within an UDF function or as string parameters when called 'directly'
 from a client per leto_udf( "leto_DbEval", "cBlock", ... ).
 If no cbBlock is given ( nor as CB neither as not empty string ), you get the raw content of a whole
 record in LetoDBf format, else the result of evaluated cbBlock for each record converted to string.
 This is a very migthy functionality, so should be used with some care.
 cbFor means a condition to be valid, else the record is unprocessed skipped over [ but counts for nNext ].
 cbWhile means an expression to stop when it results false ( .F. ). If empty, default is: '.NOT. EOF()'.
 Both cbFor and cbWhile must return logical ( boolean ) values.
 If nNext is 0, all records down to EOF() [ or cbWhileare processed, else the amount as given.
 lRest == true ( .T. ) starts from actual record, lRest == false ( .F. ) from topmost, first record.

 The functions leto_DbUseArea, leto_OrdListAdd, leto_DbCreate(), leto_OrdCreate, leto_DbCloseArea,
 leto_RecLock, is intended for using in UDF-functions instead of rdd functions:
 dbUseArea, OrdListAdd, OrdCreate, dbCloseArea, RLock, dbUnlock

      leto_UDFMustQuit()                                       ==> lTrue
 If the console monitor quits a UDF or RPC thread on server, it sets an internal variable for this
 thread. This functions reports about that, then it shell end.
 Example: DO WHILE .NOT. leto_UDFMustQuit(); do something; ENDDO; RETURN
 Especially useful for threads started with Leto_RPC().

 Following functions same as for LetoDBf client, now also without nUserStru.
 Note: xValue will be 'NIL' in case of not found cGroupName/ cVarName.

      LETO_VARSET( cGroupName, cVarName, xValue[, nFlags )     ==> xValue
      LETO_VARGET( cGroupName, cVarName )                      ==> xValue
      LETO_VARINCR( cGroupName, cVarName )                     ==> nValue
      LETO_VARDECR( cGroupName, cVarName )                     ==> nValue
      LETO_VARDEL( cGroupName, cVarName )                      ==> lSuccess
      LETO_VARGETLIST( [cGroupName, [lValue]] )                ==> aList

 special:
      LETO_VARGETCACHED()                                      ==> xValue

 ! see explanations for purpose at client side function leto_VarGetCached() !



      10. Abbreviations and remarks

 Maybe in this text used abbreviations:

    CP    CodePage; a set of characters representing 'normal' and national special chars.
          Sometimes also abbreviated with CDP.
    DF    Date Format; a pattern how a "DATE" value is presented, where letters:
          d == day, m == month, y == year plus extra characters, e.g. "dd.mm/yyyy"
    IP    Internet Protocol address; IPv4 address
    RPC   Remote Procedure Call; execution of a procedure not at client machine, but at
          machine where LetoDBf runs.
    RTE   RunTime Error; may occure if application developer tried something he better
          have checked before doing so. Or when something really unexpected happened.
    TCP   Transmission Control Protocol; packet-based; what E.T. missed
    UDF   User Defined Function; in opposite to Harbour commands one defined by the user.
          Commonly containing Harbour commands.
    WA    work area; a logical environment representing a database table.
          Commonly they have a name retrievable with ALIAS()

 Here some remarks on the fly, which are waiting for another place to be explained.
   # It is not possible, to overwrite an used DBF, aka a DbCreate() will fail with RTE
     if already opened by another user/ connection of other thread.
   # One database table can be used only with same codepage for all other connections.
   # ALIAS() names for HbMemIO tables ( these with "mem:" prefix ) are always the same
     as the name of the DBF itself. It is not possible to give it another ALIAS, so you
     can open such a table only once per connection.
   # You cannot use *in your UDF* in mode No_Save_WA=0 !! [ the very only exception ] ALIAS names
    "Exxxxx", where "xxxxx" is a numeric value, aka: "E123" :-)
   # Using temporary index orders [ created in Server OS temporary path ] are only possible in
     mode: NO_Save_Wa = 1.
   # maximum numeric value for a field: "N", 20, 0 is: +/- 9223372036854775807
     one more (or less) and rounding will occure with trailing zeroes


-------------

      A. Some Internals

      Some explanations, what happens 'under the hood'.
 Server is developed Multi-Threaded, what is 'a bit' compareable of running multiple instances
 or many programs on a computer. This will use all the CPUs of your server and span the load over
 them.
 There runs a main thread, responsible for incoming connection request, which will start for every
 connection to a user a new thread. All the communication to one connection is then done in this new
 thread. During server start are also two further threads started: one is like the main thread
 responsible for handling the initially communication sequence for the second socket (*) for a connection.
 This only will happen for multithreaded linked applications by setting hbmk2 option; '-mt'
 If enabled, one other thread periodically checks for dead connections, and will close them in that case.
 This can be adjusted by setting the timespan for check to zero [ default is: Zombie_Check = 0 ]
 in letodb.ini.

 (*) Second socket: this second port to the server is used also for mentioned 'dead connection* check.
 And used for a group of requests by client to the server: these requests have common, that you regularly
 only receive an 'ACK' == 'succesful done' from server. This 'ACK' will NOT be send, and only in case
 of an error a negative 'NO-ACK' will be send over this second port, so i called it the 'error socket'.
 This technic drastically speeds up e.g. updating records at server, as the client must not wait
 for the positive 'ACK', can immediate proceed and also network traffic is avoided.
 But the client will not miss such a negative 'NO-ACK': in such case a runtime error message will
 pop up 'delayed' with the known Harbour error handling system at client/ user side.


 Or in my other words:
 ( https://groups.google.com/forum/#!topic/harbour-users/q_ZqmOB6Sns )
 In the classic 'client-server' model, the client send a request and receives a response from server
 - at any time. Each of these request/response need a whole 'package' to be send, even commonly less
 filled with just a few bytes instead max possible ~ 1500 Bytes.
 The amount of packages per timespan is limited, so despite fullspeed communication you see less
 network traffic.

 There is a group of requests, for which the server send an 'ACK' aka: done!
 This touches the package limit and further the client have to wait for it.
 Best example: update data, aka: REPLACE .. WITH .., lead to billions of ducky ACK ACK ACK to wait for
 after each update request. :-)  So i wanted to spare the ACK, but *not to miss* the very rare NO!-ACK.
 ( example in case: when record not locked )

 Solution: the client open a second socket to server, at which a second thread is waiting for incoming
 info ( reverse the first socket, where the server is waiting for incoming )
 So the server do not send an ACK, but only in case of problem a NO!-ACK to this second socket.
 The second thread at client receives it, and prepares a mutex secured 'global' error object.
 This is checked by the first thread for not empty, after each send request ( and when going idle mode ).
 In case of a filled error object, first main thread let runtime error system pops up with that.
 This i named the 'delayed' error :-)

 Summa: this lead to a significant performance increase, as client can push out all these 'data update'
 requests ( and some other request like UNlock, etc ), one after another without delay for the ACK,
 nice filling the queue at server. In case the client have no MultiThread capability, no second socket
 is opened, the server acts 'classic' for this connection, makes ACK ACK ACK :-)


 Requesting and detaching workareas: many other server, also HbNetIO, will open a DBF table again for each
 connection after user command: DbUseArea(). Detaching an opened workarea means, it is at server side 'closed'
 for this connection, and put into a pool of 'detached workareas'.
 If the connection needs the workarea again for next action, it requests it out of the pool of detached
 workareas. Immediate after each action, the WA is detached again.
 If another connection want to open same DBF table, it instead requests it out of the pool of detached WA,
 and detaches it again after action is done.
 In this kind, a DBF table is opened only one time by server, and by detaching/ requesting then exchanged
 between multiple connections. This have significant performance advantage for below explained default mode 1.
 Disadvanttage is: as long as one connection/ UDF needs this WA for action, no other connection can use it.
 Further it is at only one single Workarea for one connection active: this is important for UDF what need more
 than one WA.

 File open modes: LetoDBf knows three (4) modes, choose the appropiate for you needs.
 #1# Share_Tables = 0, No_Save_WA = 0 [ default if not explicitely set ]
 The server exclusive use the DBF files, so no third party software ( non LetoDBf user ) can access
 DBF tables opend by LetoDBf server. This is the fastest server mode.
 Workareas are exchanged between connections using above explained detach/ request technic.
 #2# Share_Tables = 1, No_Save_WA = 0
 Second mode is equal, except the DBFs are in this mode opened according the first connection request.
 If first user of a DBF table open it with shared attribute for DbUseArea(), also other connections can
 use it. If first connection open DBF table exclusive, no other can use that workarea simultanous.
 Open a DBF table in shared mode gives third party / non LetoDBf users the possibility to work simultanous
 with DBF tables opened by LetoDBf server. This mode is a bit slower, but when concurency access is needed
 the way to go.
 #3# No_Save_WA = 1, Share_Tables option will indicate 3rd party software is active along the server.
 The workareas are NOT exchanged between connections using detach/ request technic.
 At server side each DBF is opened in exact the same workarea-ID and ALIAS name as at client side, and
 for every next connection once again -- further shared or exclusive, like client requested the DBF table.
 In this mode, all relations and filters at client side are active at server side.
 This will enable index- or filter- expressions on FIELDs of a relationed workarea.
 This mode is generally the slowest mode, but with fast filter action. It have great advantages for long running
 UDF functions and may have performance advantages for very many simultanous users accessing the same table,
 as in this mode one connection must not wait for a detached workarea from other thread.

 From POV of performance, mode #1# is the fastest, #3# in between, slowest is #2#.
 But expect only some % performance difference between the modes, as the most limiting factor is the TCP/IP network
 itself: the count of data-packages in a timespan is limited, and each request to server and response from it, is
 a whole data package, often filled with much less content as a single package can take (~ 1500 bytes).

 ...


      B. Links, to 3rd party etc.

 MurMurHash version 3, 32bit hashing results, public domain, taken out of:
      https://github.com/aappleby/smhasher
 LetoDBF distributes only PMurHash.[c|h]

 LZ4 compression algorithm, BSD license, extreme fast compression and even more fast de-compression,
 used for on demand realtime network traffic:
      https://github.com/lz4/lz4
 LetoDBF distributes only the 'lib' directory with BSD license of that contrib.



 whish all possible fun !
 elch
