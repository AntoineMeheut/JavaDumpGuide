<!-- PROJECT SHIELDS -->
[![Contributors][contributors-shield]][contributors-url]
[![Forks][forks-shield]][forks-url]
[![Stargazers][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url]
[![GNU License][license-shield]][license-url]

<!-- PROJECT LOGO -->
<br />
<p align="center">
  <a href="https://github.com/AntoineMeheut/JavaDumpGuide">
    <img src="images/logo.png" alt="Logo" width="80" height="80">
  </a>

  <h3 align="center">JavaDumpGuide</h3>

  <p align="center">
    Guide to create a JVM DUMP on a remote server.
    <br />
    <a href="https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/clopts001.html"><strong>Explore Oracle docs</strong></a>
    <br />
    <br />
    <a href="https://github.com/AntoineMeheut/JavaDumpGuide">View Demo</a>
    ·
    <a href="https://github.com/AntoineMeheut/JavaDumpGuide/issues">Report Bug</a>
    ·
    <a href="https://github.com/AntoineMeheut/JavaDumpGuide/issues">Request Feature</a>
  </p>
</p>

<!-- TABLE OF CONTENTS -->
## Table of Contents

* [About the Project](#about-the-project)
  * [Configure the launch of the JVM](#Configure-the-launch-of-the-JVM)
  * [Make a heap dump on OutOfMemoryError or on demand](#Make-a-heap-dump-on-OutOfMemoryError-or-on-demand)
  * [Achieve a heap dump on demand](#Achieve-a-heap-dump-on-demand)
  * [Complete line of start-my-application.sh](#Complete-line-of-start-my-application.sh)
  * [Recover a dump on your server](#Recover-a-dump-on-your-server)
  * [For a on demand heap dump](#For-a-on-demand-heap-dump)
  * [Install the dump analysis tool](#Install-the-dump-analysis-tool)
  * [Analyze the dump](#Analyze-the-dump)
* [Getting Started](#getting-started)
  * [Installation](#installation)
* [Usage](#usage)
* [Roadmap](#roadmap)
* [Contributing](#contributing)
* [License](#license)
* [Contact](#contact)
* [Acknowledgements](#acknowledgements)

<!-- ABOUT THE PROJECT -->
## About The Project

### Configure the launch of the JVM
You have to open the launch script of your Java application start-my-application.sh, we will need to modify the options passed at the start of the JVM, so that it knows that you want to dump.

You will have to initialize a variable to the home associated with your user account. We will use this path for the dump to arrive directly in your home, making it easier to copy the dump file to your computer. If we do not do this, the dump will be done in the directory where the start-my-application.sh script is located, which sometimes can cause problems, because for security reasons, it is not desirable that this directory be open in writing.

We will therefore add to the script start-my-application.sh a DUMPDIR

```
Initialization of script variables
LOGDIR=/<my_server>/<my_log_repertory>/log
DUMPDIR=/home/<my_dump_repertory>
```

Then, you'll have to add some parameters to the launch line of your application's JVM.

This line before modification is like this in script start-my-application.sh

```
java -jar -Dspring.config.location=$install_dir/conf/<my_application_repertory>/application.properties -Dlogback.configurationFile=$install_dir/conf/<my_application_repertory>/logback.xml -Dconfig.file.path=$install_dir/conf/<my_application_repertory>/business/businessConfiguration.xml $install_dir/jars/<my_application_name.jar> &
```

We will now add a number of new options that I will detail.

### Make a heap dump on OutOfMemoryError or on demand
Without going into too much detail: when a JVM has no longer free memory, it will use the swap and if unfortunately all the swap is used, it will finish the code it runs on an OutOfMemoryError.

To ask the JVM to produce a heap dump on the OutOfMemoryError event we will add the following option

```
-XX:+HeapDumpOnOutOfMemoryError
```

### Achieve a heap dump on demand
To perform a dump on demand, we will use HPROF which is a native JVM used to make memory and cpu dump.

```
-agentlib:hprof=heap=dump,format=b,file=$DUMPDIR/<my_application_name.hprof>
```

### Complete line of start-my-application.sh

```
java -jar -XX:+HeapDumpOnOutOfMemoryError -agentlib:hprof=heap=dump,format=b,file=$DUMPDIR/<my_application_name.hprof> -Dspring.config.location=$install_dir/conf/<my_application_repertory>/application.properties -Dlogback.configurationFile=$install_dir/conf/<my_application_repertory>/logback.xml -Dconfig.file.path=$install_dir/conf/<my_application_repertory>/business/businessConfiguration.xml $install_dir/jars/<my_application_name.jar> &
```

### Recover a dump on your server
If it is a dump on OutOfMemory you have nothing to do, the JVM will make it when the OutOfMemory message that will occur.

### For a on demand heap dump
There are two ways to do it.

* First method:

Use the jps command with the -l option, this will allow you to see all active java processes with their PID and their name

```
> jps -l
```

Use a kill with the -3 option on the PID of the JVM you are interested in, it is this command that will trigger the heap dump, provided that the JVM has been warned before, otherwise it insults you in English. From the point of view of the FMV it has not been warned, we must put ourselves in its place.

```
> kill - 3 <pid>
```

* Second method:

Use the stop script of the frequenceur and that realize the heap dump

```
> ./<stop-my-application.sh>
```

### Install the dump analysis tool
The tool is at this address [Memory Analyzer (MAT)](https://www.eclipse.org/mat/)

### Analyze the dump
If everything went well, you must have a <my_application_name.hprof> file in your home / user directory

You need to recover this file on your disk and open it with the Memory Analizer tool you just installed.

For more information, here is the url which served me to understand how to analyze the heap dump [10-tips-for-using-the-eclipse-memory-analyzer](https://eclipsesource.com/blogs/2013/01/21/10-tips-for-using-the-eclipse-memory-analyzer/)

<!-- GETTING STARTED -->
## Getting Started

To get a local copy up and running follow these simple steps.

### Installation
 
1. Clone the repo
```
git clone https://github.com/AntoineMeheut/JavaDumpGuide
```

<!-- USAGE EXAMPLES -->
## Usage

Clone this project to use it for your developments or simply copy/paste the java script for your start-my-application.sh

<!-- ROADMAP -->
## Roadmap

See the [Project](https://github.com/AntoineMeheut/JavaDumpGuide/projects) for a list of proposed features (and known issues).

<!-- CONTRIBUTING -->
## Contributing

Contributions are what make the open source community such an amazing place to be learn, inspire, and create. Any contributions you make are **greatly appreciated**.

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

<!-- LICENSE -->
## License

Distributed under the GNU General Public License v3.0 License. See `LICENSE` for more information.

<!-- CONTACT -->
## Contact

If you want to contact me [just clic](mailto:github.contacts@protonmail.com)

Project Link: [https://github.com/AntoineMeheut/JavaDumpGuide](https://github.com/AntoineMeheut/JavaDumpGuide)

<!-- ACKNOWLEDGEMENTS -->
## Acknowledgements

This package was created by reading 10 Tips for using the Eclipse Memory Analyzer from Ian Bull.

* [Ian Bull](https://eclipsesource.com/blogs/author/ianbull/)
* [10 Tips for using the Eclipse Memory Analyzer](https://eclipsesource.com/blogs/2013/01/21/10-tips-for-using-the-eclipse-memory-analyzer/)

<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->
[contributors-shield]: https://img.shields.io/github/contributors/AntoineMeheut/JavaDumpGuide?color=green
[contributors-url]: https://github.com/AntoineMeheut/JavaDumpGuide/graphs/contributors
[forks-shield]: https://img.shields.io/github/forks/AntoineMeheut/JavaDumpGuide
[forks-url]: https://github.com/AntoineMeheut/JavaDumpGuide/network/members
[stars-shield]: https://img.shields.io/github/stars/AntoineMeheut/JavaDumpGuide
[stars-url]: https://github.com/AntoineMeheut/JavaDumpGuide/stargazers
[issues-shield]: https://img.shields.io/github/issues/AntoineMeheut/JavaDumpGuide
[issues-url]: https://github.com/AntoineMeheut/JavaDumpGuide/issues
[license-shield]: https://img.shields.io/github/license/AntoineMeheut/JavaDumpGuide
