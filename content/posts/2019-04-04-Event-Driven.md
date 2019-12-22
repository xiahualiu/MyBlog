---
title: "From SICP - Understand Event Driven Programming"
date: 2019-04-04
draft: false
---

# Introduction

I am recently working on the wizard book SICP (The Structure and Interpretation of Computer Programs) that was recommended by one of my friends who study computer science. To be honest, the book is not so simple and naive as it was firstly acknowledged as the introduction book of MIT CS major. It abstracts and reveals essential programming philosophy in a easiest way, by the vehicle that known as the programming language Scheme. At first I thought scheme was a painful language for normal programmers because I only used C/C++  language that is of imperative style. However after reading the book and finishing every exercise all the way down, I gradually learned how to maneuver it and really got it on the right way.

<!--more-->

In this post I used scheme as the language to illustrate what is event-driven programming, I once worked on a project that required to implement a simple protocal stack on a chip. However at that time I did not know about the event-driven paradigm so I make the processor to process the message packaging and unpackaging in a queue. In fact it worked pretty well (queue is also used in event-driven programming), however because lack of engineering experience, the whole program turned into a really intricate complex, with so many useless parts scattered everywhere. The whole project is difficult to maintain and debug.

In Chapter 3, SICP gives a special example in scheme, showing students how to use scheme to implement a small circuit simulator just like verilog. However it did not go into details, just provides a rough sketch. And the codes are in a way that easy for adept programmer to read, not for new programmers like me.

