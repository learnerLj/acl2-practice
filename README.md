# Configure the environment
>= ubuntu 20.04
```bash
sudo bash init-env.sh
```

your directory looks like this:
```
.
├── README.md
├── acl2
├── init-env.sh
├── sbcl-2.4.4-x86-64-linux
└── sbcl.tar.bz2
```

# Formal language example
GossipeSub protocol modeling is in `acl2/tree/master/books/workshops/2023/kumar-etal`
tutorial: https://www.cs.utexas.edu/users/moore/acl2/v8-5/combined-manual/index.html?topic=ACL2____ACL2-TUTORIAL


## Eclipse Modeling

see code `acl2/books/workshops/2023/kumar-etal/attacks/eclipse.lisp`

An Eclipse attack in a peer-to-peer network involves isolating a target peer (victim) from the rest of the network by overwhelming it with connections from malicious nodes. This prevents the victim from receiving or sending legitimate messages.

The code provided for the ACL2 theorem prover illustrates the implementation of an Eclipse attack on a peer-to-peer network, specifically tailored for the Ethereum 2.0 (Eth2.0) network. Here's a breakdown of the key components and steps involved in defining the Eclipse attack:

1. Attack Setup: The setup includes defining the constants and parameters for the network, weights for gossip topics, and the total weight parameters (TWP) for the Eth2.0 network.
2. Simulation: The simulation involves initializing a group of peers and emitting events that simulate the Eclipse attack. The victim peer (P40) is surrounded by attacker peers (P162, P226, P45), and events are generated to isolate the victim.
3. Violation Detection: The `eclipse-attack-violations` function checks for violations in the network caused by the attack, ensuring that the simulation accurately reflects the attack scenario.

### Setup

This ACL2 code provides a detailed and structured approach to defining and simulating an Eclipse attack on a peer-to-peer network, specifically tailored for the Ethereum 2.0 gossip protocol.

1. Package and Book Inclusion:

Lisp uses packages to organize code into namespaces, preventing name clashes and managing dependencies. 

   ```lisp
   (in-package "ACL2S")
   (include-book "attack")
   ```
`in-package`: This function sets the current package. The ACL2S package is likely a specific namespace within ACL2 for certain kinds of definitions and theorems.
`include-book`: This function includes a book, which is a collection of pre-defined functions, theorems, and proofs. In this case, the attack book is being included.

2. Defining Topics:
   ```lisp
   (definec alltopics-ecl () :lot
     '(AGG BLOCKS SUB1 SUB2 SUB3 SUB4 SUB5 SUB6 SUB7 SUB8 SUB9 SUB10 SUB11 SUB12))
   ```

`definec`: This macro defines a function or constant. The first argument is the name (alltopics-ecl), the second is the argument list (empty here, indicating it's a constant), the third is the type (:lot, which stands for list of topics), and the fourth is the value (a list of topics).
List notation: The list '(AGG BLOCKS SUB1 SUB2 ...) is a quoted list of symbols. The quote (') prevents the list from being evaluated, treating it as a literal.

3. Constants for Gossip Parameters:
   These constants are based on the gossip scoring parameters of the Eth2.0 network.
   ```lisp
   (defconst *decayToZero-ecl* 1/100)
   (defconst *seconds-per-slot-ecl* 1)
   (defconst *one-epoch-duration-ecl* 1)
   (defconst *aggregateWeight-ecl* (/ 1 2))
   (defconst *beaconBlockWeight-ecl* (/ 8 10))
   (defconst *slot-duration-ecl* (* *seconds-per-slot-ecl* 1))
   (defconst *slots-per-epoch-ecl* 10)
   (defconst *blocks-per-epoch-ecl* *slots-per-epoch-ecl*)
   (defconst *decay-epoch-ecl* 5)
   ```

`defconst`: This macro defines a named constant. The name of the constant is usually in asterisks (*) to indicate that it is a global constant.
Arithmetic operations: Lisp supports standard arithmetic operations such as division (/), multiplication (*), and addition. For example:
(/ 1 2) divides 1 by 2.
(* *seconds-per-slot-ecl* 1) multiplies the value of *seconds-per-slot-ecl* by 1.
Referencing constants: Constants are referenced by their names, and operations can be performed using their values.


### Weight and Parameter Definitions

4. Weights and Parameters for Block, Aggregate, and Subnet Topics:
   These constants define the weights and parameters used in the gossip scoring mechanism.
   ```lisp
   (defconst *eth-default-block-weights-ecl* (weights ...))
   (defconst *eth-default-agg-weights-ecl* (weights ...))
   (defconst *eth-default-agg-subnet-weights-ecl* (weights ...))
   (defconst *eth-default-block-params-ecl* (params ...))
   (defconst *eth-default-aggregate-params-ecl* (params ...))
   (defconst *eth-default-aggregate-subnet-params-ecl* (params ...))
   ```

### Configuration for Attack

