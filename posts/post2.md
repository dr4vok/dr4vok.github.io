#### Hey guys, so today lets talk about Processes in detail

**Note that in this article i will ONLY talk about processes as** 
- **Concept**
- **Properties and explaining them in detail**

##### **Lets start with what is a process? probably the best first question we can ask.** 

So a **process** (Or *Task*) is a program in a midst of execution , but its not the program itself like ".exe" files , a process is an **active** program and related resources , 2 or more processes can exist that are sharing the same program and they also can share resources like (open files or address space)

For example, if you ran 
```bash
ls
```
Linux starts a process to execute `ls` displays the output and then ends the process. notice that the process is not the actual C file, but the active program being executed 

The process also include a set of resources such as 
- open files
- pending signals
- internal kernel data
- processor state
- a memory address space with one or more memory mappings
- one or more threads of execution
- a **data section containing global variables**)

**Lets start clearing what every resource mean in plain english**

##### **What is open file?**
Its a file that is currently associated with a file descriptor.

##### **Okay okay what is file descriptor?**
File descriptor is per-process unique, non-negative integer with negative values being reserved to indicate "no value" or error conditions.
Its used to identify open file for the purpose of file access. 

File Descriptors are part of the POSIX API, Each process should have three standard POSIX file descriptors corresponding to the three standard streams

_Note: POSIX or Portable Operating System Interface are just some standards, check https://en.wikipedia.org/wiki/POSIX for more info_ 

0 -> stdin (standard input) \
1 -> stdout (standard output) \
2 -> stderr (standard error)

Frankly talking, the daemons are rare exception where in traditional SysV daemon it close all open FD except stdin , stdout and error (this ensures that no accidentally passed file descriptor stays around in the daemon process). later in step 9 it connects  /dev/null to stdin and stdout and error



##### **Back to file descriptors.** 
In the traditional implementation of Unix, file descriptor use numbers to look up information on a list that belongs to only one running process, it uses file descriptor table for that, its maintained by the kernel. the kernel in turn looks into a system wide table of files opened by all processes called the file table. This table records the _mode_ with which the file (or other resource) has been opened such as for reading, writing or appending , or other mods like (access mode). it also points into a third table (yea allot of tables here) called inode table. an inode is basically set of data that describes a particular file.
To perform input or output the process passes the file descriptor to the kernel through a system call and the kernel will access the file on behalf of the process. the process itself doesnt have direct access to the file or inode


##### **NOW LETS GET OUR HANDS DIRTY**
there are sets of file descriptors open in a process U can access under the path of
`/proc/PID/fd`, the PID is the process identifier
 
 File descriptor are in the follwoing paths
 `/proc/PID/fd/0` is `stdin`
`/proc/PID/fd/1` is `stdout`
`/proc/PID/fd/2` is `stderr`


U can see the open files on ur linux machine by using `lsof` like this
```shell
# lsof
COMMAND PID USER FD TYPE DEVICE SIZE/OFF NODE NAME
systemd 1 root cwd DIR 8,1 4096 2 /
systemd 1 root rtd DIR 8,1 4096 2 /
systemd 1 root txt REG 8,1 1595792 9961784 /lib/systemd/systemd
systemd 1 root mem REG 8,1 1700792 9961570 /lib/x86_64-linux-gnu/libm-2.27.so
systemd 1 root mem REG 8,1 121016 9961695 /lib/x86_64-linux-gnu/libudev.so.1
--snip--
vi 1994 juser cwd DIR 8,1 4096 4587522 /home/juser
vi 1994 juser 3u REG 8,1 12288 786440 /tmp/.ff.swp
--snip--
```
The output lists the following fields in the top row:
```shell
COMMAND: The command name for the process that holds the file
descriptor.
PID: The process ID.
USER: The user running the process.
FD: This field can contain two kinds of elements. In most of the preceding output, the FD column shows the purpose of the file. The FD field
can also list the file descriptor of the open file—a number that a pro-
cess uses together with the system libraries and kernel to identify and
manipulate a file; the last line shows a file descriptor of 3.
TYPE: The file type (regular file, directory, socket, and so on).
DEVICE: The major and minor number of the device that holds the file.
SIZE/OFF: The file’s size.
NODE: The file’s inode number.
NAME: The filename.
```

_check the manual page of lsof to get most of it, this is just a demonstration to see ur open files_

_Note: i got this part from book how linux works. i got into problem that my terminal doesnt show the first column because of the scroll limit.do i want to edit the settings of my terminal? absolute not, im too lazy for this, so i just did this `sudo lsof | less` and voala u can now move with ur arrow key up and down and search by typing /. dm my insta if u want me to make `less` command tutorial_


##### **Now lets get into "pending signals" part**

What is signals? (i will use the description from the Advanced programming in the UNIX env book because this is the open tab right now)
So signals are software used to interrupt. It also provides a way of handling events such as terminating , alarming and such more

**Okay its been allot and you will probably got overwhelmed because all of this information so i will speed run the rest;**

**processor state** -> The state field of the process descriptor describes the current condition of the process
Each process on the system is in exactly one of **5 different states** 
- `TASK_RUNNING` : The process is **runnable**; either **running** or a **run-queue** waiting to run. 
	- Only possible in user-space and can apply to a process in kernel-space that is running
- `TASK_INTERRUPTIBLE` : The process is sleeping/Blocked waiting for some condition to exist.
	- The kernel sets the process's state to `TASK_RUNNING` when the condition exist.
		  The process also awakes prematurely and becomes runnable if it receives a signal.
- `TASK_UNINTERRUPTIBLE` : Identical to `TASK_INTERRUPTIBLE` except that it **doesnt wake up** and become runnable if receives a signal.
		- Its being used in situations where the process must wait without interruption or when the event is expected to occur quickly.
		  Because the task does not respond to signals in this state, `TASK_UNINTERRUPTIBLE` is less often used than `TASK_INTERRUPTIBLE`.
		  This is why you have those unkillable process with state `D` in `ps` , Bec, the task will not respond to signals and cannot send it a `SIGKILL` , if you somehow managed to terminate it , it wouldnt be wise because the task is supposedly in the middle of an important operation.
- `__TASK_TRACED` :The process is being `traced` by another process such as a debugger via `ptrace`
-  `__TASK_STOPPED` : Process execution has stopped ; the task is not running nor its eligible to run , this occurs if the  task receives the `SIGSTOP`, `SIGTSTP`, `SIGTTIN`, or `SIGTTOU` signal or if it receives any signal while it is being debugged. 


**One or more threads of execution** -> threads of executio (shortened to threads)  are the objects of activity within the process


**THAT WAS ALLOT, THX FOR STICKING AROUND!!**

**Resources**:
https://www.geeksforgeeks.org/linux-unix/processes-in-linuxunix/ \
Linux kernel development book by Robert's love \
How linux works what every super user should know 3rd edition \
Advanced Programming in the UNIX Environment by W. Richard Stevens \
https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap03.html \
https://en.wikipedia.org/wiki/File_descriptor \
https://man7.org/linux/man-pages/man7/daemon.7.html
