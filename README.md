# Starve-free-reader-writers-problem

The readers-writers problem is a classical problem of process synchronization, it relates to a data set such as a file that is shared between more than one process at a time. Among these various processes, some are Readers - which can only read the data set; they do not perform any updates, some are Writers - can both read and write in the data sets. Solution to this problem can be reader-biased or writer-biased but the problem with the solutions is that one of the side i.e. either reader processes or writer processes may starve during execution.

I will be initially explaining the classic reader-biased solution (where writers starve) to the problem, followed by how the solution can be made starve-free for both the readers as well as the writers.

## Reader biased solution

### Semaphores
```
// Semaphores

semaphore reader_mutex = 1 //
semaphore write = 1 

// Shared variable

int readcount = 0
```
### Reader
```
while(true){

    wait(reader_mutex)
    readcount++
    if(readcount==1){
        wait(write)
    }
    signal(reader_mutex)

    //Reading action performed

    wait(reader_mutex)
    readcount--
    if(readcount==0){
        signal(write)
    }
    signal(reader_mutex)

}
```
### Writer
```
while(true){

    wait(equal)
    wait(write)
    signal(equal)

    //Writing action performed

    signal(write)

}
```

## Starve free solution

### Semaphores
```
// Semaphores

semaphore reader_mutex = 1 //
semaphore write = 1 
semaphore equal = 1

// Shared variable

int readcount = 0
```

### Reader
```
while(true){

    wait(equal)
    wait(reader_mutex)
    readcount++
    if(readcount==1){
        wait(write)
    }
    signal(reader_mutex)
    signal(equal)

    //Reading action performed

    wait(reader_mutex)
    readcount--
    if(readcount==0){
        signal(write)
    }
    signal(reader_mutex)

}
```

### Writer
```
while(true){

    wait(equal)
    wait(write)
    signal(equal)

    //Writing action performed

    signal(write)

}
```