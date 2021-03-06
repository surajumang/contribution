# CHECKSTYLE-TESTER

checkstyle-tester is a tool for Checkstyle report generation over very [basic set of java projects](https://github.com/checkstyle/contribution/blob/master/checkstyle-tester/projects-to-test-on.properties).
It consists of two Groovy scripts: launch.groovy and diff.groovy. Thus, in order to use the tool make sure you have the Groovy runtime installed on your developer machine (min required version is 2.4.8).

## [launch.groovy] Checkstyle report generation

**launch.groovy** is a script which allows you to generate Checkstyle report over target projects. It invokes Maven Checkstyle plugin. In order to use the script you should run the following command in your command line:

```
groovy launch.groovy --listOfProjects projects-to-test-on.properties --checkstyleCfg my_check.xml
```

or with short command line arguments names:

```
groovy launch.groovy -l projects-to-test-on.properties -c my_check.xml
```

If you want to force Maven Checkstyle Plugin to ignore exceptions:

```
groovy launch.groovy --listOfProjects projects-to-test-on.properties --config my_check.xml --ignoreExceptions
```

or with short command line arguments names:

```
groovy launch.groovy -l projects-to-test-on.properties -c my_check.xml -i
```

The script receives the following command line arguments:

**listOfProjects** (l) - path to the file which contains the projects which sources will be analyzed by Checkstyle during report generation (required);

**config** (c) - path to the file with Checkstyle configuration (required);

**ignoreExceptions** (i) - whether Checkstyle Maven Plugin should ignore exceptions (optional, default is false).

**ignoreExcludes** (g) - whether to ignore excludes specified in the list of projects (optional, default is false).

When the script finishes its work the following directory structure will be created in the root of cehckstyle-tester directory:

*/repositories* - directory with downloaded projects sources which are specified in projects-to-test-on.properties;

*/reports* - directory with Checkstyle reports. 

Checkstyle reports will appear at 'reports' folder (checkstyle-tester/reports). 
**First run** should be executed with all rules enabled to make sure that new check does not fail. 
You may see failure of `TreeWalker` but as long as it is applied for no-compilable sources (test, resources) you don't need to worry about it.  **Second run** shall be done to prove that check is working correctly and for this one select the most accurate repositories.

Please use `-Dcheckstyle.consoleOutput=true` option if you need to see all Checkstyle errors in console, but expect it to print very and very much lines of terminal logs in this case. Launch command in this case will be:

```
groovy launch.groovy projects-to-test-on.properties my_check.xml -Dcheckstyle.consoleOutput=true
```

**Attention:** this project by deafult uses released version of Checkstyle and sevntu.checkstyle
If you you need to use custom (snapshot) versions please update pom.xml to reference that versions ([checkstyle version](https://github.com/checkstyle/contribution/blob/master/checkstyle-tester/pom.xml#L29), [sevntu.checkstyle version](https://github.com/checkstyle/contribution/blob/master/checkstyle-tester/pom.xml#L34)), and please make sure that custom(newly generated) versions are located in your local maven repo 

```
ls  ~/.m2/repository/com/puppycrawl/tools/checkstyle/
ls  ~/.m2/repository/com/github/sevntu/checkstyle/sevntu-checkstyle-maven-plugin
```

to build SNAPSHOT version of `checkstyle` please run in his repo

```
mvn clean install -DskipTests -Dcobertura.skip=true -Dfindbugs.skip=true -Dpmd.skip=true
```

If you want to validate new check from `sevntu.checkstyle`(https://github.com/sevntu-checkstyle/sevntu.checkstyle) project, 
before you use "launch.groovy" you need to clone it and deploy to your local maven repo by following command

```
./deploy.sh --maven
```

WINDOWS USERS:

*./deploy.sh* can be luanched on Windows OS by usage [https://www.cygwin.com/](Cygwin).
Preinstall default commands - http://www.appveyor.com/updates/2015/05/30 search for "Installation command used:"
Follow example how we do this in Windows CI server - https://github.com/checkstyle/checkstyle/blob/master/appveyor.yml#L71 search for matrix item "checkstyle-tester on guava"

=============================================

## [diff.groovy] Diff report generation

In order to generate a compact diff report before and after your changes you can use diff.groovy script which performs all required work automatically. Note, diff.groovy ignores excludes specified in the list of projects file.
Please execute the following command in your command line to run diff.groovy:

```
groovy diff.groovy --localGitRepo /home/johndoe/projects/checkstyle --baseBranch master --patchBranch i111-my-fix --config my_check.xml --listOfProjects projects-to-test-on.properties
```

or with short command line arguments names:

```
groovy diff.groovy -r /home/johndoe/projects/checkstyle -b master -p i111-my-fix -c my_check.xml -l projects-to-test-on.properties
```

If you want to specify different Checkstyle configs for base branch and patch branch use the following command:

```
groovy diff.groovy --localGitRepo /home/johndoe/projects/checkstyle --baseBranch master --patchBranch i111-my-fix --baseConfig base_config.xml --patchConfig patch_config.xml --listOfProjects projects-to-test-on.properties
```

or with short command line arguments names:

```
groovy diff.groovy -r /home/johndoe/projects/checkstyle -b master -p i111-my-fix -bc base_config.xml -pc patch_config.xml -l projects-to-test-on.properties
```

The script receives the following set of command line arguments:

**localGitRepo** (r) - path to the local Checkstyle repository (required);

**baseBranch** (b) - name of the base branch in local Checkstyle repository (optional, default is master branch);

**patchBranch** (p) - name of the branch with your changes (required);

**baseConfig** (bc) - path to the base checkstyle configuration file. It will be applied to base branch (required if patchConfig is specified);

**patchConfig** (pc) - path to the patch checkstyle config file. It will be applied to patch branch (required if baseConfig is specified);

**config** (c) - path to the checkstyle config file. It will be applied to base and patch branches (required if baseConfig and patchConfig are not secified);

**listOfProjects** (l) - path to the file which contains the projects which sources will be analyzed by Checkstyle during report generation.

**shortFilePaths** (s) - whether to save report file paths as a shorter version to prevent long paths. This option is useful for Windows users where they are restricted to maximum directory depth (optional, default is false).

When the script finishes its work the following directory structure will be created in the root of cehckstyle-tester directory:

*/repositories* - directory with downloaded projects sources which are specified in projects-to-test-on.properties;

*/reports/diff* - directory with diff reports;

*reports/baseBranch* - directory with Checkstyle reports which are generated with Checkstyle base version (based on specified base branch);

*reports/patchBranch* - directory with Checkstyle reports which are generated with Checkstyle version that contains your changes (based on specified patch branch).

You will find *index.html* file in /reports/diff directory. The file represents summary diff report.

To generate a diff report before and after your changes manually please use the tool
[https://github.com/attatrol/ahsm](https://github.com/attatrol/ahsm)

ATTENTION: 

you can specify at projects-to-test-on.properties path to local file system if you have some 
repositories on your local, example: "checkstyle=/home/username/java/git-repos/checkstyle/checkstyle".

## Deploy Report: 

The created report can be deployed to github pages repo (https://pages.github.com/) to share with others:

1) please follow instruction from https://pages.github.com/ to create your static web site on github;

2) please copy the whole "reports/diff" (if you use diff.groovy script) or "reports" folder (if you use only launch.groovy) to newly created repo;

3) please make sure that report is available as http://YOURUSER.github.io/ ;

4) please make sure that at web site source links to violations are working as it is main part of report, just list of violations is not enough.
