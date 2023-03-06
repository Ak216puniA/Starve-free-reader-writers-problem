# Starve-free-reader-writers-problem

The readers-writers problem is a classical problem of process synchronization, it relates to a data set such as a file that is shared between more than one process at a time. Among these various processes, some are Readers - which can only read the data set; they do not perform any updates, some are Writers - can both read and write in the data sets. Solution to this problem can be reader-biased or writer-biased but the problem with the solutions is that one of the side i.e. either reader processes or writer processes may starve during execution.

I will be initially explaining the classic reader-biased solution (where writers starve) to the problem, followed by how the solution can be made starve-free for both the readers as well as the writers.

## Reader biased solution
In the following solution, more preference is given to readers of the problem. If reader processes keep on entering the queue, writers might starve hence making this solution reader biased.

### Semaphores
```
// Semaphores

semaphore reader_mutex = 1 
//Ensures mutual exclusion for the critical sections consisiting shared variable readcount

semaphore write = 1 
//Prevents writer from accessing the resource while some readers are reading

// Shared variable

int readcount = 0 
//Counts the number of readers currently accessing the resource
```
### Reader
```
while(true){

    wait(reader_mutex) 
    //Ensures the no reader other than the one currently inside, accesses the critical section 

    readcount++
    if(readcount==1){ 
        wait(write)
    } 
    //If readcount==1 i.e. the process is the first reader, ensure that the writer isn't writing anything in the resource

    signal(reader_mutex)
    //Signals that the critical section can be accessed by another reader

    //Reading action performed

    wait(reader_mutex)
    //Ensures mutual exclusion for critical section

    readcount--
    if(readcount==0){
        signal(write)
    }
    //If no reader is reading, signal writer to start writing

    signal(reader_mutex)
    //Signals critical section ready to be used

}
```
### Writer
```
while(true){

    wait(write)
    //Wait for write signal

    //Writing action performed

    signal(write)
    //Signal completion of writing process

}
```
As observed in the pseudocode above, writer is allowed to access the resaource only when readcount=0 i.e. no reader is reading. Now consider a situation when there are some reader processes in the queue and a writer enters. In such case, writer need to wait till all the readers complete their reading and even then if some new readers enter the queue, the wait will only extend. This leads to a situation termed as starvation when a low priority process (writer) gets jammed for a long time due to some high priority processes (readers). Hence, maing this solution reader biased.

## Starve free solution
The major problem faced in the reader biased solution was the higher priority of the readers over writers as set by the code. Writers were allowed access to resource if and only if readcount reaches zero i.e. as long as there were any reader processes left the queue writer cannot be executed. Based over these observations, our main objective in the starve free solution will be to ensure equal opportunities for the both sides which further can be fulfilled if we maintain the order in which the processes are arriving i.e. ensure FIFO queue for processes. So, in the starve free solution we will adding another semophore namely, 'equal' to ensure fairness among reader and writer processes. 

### Semaphores
```
// Semaphores

semaphore reader_mutex = 1 
//Ensures mutual exclusion for the critical sections consisiting shared variable readcount

semaphore write = 1 
//Prevents writer from accessing the resource while some readers are reading

semaphore equal = 1
//Ensures fairness among readers and writers and prevents starvation

// Shared variable

int readcount = 0
//Counts the number of readers currently accessing the resource
```

### Reader
```
while(true){

    wait(equal)
    //Wait for the process in the queue that is currently running (the first process of the queue)
    //Ensures that once a process encompassed wait(equal) no other process be able to enter write or read entry section till signal(equal) has been called

    wait(reader_mutex) 
    //Ensures the no reader other than the one currently inside, accesses the critical section 

    readcount++
    if(readcount==1){ 
        wait(write)
    } 
    //If readcount==1 i.e. the process is the first reader, ensure that the writer isn't writing anything in the resource

    signal(reader_mutex)
    //Signals that the critical section can be accessed by another reader

    signal(equal)
    //Signals that the next process in queue can run

    //Reading action performed

    wait(reader_mutex)
    //Ensures mutual exclusion for critical section

    readcount--
    if(readcount==0){
        signal(write)
    }
    //If no reader is reading, signal writer to start writing

    signal(reader_mutex)
    //Signals critical section ready to be used


}
```

### Writer
```
while(true){

    wait(equal)
    //Wait for the process in the queue that is currently running (the first process of the queue)
    //Ensures that once a process encompassed wait(equal) no other process be able to enter write or read entry section till signal(equal) has been called

    wait(write)
    //Wait for write signal

    signal(equal)
    //Signals that the next process in queue can run

    //Writing action performed

    signal(write)
    //Signal completion of writing process

}
```
As you can now observe we encapsulated the entry sections for both reader and writer with wait(equal) and signal(equal). So, now if we consider the same situation i.e. there are some readers followed by a writer in the queue, writer will not be pushed backwards in the queue if any new reader arrives. Assuming that a reader process is in its entry section, equal will reamin 0 preventing execution of the next process. Now, as soon as the reader signals equal, signal semaphore will become 1 and the next process waiting for it in the queue will be able to access it's entry section no matter whether it's a reader process or a writer process (both writer and reader codes are waiting on the equal semaphore so the which one will get access to equal first depends on there arrival time). The remaining flow of the code remains the same for both kind of processes.