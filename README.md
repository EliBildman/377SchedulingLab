
# 377 Scheduling Lab

## Overview

In this lab, we will look over how Xv6 schedules processes and implement a basic priority queue scheduler. By the end of this lab, you should should
understand how and where Xv6 schedules processes. Be sure to have the gradescope questions open as you go through.

## Get Everything in Place

1. Copy this repo into your elnux machine. 
2. Change into the xv6-public directory. 
3. Run `make` and `make-qemu` to make sure everything is working.
4. Exit with `ctrl-a -> x`

## Look Around

Open the `proc.c` file and find the `scheduler(void)` function. This is the function that runs scheduling for all of Xv6.

```c
void
scheduler(void)
{
  struct proc *p;
  struct cpu *c = mycpu();
  c->proc = 0;
  
  for(;;){
    // Enable interrupts on this processor.
    sti();

    // Loop over process table looking for process to run.
    acquire(&ptable.lock);
    for(p = ptable.proc; p < &ptable.proc[NPROC]; p++){
      if(p->state != RUNNABLE)
        continue;

      // Switch to chosen process.  It is the process's job
      // to release ptable.lock and then reacquire it
      // before jumping back to us.
      c->proc = p;
      switchuvm(p);
      p->state = RUNNING;

      swtch(&(c->scheduler), p->context);
      switchkvm();

      // Process is done running for now.
      // It should have changed its p->state before coming back.
      c->proc = 0;
    }
    release(&ptable.lock);

  }
}
```

Read through the function. Answer questions 1.1 and 1.2 on gradescope.

## Get Familiar With the Commands

Run Xv6 by executing `make qemu-nox` again. This repo has a few additions to help you with this lab. For starters, a user program called `test.c` is added. Take a
take a look a this file, it should be self explanatory. Next, two commands, `ps` and `nice` have been added. Run `ps`. Your output should look like this:

```
$ ps
name     pid     state   priority 
init     1       SLEEPING        2 
 sh      2       SLEEPING        2 
 ps      3       RUNNING         2 
```

You'll notice that processes have a priority field. This will be useful later in creating our priority queue. You can change the priority of a process by using
`nice [pid] [priority]`. Try it out and print the processes again with `ps`. 

The default priority of processes is 10. Processes created in the terminal get a priority of 2, this is why all the processes here are priority 2. Note the priority queue hasn't been implemented yet so the priorities have no effect.

Run test.c with `test &` (the `&` has the process run in the background). Run `ps`. Your output should look like this:

```
$ ps
name     pid     state   priority 
init     1       SLEEPING        2 
 sh      2       SLEEPING        2 
 ps      5       RUNNING         2 
 test    4       RUNNING         2 
```

## Implement the Priority Queue

Replace the `scheduler(void)` function with this starter code:

```c
void scheduler(void)
{
  struct proc *p, *p1;
  struct cpu *c = mycpu();
  c->proc = 0;

  for (;;)
  {
    // Enable interrupts on this processor.
    sti();
    struct proc *highP;
    // Loop over process table looking for process to run.
    acquire(&ptable.lock);
    for (p = ptable.proc; p < &ptable.proc[NPROC]; p++)
    {
      if (p->state != RUNNABLE)
        continue;

      highP = p;

      //TODO: look through process table and choose one with highest priority
    
      //YOUR CODE GOES HERE

      p = highP;
      c->proc = p;
      switchuvm(p);
      p->state = RUNNING;

      swtch(&(c->scheduler), p->context);
      switchkvm();

      // Process is done running for now.
      // It should have changed its p->state before coming back.
      c->proc = 0;
    }
    release(&ptable.lock);
  }
}
```

Under the TODO, implement a loop to look through the `ptable` for the process with the *lowest* priority. `highP` should point to this process when completed. You can access a process' priority with `p->priority` if `p` is a pointer to a process struct.

Upload your `proc.c`  to gradescope question 2 when done.

## Test It Out!

We need to be careful when testing this scheduler. Processes get priority 2 when run from the the terminal, but priority 10 otherwise, meaning anything we run will take precedence over internal processes like taking inputs from the terminal. However, Xv6 runs by default with two processors, giving us some leeway.

Run test.c with `test &` again. Take a look at the process list with `ps`. Adjust the priority of your test process to 15. Run test.x again the same way. Print the process list. It should look like this:

```
$ ps
name     pid     state   priority 
init     1       SLEEPING        2 
 sh      2       SLEEPING        2 
 ps      11      RUNNING         2 
 test    5       RUNNABLE        15 
 test    10      RUNNING         2 
 ```

Notably, your test process with priority 15 should be `RUNNABLE` and your test process with priority 2 should be `RUNNING`. If this is not the case, something went wrong in your implementation.

Answer questions 3.1, 3.2, and 3.3 on gradescope.
