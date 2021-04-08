---
Keywords: DBMS, transaction, tis
Copyright: (C) 2021 Riki Otaki
---

# Dissect: Transactional Informational Systems

## Chapter 3

### Goal 

The motivation of Chapter 3 is to define a few criteria to determine whether a series of transactions which is called a schedule or a history is "correct". There are schedules which should be categorized as wrong such as lost-update, inconsistent read, and dirty read. It is essential that the correctness criteria capture these schedules and avoid them. 

The major correctness criteria introduced in this chapter is the following:
- FSR (Final State Serializability)
- VSR (View Serializability)
- CSR (Conflict Serializability)

This chapter examines the properties of the above criteria respectively.

### How do you define correctness of a schedule?

Correctness of a schedule is defined by the following steps:
1. Define a notion of _equivalence_ for schedules.
2. Define _serializability_ via equivalence to _serial_ history.

- Serial history
  A history $ s $ is serial if for any two transactions $ t_i $ and $t_j$ in it, where $i \neq j$, all operations from $t_i$ are ordered in $s$ before all operations from $t_j$, or vice versa.

$$[S]_{\approx} = {[s]_{\approx}} | s \in S$$
$$ [S]_{\approx} = {[s]_{\approx}} | s \in S $$

- Equivalence
  1. We define equivalence relation $\approx$ on the set of $S$ of all schedules. This gives rise to a decomposition of $S$ into equivalence classes [] according to $\approx$:
  $$ [S]_{\approx} = {[s]_{\approx}} | s \in S $$
  2. Those classes for which a serial schedule can be chosen as the representative will be called _serializable_.
  

### FSR (Final State Serializability)



### VSR (View Serializability)

### CSR (Conflict Serializability)








