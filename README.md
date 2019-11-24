JavaDumpGuide
=============
 Guide to creating a JVM DUMP on a remote server

Objectives of this guide
------------------------

1. Configure the launch of its JVM
2. Make a heap dump on OutOfMemoryError or on demand
3. Install the dump analysis tool
4. Analyze the dump
5. Analyze memory and code performance in real time

Configure the launch of its JVM
-------------------------------
You have to open the launch script of your Java application <start_my_application.sh>, because we will need to modify the options passed at the start of the JVM, so that it knows that you want to dump.

You will have to initialize a variable to the home associated with your user account. We will use this path for the dump to arrive directly in your home, making it easier to copy the dump file to your computer. If we do not do this, the dump will be done in the directory where the <start_my_application.sh> script is located, which sometimes can cause problems, because for security reasons, it is not desirable that this directory be open in writing.

We will therefore add to the script <start_my_application.sh> a DUMPDIR
```bash
Initialization of script variables
LOGDIR=/<my_server>/<my_log_repertory>/log
DUMPDIR=/home/<my_dump_repertory>
```

Then, you'll have to add some parameters to the launch line of your application's JVM.

This line before modification is like this in script <start_my_application.sh>
```bash
java -jar -Dspring.config.location=$install_dir/conf/<my_application_repertory>/application.properties -Dlogback.configurationFile=$install_dir/conf/<my_application_repertory>/logback.xml -Dconfig.file.path=$install_dir/conf/<my_application_repertory>/business/businessConfiguration.xml $install_dir/jars/<my_application_name.jar> &
```

We will now add a number of new options that I will detail.

Make a heap dump on OutOfMemoryError or on demand
-------------------------------------------------
Sans rentrer trop dans les détails : quand une JVM ne dispoe plus de mémoire free, it will use the swap and if unfortunately all the swap is used, it will finish the code it runs on an OutOfMemoryError.

To ask the JVM to produce a heap dump on the OutOfMemoryError event we will add the following option
```bash
-XX:+HeapDumpOnOutOfMemoryError
```


Achieve a heap dump on demand
-----------------------------
To perform a dump on demand, we will use HPROF which is a native JVM used to make memory and cpu dump.

```bash
-agentlib:hprof=heap=dump,format=b,file=$DUMPDIR/<my_application_name.hprof>
```

Complete line of <start_my_application.sh>
------------------------------------------

```bash
java -jar -XX:+HeapDumpOnOutOfMemoryError -agentlib:hprof=heap=dump,format=b,file=$DUMPDIR/<my_application_name.hprof> -Dspring.config.location=$install_dir/conf/<my_application_repertory>/application.properties -Dlogback.configurationFile=$install_dir/conf/<my_application_repertory>/logback.xml -Dconfig.file.path=$install_dir/conf/<my_application_repertory>/business/businessConfiguration.xml $install_dir/jars/<my_application_name.jar> &
```

Recover a dump on your server
-----------------------------
If it is a dump on OutOfMemory you have nothing to do, the JVM will make it when the OutOfMemory message that will occur.

For a on demand heap dump 
-------------------------
There are two ways to do it.

1- First method
Use the jps command with the -l option, this will allow you to see all active java processes with their PID and their name

```bash
> jps -l
```

Use a kill with the -3 option on the PID of the JVM you are interested in, it is this command that will trigger the heap dump, provided that the JVM has been warned before, otherwise it insults you in English. From the point of view of the FMV it has not been warned, we must put ourselves in its place.
```bash
> kill - 3 <pid>
```

2- Second method
Use the stop script of the frequenceur and that realize the heap dump

```bash
> ./<stop_my_application.sh>
```

Install the dump analysis tool
------------------------------
The tool is at this address https://www.eclipse.org/mat/

Analyze the dump
----------------
If everything went well, you must have a <my_application_name.hprof> file in your home / user directory

You need to recover this file on your disk and open it with the Memory Analizer tool you just installed.

For more information, here is the url which served me to understand how to analyze the heap dump
[https://eclipsesource.com/blogs/2013/01/21/10-tips-for-using-the-eclipse-memory-analyzer/](10-tips-for-using-the-eclipse-memory-analyzer)
