# 50.043 Introduction to Database System

## Learning Outcomes

By the end of this unit, you should be able to

1. List the problems handled by database management systems.
2. Describe the techniques used in database system to solve these problems.

## What is a database? 
Generally speaking, a database is an organized collection of data. 

For instance, 

* your excel sheet that keep tracks of your monthly finance.
* a text file that stores the list of items you want to buy for christmas or CNY.
* a collection of student records in an LMS.
* a collection of product information and stock level for a minimart chain.

## What is a database management system?
It is a system (hopefully a software system) that manages databases. Many DBMS orchestrates and manages multiple databases simultanenously. For instance, Oracle, MySQL, MS SQL, MongoDB, Firebase, Amazon RDs and etc. 

It is confusing when people use to the term "database" to refer to a DBMS. 


## Characteristics of a database management system

For a DBMS to be pragmatically useful, here are some of the characteristics that it may possess

1. It should be efficient. Queries and data operation should be evaluated and performed in an efficient way.
2. It should be crash consistent. In the event of unexpected crash and system distruption, the data stored should remain consistent.
3. It should support concurrent access. There are many users and softwares that could access the data without interfering each other.
4. It should support data abstraction. Data and their relationships are allowed to be described in a logical manner without the users / software to be exposed with the detail implementation.

## Common Techniques available in database management systems

1. Storage and Index. DBMSes have their own internal storage system. It often differs from those available in the operating system, due to use case difference. Internal storage system enables DBMSes to organize and retrieve data in a systematic and efficient way. Indices are inspired from data structure and algorithms. With indices, data queries and operation can be further optimized. 
2. Transaction. Many DBMSes support transactional operations. Trasaction groups multiple operations to be performed as an atomic sequence of actions to be performed. Conflicts caused by concurrent access can be resolved by studying and manipulating multiple transactions. Transactions are logged, so that in the event of system crashes, partial data changes can be reverted back to the nearest consistent state.
3. Data Model and SQL. Many tranditional DBMSes support data model and SQL. This combination is a simple and expressive data abstraction for many applications. Softwares developed based on an existing database (system) could be possible migrate to another with marginal technical effort. 

Recently many big data systems extend their support to these techniques and it proves the essentiality of them.


## Overview of Data Modelling

For the first part of this module, we dwell into the details of data modelling. 

![](./images/overview-of-data-modelling.png)

In the upper diagram, we describe the user interaction with the database as blackbox. 

In the lower diagram, we define the processes of data modelling. 

1. Conceptual Model - in this step, we try to capture the user needs from user study, and identify what data the application has.

2. Logical Model - in this step, supported by the user needs identified, we study how data are been accessed, updated and maintained from the business perspective. We design a high level structure of the data storage and the set of possible constraints over the data to enforce varies integrity derived from the business logics. 

3. Physical Model - in this step, we translate the logical model into a specific database (supported by the database management system that has been deployed).



