/* schedule.c
 * This file contains the primary logic for the 
 * scheduler.
 */
#include "schedule.h"
#include "macros.h"
#include <stddef.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

#define NEWTASKSLICE (NS_TO_JIFFIES(100000000))

/* Local Globals
 * rq - This is a pointer to the runqueue that the scheduler uses.
 * current - A pointer to the current running task.
 */
struct runqueue *rq;
struct task_struct *current;

/* External Globals
 * jiffies - A discrete unit of time used for scheduling.
 *       There are HZ jiffies in a second, (HZ is 
 *       declared in macros.h), and is usually
 *       1 or 10 milliseconds.
 */
extern long long jiffies;


/*-----------------Initilization/Shutdown Code-------------------*/
/* This code is not used by the scheduler, but by the virtual machine
 * to setup and destroy the scheduler cleanly.
 */
 
/* initscheduler
* Sets up and allocates memory for the scheduler, as well
* as sets initial values. This function should also
* set the initial effective priority for the "seed" task 
* and enqueu it in the scheduler.
* INPUT:
* newrq - A pointer to an allocated rq to assign to your
*     local rq.
* seedTask - A pointer to a task to seed the scheduler and start
* the simulation.
*/
void initschedule(struct runqueue *newrq, struct task_struct *seedTask){
    srand(time(NULL));
    seedTask->first_time_slice=rand()%NEWTASKSLICE+10;
    seedTask->time_slice=seedTask->first_time_slice;
    rq->active = (struct sched_array *)malloc(sizeof(struct sched_array));
    rq->expired = (struct sched_array *)malloc(sizeof(struct sched_array));
    INIT_LIST_HEAD(&rq->active->list);
    INIT_LIST_HEAD(&rq->expired->list);
    enqueue_task(seedTask,rq->active);
    rq->nr_running++;
}

/* killschedule
 * This function should free any memory that 
 * was allocated when setting up the runqueu.
 * It SHOULD NOT free the runqueue itself.
 */
void killschedule(){
}

/*-------------Scheduler Code Goes Below------------*/
/* This is the beginning of the actual scheduling logic */

/* schedule
 * Gets the next task with the shortest runtime(time slice) remaining
 */
void schedule(){
    if (rq->nr_running){
        struct sched_array *shortest;
        shortest = list_entry(rq->active->list.next,struct sched_array,list);
        if (rq->curr!=NULL){
            if(!rq->curr->time_slice)
                rq->curr->time_slice=rq->curr->first_time_slice;
            list_move_tail(&(rq ->curr->array->list),&(rq->active->list)); 
            struct sched_array *pos;
            list_for_each_entry(pos,&(rq->active->list),list){
                if (shortest->task->time_slice>pos->task->time_slice)
                    shortest=pos;
            }
            rq->curr->need_reschedule=0;
        }
        if(rq->curr!=shortest->task){
            context_switch(shortest->task);
            rq->curr=shortest->task;
        }
    }
}


/* enqueue_task
 * Enqeueus a task in the passed sched_array
 */
void enqueue_task(struct task_struct *p, struct sched_array *array){
    struct sched_array *node = (struct sched_array*)malloc(sizeof(struct sched_array));
    node->task = p;
    p->array = node;
    list_add_tail(&node->list,&array->list);
}

/* dequeue_task
 * Removes a task from the passed sched_array
 */
void dequeue_task(struct task_struct *p, struct sched_array *array){ 
    struct sched_array *node = (struct sched_array*)malloc(sizeof(struct sched_array));
    rq->curr=p;
    if(rq->curr!=NULL)
        rq->curr=NULL;
    list_del(&p->array->list);
    node->task = p;
    p->array = node;
    list_add(&node->list,&rq->expired->list);
    if(rq->best_expired_prio > p->prio)
        rq->best_expired_prio = p->prio;
}

/* sched_fork
 * Sets up schedule info for a newly forked task
 */
void sched_fork(struct task_struct *p){
    p->first_time_slice=rand()%NEWTASKSLICE+10;
    p->time_slice=p->first_time_slice;
}

/* scheduler_tick
 * Updates information and priority
 * for the task that is currently running.
 */
void scheduler_tick(struct task_struct *p){
    p->time_slice--;
    if(p->time_slice <= 0)
        p->need_reschedule = 1;
}

/* wake_up_new_task
 * Prepares information for a task
 * that is waking up for the first time
 * (being created).
 */
void wake_up_new_task(struct task_struct *p){
    enqueue_task(p,rq->active);
    rq->nr_running++;
}

/* __activate_task
 * Activates the task in the scheduler
 * by adding it to the active array.
 */
void __activate_task(struct task_struct *p){
    enqueue_task(p,rq->active);
    rq->nr_running++;
}

/* activate_task
 * Activates a task that is being woken-up
 * from sleeping.
 */
void activate_task(struct task_struct *p){
    enqueue_task(p,rq->active);
    rq->nr_running++;
}

/* deactivate_task
 * Removes a running task from the scheduler to
 * put it to sleep.
 */
void deactivate_task(struct task_struct *p){
    dequeue_task(p,NULL);
    rq->nr_running--;
}
