1. [JLS §17.4.1. Shared Variables](https://docs.oracle.com/javase/specs/jls/se17/html/jls-17.html#jls-17.4.1):  
    
    > Memory that can be shared between threads is called _shared memory_ or _heap memory_.  
    >   
    > All instance fields, `static` fields, and array elements are stored in heap memory. In this chapter, we use the term _variable_ to refer to both fields and array elements.  
    >   
    > Two accesses to (reads of or writes to) the same variable are said to be _conflicting_ if at least one of the accesses is a write.  
    
2. [JLS §17.4.5. Happens-before Order](https://docs.oracle.com/javase/specs/jls/se17/html/jls-17.html#jls-17.4.5):  
    
    > When a program contains two _conflicting accesses_ (§17.4.1) that are not ordered by a _happens-before relationship_, it is said to contain a **data race**.
    
    