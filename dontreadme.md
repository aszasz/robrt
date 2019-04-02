### Random stuff goes on this file

## What does Robrt do?

Each time there is a commit push in a given github repo 
or a pending pull request is updated (in a given github repo),
given the proper set up, robrt can build and deploy the project
with the changes as requested in files in the project repository
itself.

(Robrt must be installed and run in a server 
(a computer internet connected) properly set 
so it can monitor several github repos,
provided these given repos are also properly set.

The magic happens using Docker in the server (again:
the repo itself will contain the instructions regarding
the assembly of the Docker container to be used 
(copied or created from scratch).

Yes, there are security issues that need to be worked
before using robrt in a repository where public push 
or pull requests are allowed (a pull request would 
make a build and a deployment of its own).

## On the server

So here is what is required to make Robrt work:

In the (assumed linux with system d) server
(`#` indicates command given as root):

- the robrt package (`robrt.tgz`)  generated elsewhere 
(likey on your computer, if you are reading this,
once you've cloned this repo.
Robrt is *not* continuously integrated using Robrt itiself)

- nodejs: JavaScript run-time environment that executes 
JavaScript code outside of a browser; built on Chrome's V8 
JavaScript engine.

- npm (that shall manage to get js libraries required by
the robrt package and make it into an executable -- within env)

- docker

- nginx (it can work without it, but is not on the safety side)

`# apt-get install -y nginx nodejs nodejs-legacy npm docker.io`
does it in debian-derivative distros 

`# npm install -g robrt.tgz` does the unpacking
(where tgz is the packing done elsewhere)

creating a file  `/etc/system/systemd/robrt.service` containing
the below makes robrt a service to systemd 
(System D, the "service manager"):

```
[Unit]
Description=Robrt CI robot
After=network.target

[Service]
Type=simple
PIDFile=/run/robrt.pid
ExecStart=/usr/bin/env robrt listen 6667
TimeoutStopSec=30
KillMode=mixed
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

when nginx is installed, a file corresponding to the above
-- making ngnix a service to be controled by systemd -- 
is created for and `# systemctl start nginx` will get it going


then a section like the following shall be added to 
`/etc/nginx/sites-available/default`:

```
	# Proxy Robrt
	location /robrt/incoming {
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header X-NginX-Proxy true;
		proxy_pass http://localhost:6667;
		proxy_ssl_session_reuse off;
		proxy_set_header Host $http_host;
		proxy_cache_bypass $http_upgrade;
		proxy_redirect off;
	}
```

after creating it 

```
# systemctl daemon-reload 
# systemctl enable robrt
# systemctl restart robrt 
# nginx -s reload
```

shall have it running.



- robrt executable: a js generated with haxe: haxe is both a language and a compiler: the compiler reads the haxe language and generates (we say target) js (as well as cpp, java, python, lua, php, c#, besides bitecode for actionscript3, neko and hashlink)
(letsencrypt seems good too)

- robrt.tgz is a pseudo executable packed with ` npm #pack


You build robrt with haxe here (maybe here is a submodule of your server set up)

```
haxe build.hxml
```

This will build a js to run with node; calling node robrt.js would do the work (node being a program installed in the computer that wants to run robrt that interprets javascript... still we have 

- npm (node package manager... despite attempts to exclude the name node from the name), which is a way to make easier to run and maintain dependencies for running all the (js) programs required for the project. One must run something like  `npm install --global` so npm makes it an executable file (within /usr/bin) and 


Currently Build.hxml tells me that the compile path is /src and sets a compiler flag (directive) use\_rtti\_doc (runtime Type Information documentation)



then target js with a file named robrt.js (that will run called by npm... with comand npm start, that reads what is to run)

using libs:
- `version` (tool for version numbering?)
- `hxnodejs` (tool for working with npm and nodejs idiosyncrasies ? )
- `jmf-npm-externs` (jonas malaco filho tools for handling externs in the npm environment ? more idiosyncrasies)
- `continuation` (a mess to reduce callbackhell... one writes direct functions, continuation helps to change it into callback hell)
- `libyaml` (YAML is a human friendly data serialization standard for all programming languages)

the directive 'js-source-map` tells haxe to provide a file that holds the correspondence between the generated code in javascript with the source code in haxe, so runtime errors can point (at least the direction) of the problem. It also requires the below
```
    js.Lib.require('source-map-support').install();
    haxe.CallStack.wrapCallSite = js.Lib.require('source-map-support').wrapCallSite;
```
which is enabled soon at the main of entry point (Robrt.hx)

--macros are interpreted, on this particular case, both macros use haxe.macro.Context class static function OnAfterGeneneration, that adds a callback function callback which is invoked after the compiler generation phase:
- adding the shebang #!/usr/bin/env node as the first line for the robrt.js output
- (if not Windows) making the file robrt.js executable using the system command chmod 

hmm install would look up for all libs and dependencies and save them under 


I don't how this correspondence is used in run time. A should try to run something simple with it then

If a class is anotated with :rtti metadata (or a class extends a class that was anotated with :rtti metadata) the compiler will generate runtime type information for it.
(This probably means that the program can use information about the type in runtime. (Does that mean going away from type safety?)


what else would github folder contain, besides hook?

js could contain other stuff outside npm? scripts to go on a web page?

Tests are inside source for thing that are not a library, is that the idea? Why does it get the name unit (instead of test)?

### Event
Event class holds all the accepted/possible/imagined events robrt must consider/deal with. So they are listed here

So one can tell robrt may:
-start (started): while it does not know of next thing done while no error of fail returned yet.

- open repo: while it does not get a response
--then it may get a repository error 
--or a message from repository that there is a failed merge (the expected commit to build could not be merged... so there is no material to proceed)

- prepare: if we reach this point, it means we found the repository and a commit to build, the repo must have a json file with instructions about preparation for build, building and/or export.
Preparing is about creating the environment requirements (install/verify necessary tools) to build the commit
It may fail by:
-- not finding the json file (.robrt.json per default?)
-- not being able to parse the json file
- not finding the prepare instructions in the parsed file
-- instructions to prepare fail (cannot be executed/are not understood)

- build: it is about making / changing the project code into something that can be executed. If this point was reached, preparation (as requested) succeeded.
It may fail by:
-- missing build instructions (it might continue still?)
-- no-repo-build (¿missing the repository build?)
-- failure while following build instructions
-- build instructions return with error

- export: moving this that can be executed to be executed somewhere. If we get here, we had a build success (¿or no build?).
This can "fail" by:
-- not having instructions for exporting
-- return error while following instructions

- Done: if there was export instructions, we have export Success and it is Done.

### Server Config

This class defines that:

-the robrt server will may look up for a list of repositories
(¿a file within the server with the repos it watches shall correspond to this?)

- each repository has
-- name: the path to reach it