So I reiterate the code here, and describe the code segments in a human sequence (compared with SICP's only-god-know sequence).

# Full Overview

First of all, we only implement three basical gates (and or not) as primitives. We can imagine the circuit like the one below: In fact all the digital system can unfold into this three gates. So only 3 primitives is enough. 

![Fig 1. A Half Adder Circuit][1] 

We must know how to we make our program behave like the real circuit. The program must transmit the signal from a node to another node, and changed them according to the gates' different input output rules. What's more, we must know the delay of each gate and make every event happend in the time sequence. We will first go through main event-driven part and ignore the time line.

The signals carried on circuit wires vary over time, when a signal changes, we know it causes a sequence of follow wires jumpping from 0 to 1 or 1 to 0 due to the logic gates rules. So we must invent a program mechanics to let a logic gate node able to catch a change on their input parameters and change their output, but how?

The answer is add an extra intermedia to hold a state, and let it trigger the event. In this article, the **wire** object takes this duty, we define wire as below:

```scheme
(define (get-signal wire) (wire 'get-signal))
(define (set-signal! wire new-value)
  ((wire 'set-signal!) new-value))
(define (add-action! wire action-procedure)
  ((wire 'add-action!) action-procedure))

(define (make-wire)
  (let ((signal-value 0) (action-procedures '()))
    (define (set-my-signal! new-value)
      (if (not (= signal-value new-value))
          (begin (set! signal-value new-value)
                 (call-each action-procedures))
          'not-changed))
    (define (accept-action-procedure! proc)
      (set! action-procedures
            (cons proc action-procedures))
      (proc))
    (define (dispatch m)
      (cond ((eq? m 'get-signal) signal-value)
            ((eq? m 'set-signal!) set-my-signal!)
            ((eq? m 'add-action!) accept-action-procedure!)
            (else (error "Unknown operation on wires"))))
    dispatch))
```

We can see that the wire holds a local variable `signal-value` and a procedure list `action-procedures`. Wire has three interfaces to interact with other part of the program. The interesting part is the `set-my-signal!` call, which does not simply change the signal value, but also triggeres the procedures in the list.

So think when the state of a wire is changed, it will call the procedure (these procedures are stored internal but defined as global) to change the state of adjacent wires, by calling their `set-signal!` calls, and `set-signal!` is defined in a particular way that tiggeres internal-stored procedures which affect the next wires and there becomes a chain reaction till processor reaches the output wires where no procedures are stored, all the wires are refreshed.

What are the procedures in the list? They are called when the value changes, in this case, they are logic gates. We must define these procedures in a way the connected wires are bound together, because we cannot bind them inside the wire object due to local environment rules. We want to change the connected wires states when called, it is important to construct the procedure environment enclosed in global environment and only in this way the wires state can be changed when internal precedures are called.

The `call-each` calls these procedures, which are defined as global procedures (very important) and have no arguments.

```scheme
(define (call-each procedures)
  (if (null? procedures)
      'done
      (begin ((car procedures))
             (call-each (cdr procedures)))))
```

# Logic Gates
The three logic gates are defined below:

```scheme
(define (inverter input output)
  (define (invert-input)
    (let ((new-value (logical-not (get-signal input))))
        (set-signal! output new-value)))
  (add-action! input invert-input) 'ok)

(define (logical-not s)
  (cond ((= s 0) 1)
        ((= s 1) 0)
        (else (error "Invalid signal" s))))

(define (and-gate input1 input2 output)
  (define (and-input)
    (let ((new-value (logical-and (get-signal input1) (get-signal input2))))
      (set-signal! output new-value)))
  (add-action! input1 and-input)
  (add-action! input2 and-input) 'ok)

(define (logical-and s1 s2)
  (cond ((= (* s1 s2) 1) 1)
        ((or (= (+ s1 s2) 0) (= (+ s1 s2) 1)) 0)
        (else (error "Invalid signal" s1 s2))))

(define (or-gate input1 input2 output)
  (define (or-input)
    (let ((new-value (logical-or (get-signal input1) (get-signal input2))))
      (set-signal! output new-value)))
  (add-action! input1 or-input)
  (add-action! input2 or-input) 'ok)

(define (logical-or s1 s2)
  (cond ((and (= s1 1) (= s2 0)) 1)
        ((and (= s2 1) (= s1 0)) 1)
        ((and (= s1 1) (= s2 1)) 1)
        ((and (= s1 0) (= s2 0)) 0)
        (else (error "Invalid Signal" s1 s2))))
```

These gates themselves are like injectors, they create the procedures and injected them into the input wires.

# Probes

We just finished the gates and the wires, and the simulation is ready to go, but we need a probe on the wires to tell us the change happended to what wire and from what state to another state.

```scheme
(define (probe name wire)
  (add-action! wire
               (lambda ()
                 (display name) (display " ")
                 (display " New-value = ")
                 (display (get-signal wire))
                 (newline))))
```

# Go!

Using the gate primitives defined above, we can implement the `half-adder` in picture above:

```scheme
(define (half-adder A B S C)
  (define D (make-wire))
  (define E (make-wire))
  (and-gate A B C)
  (or-gate A B D)
  (inverter C E)
  (and-gate D E S))
```

I use DrRacket and SCM to test the program, here is the result of DrRacket:

```racket
> (define in1 (make-wire))
> (define in2 (make-wire))
> (define out1 (make-wire))
> (define cout1 (make-wire))
> (probe 'in1 in1)
in1  New-value = 0
> (probe 'in2 in2)
in2  New-value = 0
> (probe 'out1 out1)
out1  New-value = 0
> (probe 'cout1 cout1)
cout1  New-value = 0
> (half-adder in1 in2 out1 cout1)
'ok
> (set-signal! in1 1)
out1  New-value = 1
in1  New-value = 1
'done
> (set-signal! in2 1)
out1  New-value = 0
cout1  New-value = 1
in2  New-value = 1
'done
```

# Time Line

We finished the event-driven part. Now I believe you have a basical idea of the program architecture. Now move to another question, the difficulty raised, if we add a delay to all the gates, and want to see both state and time line of the change, how to make it?

We need to import a new concept into our program, a time queue. By this case, it is **The Agenda**.

## The Agenda

The agenda is actually a queue, every element in this queue contains a time stamp and the task. 

We can easily wrote down queue structure in scheme:

```scheme
;;;This file shows how to present queue in scheme (message-passing style)

(define nil '())

(define (empty-queue? queue)
  ((queue 'empty)))
(define (front-queue queue)
  ((queue 'front)))
(define (insert-queue! queue item)
  ((queue 'insert) item))
(define (delete-queue! queue)
  ((queue 'delete)))
(define (print-queue queue)
  ((queue 'print)))

(define (make-queue)
  (let ((front-ptr nil)
        (rear-ptr nil))
    (define (empty-queue?)
      (eq? front-ptr nil))
    (define (front-queue)
      (if (empty-queue?)
          (error "ERROR front-queue called with an empty queue")
          (car front-ptr)))
    (define (set-front-ptr! pair)
      (set! front-ptr pair))
    (define (set-rear-ptr! pair)
      (set! rear-ptr pair))
    (define (insert-queue! item)
      (cond ((empty-queue?)
             (set-front-ptr! (cons item nil))
             (set-rear-ptr! front-ptr)
             front-ptr)
            (else (set-cdr! rear-ptr (cons item nil))
                  (set-rear-ptr! (cdr rear-ptr))
                  front-ptr)))
    (define (delete-queue!)
      (cond ((empty-queue?)
             (error "ERROR delete-queue called with an empty queue"))
            (else (set-front-ptr! (cdr front-ptr))
                  front-ptr)))
    (define (print-queue)
      front-ptr)

    (define (dispatch m)
      (cond ((eq? m 'empty) empty-queue?)
            ((eq? m 'front) front-queue)
            ((eq? m 'insert) insert-queue!)
            ((eq? m 'delete) delete-queue!)
            ((eq? m 'print) print-queue)
            (else (error "Unknown queue operation"))))
    dispatch))
```

### Implementing the agenda

Agenda is a list of *time segments*. Each time segment is a pair of time and a queue (holds the procedures to be run at that time).

Recall the signal process of each node, nodes are registered with events, we can insert a small piece of code in each event to push a *time segment* behind the agenda queue, so after all events terminate, by checking the agenda we know when these changes happends.

The problem is when we implement the agenda, we only know the delay of gates. Apropos the delay sequence and the exact time of each event, we have to accumulate the former delays and this elicits a formal problem that we have to maintain a current time variable, and `set!` it every time when we push a time segment in the agenda structure.

First we implememt the agenda structure:

```scheme
(define (make-agenda) (list 0))
(define (current-time agenda) (car agenda))
(define (set-current-time! agenda time)
(set-car! agenda time))
(define (segments agenda) (cdr agenda))
(define (set-segments! agenda segments)
(set-cdr! agenda segments))
(define (first-segment agenda) (car (segments agenda)))
(define (rest-segments agenda) (cdr (segments agenda)))
(define (empty-agenda? agenda)
  (null? (segments agenda)))
```

To add an action to an agenda, we first check if the agenda is empty. If so, we create a time segment for the action and install this in the agenda. Otherwise, we scan the agenda, examining the time of each segment. If we find a segment for our appointed time, we add the action to the associated queue. If we reach a time later than the one to which we are
appointed, we insert a new time segment into the agenda just before it. If we reach the end of the agenda, we must ceate a new time segment at the end.

```scheme
(define (add-to-agenda! time action agenda)
  (define (belongs-before? segments)
    (or (null? segments)
        (< time (segment-time (car segments)))))
  (define (make-new-time-segment time action)
    (let ((q (make-queue)))
      (insert-queue! q action)
      (make-time-segment time q)))
  (define (add-to-segments! segments)
    (if (= (segment-time (car segments)) time)
        (insert-queue! (segment-queue (car segments))
                       action)
        (let ((rest (cdr segments)))
          (if (belongs-before? rest)
              (set-cdr!
               segments
               (cons (make-new-time-segment time action)
                     (cdr segments)))
              (add-to-segments! rest)))))
  (let ((segments (segments agenda)))
    (if (belongs-before? segments)
        (set-segments!
         agenda
         (cons (make-new-time-segment time action)
               segments))
        (add-to-segments! segments))))

(define (remove-first-agenda-item! agenda)
  (let ((q (segment-queue (first-segment agenda))))
    (delete-queue! q)
    (if (empty-queue? q)
        (set-segments! agenda (rest-segments agenda)))))

(define (first-agenda-item agenda)
  (if (empty-agenda? agenda)
      (error "Agenda is empty: FIRST-AGENDA-ITEM")
      (let ((first-seg (first-segment agenda)))
        (set-current-time! agenda
                           (segment-time first-seg))
        (front-queue (segment-queue first-seg)))))
```
[1]: /img/2019-4-4-Event-Driven/HA.png