5. Total Weight Parameters (TWP) for Eth2.0:
   Defines the mapping of topics to their respective weights and parameters.
   ```lisp
   (defconst *eth-twp-ecl*
     `((AGG . (,*eth-default-agg-weights-ecl* . ,*eth-default-aggregate-params-ecl*))
       (BLOCKS . (,*eth-default-block-weights-ecl* . ,*eth-default-block-params-ecl*))
       (SUB1 . (,*eth-default-agg-subnet-weights-ecl* . ,*eth-default-aggregate-subnet-params-ecl*))
       ...
       (SUB12 . (,*eth-default-agg-subnet-weights-ecl* . ,*eth-default-aggregate-subnet-params-ecl*))
     ))
   ```

### Eclipse Attack Implementation

6. Eclipse Attack Simulation:
   The actual simulation of the Eclipse attack is performed using the `emit-evnts-eclipse` and `eclipse-attack-violations` functions. This part defines the attack scenario where a victim peer (P40) is targeted by the attack.
   ```lisp
   (time$
    (b* ((ticks 100)
         (pats '(P162 P226 P45))
         (attacktopics '(SUB1 SUB2 SUB3))
         (grp (reduce* graph->group 
                       (initialize-group-of-meshpeers '()
                                                      '()
                                                      (topics)
                                                      100)
                       *ropsten*
                       (alltopics-ecl)))
         (evnts (emit-evnts-eclipse pats pats 'P40 (alltopics-ecl)
                                    attacktopics 20 0 18 ticks)))
      (eclipse-attack-violations grp evnts evnts pats attacktopics 'P40 0
                                 *eth-twp-ecl* 42 nil)))
   ```

- `time$`: This macro is used to measure the time taken for the simulation to run.
- `b*`: This is a binding macro that binds multiple variables at once.
- `ticks`: The total number of time units for the simulation.
- `pats`: The list of peers participating in the attack.
- `attacktopics`: The list of topics under attack.
- `grp`: The group of peers in the network, initialized and modified by the reduce* function.
    - `initialize-group-of-meshpeers`: Initializes a group of mesh peers with given parameters.
    - `reduce*`: Applies a function to elements of a list, accumulating the results.
    - `emit-evnts-eclipse`: Generates events that simulate the Eclipse attack.
    - `eclipse-attack-violations`: Checks for violations in the network caused by the attack.
- `evnts`: The events generated during the Eclipse attack.


### Run the Eclipse Attack
The `emit-evnts-eclipse` function is designed to emit attack events from a list of attacking peers to a victim peer, across specified topics. The function recursively generates events until the specified number of ticks have elapsed.


```lisp
(skip-proofs
 ;; emit attack events from p1 to p2, in the attacked topic top
 (definecd emit-evnts-eclipse (pats :lop pats2 :lop p2 :peer ts ats :lot n m elapsed
                                    ticks :nat)
   :loev
   (match pats
     (() (app `((,p2 HBM ,elapsed))
              (emit-evnts-eclipse pats2 pats2 p2 ts ats n m elapsed (- ticks elapsed))))
     ((p . rst) (app (emit-meshmsgdeliveries-peer-topics p p2 (set-difference-equal ts ats) n)
                     (emit-meshmsgdeliveries-peer-topics p p2 ats m)
                     (if (<= (- ticks elapsed) 0)
                         nil
                       (emit-evnts-eclipse rst pats2 p2 ts ats n m elapsed ticks)))))))

```
Function Definition:

- `definecd`: Macro for defining certified functions in ACL2.
- pats: List of attacker peers.
- pats2: A copy of the list of attacker peers (used for recursive calls).
- p2: The victim peer.
- ts: List of all topics.
- ats: List of attack topics.
- n: Number of messages for non-attack topics.
- m: Number of messages for attack topics.
- elapsed: The current elapsed time.
- ticks: Total number of ticks for the simulation.

**Base Case:**

```lisp
(() (app `((,p2 HBM ,elapsed))
         (emit-evnts-eclipse pats2 pats2 p2 ts ats n m elapsed (- ticks elapsed))))
```
When the list of attackers (pats) is empty, it appends a Heartbeat Message (HBM) event for the victim peer (p2) at the current elapsed time.
It then recursively calls emit-evnts-eclipse with the original list of attackers (pats2), reducing the remaining ticks by the elapsed time.

**Recursive Case:**

```lisp
((p . rst) (app (emit-meshmsgdeliveries-peer-topics p p2 (set-difference-equal ts ats) n)
                (emit-meshmsgdeliveries-peer-topics p p2 ats m)
                (if (<= (- ticks elapsed) 0)
                    nil
                  (emit-evnts-eclipse rst pats2 p2 ts ats n m elapsed ticks))))
```

- For each attacker p, it emits messages to the victim (p2) for both non-attack topics and attack topics.
- `emit-meshmsgdeliveries-peer-topics`: Function that emits messages from peer p to peer p2 on the specified topics.
- For non-attack topics: emit-meshmsgdeliveries-peer-topics p p2 (set-difference-equal ts ats) n
- For attack topics: emit-meshmsgdeliveries-peer-topics p p2 ats m

The function then checks if the remaining ticks minus the elapsed time is less than or equal to 0.
If yes, it stops the recursion (returns nil).
Otherwise, it recursively calls emit-evnts-eclipse with the rest of the attacker peers (rst), maintaining the original list of attackers (pats2), and the current elapsed time.

## Message Diliver Model

```lisp
(skip-proofs
 ;; emit messages from peer p1 to peer p2 on the specified topics
 (definecd emit-meshmsgdeliveries-peer-topics (p1 :peer p2 :peer topics :lot num-messages :nat)
   :loev
   (if (zp num-messages)
       nil
     (match topics
       (() nil)
       ((topic . rest-topics)
        (app `((,p1 ,p2 ,topic))
             (emit-meshmsgdeliveries-peer-topics p1 p2 rest-topics (1- num-messages)))))))
```
Parameters:
- p1: The sender peer.
- p2: The receiver peer.
- topics: List of topics for which messages are being delivered.
- num-messages: Number of messages to be delivered.

**Base Case:**

If num-messages is zero: The function returns nil, indicating no more messages to send.
If topics is empty: The function returns nil, indicating no more topics to process.

**Recursive Case**

For each topic, emit a message from p1 to p2 for the current topic.
Append this message to the result of recursively calling emit-meshmsgdeliveries-peer-topics with the rest of the topics and one less message to send.