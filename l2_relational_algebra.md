# 50.043 -  Relational Algebra

## Learning Outcomes

By the end of this unit, you should be able to 

1. Interpret relational algebra terms (queries)
2. Define relational algebra terms to query a relational model


## Relational Algebra 

Given that relational model is a logical model (abstracting away the implementation details), we need something operations defined on the same layout to manupulate the data in a relational model. (So that we don't need to deal with the implementation details yet.)

Like math algebra, Relational Algebra is a way to express data operation through symbols and relation manipulations. 
1. The data manipulation operations are defined in terms of a sequence of operation applications.
2. Each symbolic operator takes one or more relation(s) (or intermediate relations) as operands and produces one result relation.

For example, given a relation of 
Publish(<ins>article_id, book_id, publisher_id</ins>)

The instance of a Publish relation is given

|article_id|book_id|publisher_id|
|---|---|---|
| a1 | b1 | p1 |
| a2 | b1 | p1 |
| a1 | b2 | p2 |

The query "finding all articles that are published by both publishers p1 and p2" can be expressed as the following relational algebra term.

$$
\Pi_{article\_id}(\sigma_{(publisher\_id='p1')}(Publish)) \cap \Pi_{article\_id}(\sigma_{(publisher\_id='p2')}(Publish)) $$


### Selection 
A selection operator, $\sigma_{P}(R)$, takes a logical predicate $P$ and a relation $R$ and returns all the tuple in $R$ that satisfy $P$. 

Note that $P$ can be 
1. a simple predicate such as a equality test $name = "tom" or 
2. a conjunction or disjunction of predicates, e.g. $name = "tom"\ {\tt AND} \ age > 21$.

For example, the following relational algebra expression returns a relation with all tuples from $Publish$ with $publisher\_id$ equals to `p1`.

$$\sigma_{(publisher\_id='p1')}(Publish)$$

### Projection 
A projection operator $\Pi_{A_1,A_2,...}(R)$, takes a set of attribute names $A_1,...,A_n$ and a relation $R$, and returns a relation that contains the data of columns $A_1,...,A_n$ in $R$.

For example, the following expression returns a relation all all $article\_id$s from $Publish$ with $publisher\_id$ equals to `p1`.

$$\Pi_{article\_id}(\sigma_{(publisher\_id='p1')}(Publish))$$


### Intersection
An intersection operation  $R \cap S$ finds all common tuples from $R$ and $S$, assuming $R$ and $S$ sharing a common schema.

We have seen an example of this earlier.

### Union 

A union operation $R \cup S$ returns a relation containing all tuples from $R$ and all thetuples from $S$, assuming $R$ and $S$ sharing a common schema.


### Difference

A difference operation $R - S$ returns a relation containing all tuples from $R$ that are not in $S$, assuming $R$ and $S$ sharing a common schema.

### Cartesian Product

A cartesian product operation $R \times S$ returns a relation containing all possible combination of some tuple from $R$ and some tuple from $S$. (Note that $R$ and $S$ might have different schema.)


For example, consider 

$R(A,B)$ and $S(C,D)$

|A|B|
|---|---|
|a1 |101|
|a2 |102|

|C|D|
|---|---|
|a3 |103|
|a4 |104|

$R\times S$ yields

|R.A|R.B|S.C|S.D|
|---|---|---|---|
|a1 |101|a3 |103|
|a1 |101|a4 |104|
|a2 |102|a3 |103|
|a2 |102|a4 |104|

Cartesian Product is one of the four possible join operators.

Let's discuss the other three.

### Inner Join

The inner join operator $(R \bowtie_{R.A = S.B, R.C=S.D,...} S)$, returns a relation that containing tuples from $R\times S$ that satisfy $R.A = S.B, R.C=S.D,...$.

Let $R(A,B,C)$ and $S(D,E,F)$ be relations.

|A|B|C|
|---|---|---|
|a1 |101|0 |
|a2 |102|1 |
|a3 |103|0 |


|D|E|F|
|---|---|---|
|a3 |103|'a'|
|a1 |107|'b'|
|a5 |105|'c'|

$R \bowtie_{R.A =S.D} S$ produces

|R.A|R.B|R.C|S.D|S.E|S.F|
|---|---|---|---|---|---|
|a1|101|0|a1|107|'b'|
|a3|103|0|a3|103|'a'|

### Natural Join

The natural join operator $(R \bowtie S)$, returns a relation that containing tuples from $R\times S$ that satisfy $R.A = S.A, R.B=S.B,...$, where $A$, $B$, ... are the common attributes between $R$ and $S$.

Note that the common column are merged after natural join.


Let $R(A,B,C)$ and $S(D,E,F)$ be relations.

|A|B|C|
|---|---|---|
|a1 |101|0 |
|a2 |102|1 |
|a3 |103|0 |


|A|E|F|
|---|---|---|
|a3 |103|'a'|
|a1 |107|'b'|
|a5 |105|'c'|

$R \bowtie S$ produces

|R.A|R.B|R.C|S.E|S.F|
|---|---|---|---|---|
|a1|101|0|107|'b'|
|a3|103|0|103|'a'|

### Right outer join

Right outer join $R ⟖_{R.A=S.B} S$ produces a relation containing all entries from the inner join and all the remaining tupel from $S$ which we could not find a match from $R$, whose attributes will be filled up with NULL.

|A|B|C|
|---|---|---|
|a1 |101|0 |
|a2 |102|1 |
|a3 |103|0 |


|D|E|F|
|---|---|---|
|a3 |103|'a'|
|a1 |107|'b'|
|a5 |105|'c'|

$R ⟖_{R.A =S.D} S$ produces

|R.A|R.B|R.C|S.D|S.E|S.F|
|---|---|---|---|---|---|
|a1|101|0|a1|107|'b'|
|a3|103|0|a3|103|'a'|
|NULL|NULL|NULL|a5|105|'c'|

Likewise for left outer join.

### Renaming 

Renaming operation $\rho_{R'(A_1,...,A_n)}(R)$ produces a new relation $R'(A_1,...,A_n)$ containing all tuples from $R$, assuming the $S_R$ and $S_{R'}$ share the same types.

We omit the attribute name $A_1,...,A_n$ when we only want to rename the relation name. Likewise we omit the relation name if we only want to rename the attributes.


### Aggregation

Aggregation operation $_{A_1,...,A_n} \gamma_{F_1(B_1),F_2(B_2),...} (R)$ produces a new relation $R'$ which contains
* $A_1,...,A_n$ as the attribute names to group by
* $F_1(B_1),...,F_m(B_m)$ as the aggregated values 
where
* $F_1, ..., F_m$ are aggregation functions such as `SUM()`, `AVG()`, `MIN()`, `MAX()`, `COUNT()`.
* $A_1, ..., A_n$, $B_1, ..., B_m$ are attributes from $R$.

For example, given $R(A,B,C)$

|A|B|C|
|---|---|---|
|a1 |101|0 |
|a2 |102|1 |
|a3 |103|0 |


$\rho_{(C,CNT)}({}_{C} \gamma_{{\tt COUNT}(B)}(R))$ produces

|C|CNT|
|---|---|
|0  | 2 |
|1  | 1 |

Sometimes we can rewrite the above expression as 
$$_{C} \gamma_{{\tt COUNT}(B)\ {\tt as}\ CNT}(R)$$
 without using the renaming operator


 ### Alternatives

 Alternative to relational algebra, relational calculus is designed to serve a similiar idea. The difference between them is that relational algebra is more procedural (like C, Java and Python) where relational calculus is more declarative (like CSS and SQL). If you are interested to find out more please refer to the text books. Note that relational calculus will not be assessed in this module.