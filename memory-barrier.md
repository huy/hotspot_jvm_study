## Memory barrier

**what is it ?**

Modern multi cores CPU has different level of cache as such write to memory location by one core is not immediately visible 
to an other. We can imagine that each core has both read buffer and write buffer to read from and write to (note that processor documents may refer to read and write cache as load and store cache). These buffers need to be flushed/drained in order to make the change the an variable by one to be seen by an other.

Memory barrier is CPU instructions that flush the read/write buffer cache. On Intel CPU (note that other CPU may work quite differently), there are two basic kinds of barriers 

* write barrier
* read barrier  

The write barrier running on a core flushes write cache of the core. The read barrier conversely drains the core read cache. 
Suppose that we modify an variable in one core and want that update to be seen by an other core then on the first core we 
need to execute write barrier instruction after the modification (to flush the core write buffer) and also issue the read 
barrier instruction before reading the variable on the other core (to drain the other core read buffer). 

    Core 1
    ------
    a = 9
    write_barrier

    Core 2
    ------
    while (true) {
       load_barrier
       if (a == 9)
        break;
    }
       
Memory barrier also mean that CPU is not allowed to reorder instruction freely across a barrier. More precisely an read memory instruction appears after a `read_barrier` can't be moved before the barrier and conversely write memory instruction appears before a `write_barrier` can't be moved after the barrier. Otherwise the reorder will defy the purpose of memory barrier.

**How a memory barrier is used Java ?**

On Intel CPU, if a variable is declared as `volatile`, whenever we modify a volatile variable then java runtime inserts a `write_barrier` instruction after the modification instruction, conversely when we read a volatile variable it inserts `read_barrier` before the instruction that reads the variable.

When a variable is declared as `final`, java runtime insert a `write_barrier` imediately after inilization of the variable by doing that any instructions reading the variable get correct uptodate value.

An other common case is `synchronize` a block of code. In that case java runtime insert `read_barrier` immediately before entering the code block and `write_barrier` immediately after exiting it.

Similarly with respect to the lock, java runtime insert `read_barrier` after the lock is successfully aquired and `write_barrier` immediately after its release.

**References**

* http://mechanical-sympathy.blogspot.com.au/2011/07/memory-barriersfences.html
* http://www.infoq.com/articles/memory_barriers_jvm_concurrency
