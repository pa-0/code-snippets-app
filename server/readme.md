# Server application framework

The server uses Qt5 libraries, a driver for the SQLite3 database and an external qt-based library - [QHTTPEngine](https://github.com/nitroshare/qhttpengine). Its source files are included with the project^1)^ code-snippet-app, so you don't need to download it additionally. The steps necessary to build it from source will be outlined for each platform below.

## Building and running a project on Ubuntu 20.04

### System configuration

It is assumed that the user has the qt5 libraries installed globally and the driver to support the SQLite3 database.

If there is not, it can do it like this:

`$ sudo apt-get install qt5-default`

`$ sudo apt-get install sqlite3`

### Building and launching a project

Building (locally, in the project folder) the QHTTPEngine library:

1.  `$ cd server/lib/qhttpengine`
    
2.  `$ mkdir build && cd ./build`
    
3.  `$ cmake ..` - This step may generate an error related to the local installation of the library. You should ignore it and move on.
    
4.  `$ make install`
    

#### Create a database. Being in the catalog :`server`

5.  `$ cd database && ./unix-setup.sh`

#### Building a project. Being in the catalog :`server`

6.  `$ mkdir build && cd build`
    
7.  `$ cmake ..`
    
8.  `$ make`
    

#### Launch the application. Being in the catalog :`server`

9.  `$ cd bin`
10.  `$ ./server`

#### Running unit tests:

11.  `$ cd bin`
12.  `$ ./test`

^1)^ The only change in the source code is changing the variables , , in the CMakeLists.txt file to paths within the application folder (so as not to install the library globally). It can result in examples included in the library not working.`BIN_INSTALL_DIR``LIB_INSTALL_DIR``INCLUDE_INSTALL_DIR`

## Build and build on Windows 10

### System configuration

It is assumed that the user has the Qt5 libraries, the driver for the SQLite3 database, and the Visual Studio package (tested with Visual Studio 2019) installed globally.

#### Installation tips:

-   Qt5 - download and run the online installer from [the Qt website](https://www.qt.io/download-open-source), then follow the instructions to download the latest version 5 subversions of the Qt libraries.
-   SQLite3 - follow the instructions on the [page](https://www.tutorialspoint.com/sqlite/sqlite_installation.htm).

### Building and running a server

#### Download the repository and extract it to the desired folder. In the console (**Developer Command Prompt for VS 2019** or equivalent for another version of Visual Studio), go to the folder with the project, then:

1.  `> cd server\lib\qhttpengine`
    
2.  `> mkdir build`
    
3.  `> cmake ..`  - you may need to specify the generator (e.g. option). The list of generators available in the system can be obtained with the command. This step may also generate an error related to the local installation of the library. You should ignore it and move on.`/G "Visual Studio 16 2019"``cmake /help`
    
4.  `> msbuild INSTALL.vcxproj`
    

#### Creating a database starting in the directory:`code-snippet-app\server\`

5.  `> cd database`
    
6.  `> .\windows-setup.bat`
    

#### Building a project starting in the directory :`code-snippet-app\server\`

7.  `> mkdir build`
    
8.  `> cd build`
    
9.  `> cmake ..`- you may need to specify the generator (e.g. option). The list of generators available in the system can be obtained with the command.`/G "Visual Studio 16 2019"``cmake /help`
    
10.  `> cd ..\src`
    
11.  `> windows-after-build-setup.bat`
    

#### Run:

12.  `> cd bin`
    
13.  `> .\server.exe`
    

# Testing

The server listens on port 8000 at (). It accepts queries from body in json format with fields:`app``localhost:8000/app``POST`

-   `author` - string (e.g. ) max 30 characters`QString`
-   `title` - string max 100 characters
-   `created` - class object converted to a number in **Unix time** format with accuracy to the second (`QDateTime``QDateTime::toSecsSinceEpoch()`)
-   `lang` - one of the language identifier strings allowed by the variable`Snippet::availableLangs_`
-   `content` - string with a maximum length of 1000 characters

and queries . You can create queries to search for snippets that match a given pattern according to the following query fields in the URL path:`GET`

-   author_subsequence - string no longer than 30 characters, if empty - any author
-   title_subsequence - string no longer than 100 characters, if empty - any title
-   created_from - start date of the search in the same form **Unix time** as above or empty if unspecified
-   created_to - end date of the search in the same form **Unix time** as above or empty if unspecified
-   lang - one of the strings identifying the language or an empty string denoting any language

##### Example: it will search for snippets whose author contains in the name and the programming language is c++.`localhost:8000/app?author_subsequence=usernam&lang=c++``usernam`

The answer to a query without a request to search for specific snippets is a list of up to 5 snippets in json format recently added by queries (formally, these are the objects with the highest id in the database).`GET``POST`

All fields are optional and can be combined as desired.

The result of each valid query is json - a list of snippet objects. It can be empty.`GET`

As a result of an incorrect query, the error code **Bad Request** is set in the socket and an appropriate message should be returned in the json field.
