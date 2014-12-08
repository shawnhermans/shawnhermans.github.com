---
layout: post
title: "Banking without the Banks: A Bitcoin Inspired Approach to Crypto-Credit"

tags: [bitcoin, banking, fractional reserve system, credit]
---

## A Brief Introduction to Credit
Unless you are independently wealthy or live in some sort of moneyless commune, you have 
probably needed to borrow money at some time in your life.  Most likely you have taken out a student loan, 
a mortgage, a car loan, or a credit card.  In most modern, capitalistic nations the majority of this 
credit is in the form of credit from private institutions such as banks. 

Banks serve as a middleman to extend credit from those with money to those who need the money. 
People give their money to the banks and the banks lend out that money to borrowers.  The 
banks access the credit worthiness of those borrowers in order to determine if a borrow is likely to repay their 
debt.  The borrow pays back the bank the money with associated interest.  The bank may along some of that interest
to the original lender of the money depending on how the lender deposited that money.  

Banks typically use credit scores to evaluate the credit worthiness of a potential borrower.  
[Credit scores in the United States](http://en.wikipedia.org/wiki/Credit_score_in_the_United_States)
are based upon the following factors.  

* Payment history
* Length of credit history
* Debt burden
* Recent credit request
* Types of credit

Additionally, lenders often talk about [six C's of credit](http://en.wikipedia.org/wiki/Credit_\(finance\)). 

* Character
* Capacity
* Collateral
* Conditions
* Credit History
* Capital 

## Credit without the Banks
In our current banking system, the banks are the ones who get to decide who to lend money to and at 
what rate of interest.  They do this by taking other people's money and looking through your detailed financial history. 
All of this wouldn't be so bad if the banks were doing their job of properly accessing credit worthiness, but the 
subprime mortgage crisis of 2007-2009 demonstrated this wasn't the case. 

Is it possible to create the Bitcoin equivalent of a credit market? This market would be one where lenders
would be able to lend money to borrowers without the need for centralized banking infrastructure. If this type
of distributed credit market is possible, what would it look like? 

[Kickstarter](https://www.kickstarter.com/) and similar services provide one model for this type of credit market.  
In this type of model individual lenders and individual borrowers are directly put into contact with one another. 
Lenders choose on a case by case basis whether or not to loan money.  Apparently, 
[BTCjam](https://btcjam.com/how_it_works/overview) offers this type of service for Bitcoin based lending. 

This type of model has many issues.  First, it requires the lender to have knowledge of specific borrowers and 
their particular needs. These may be okay for big, crowd funded projects, but not okay for someone who is borrowing 
money to pay their medical bills. Second, the lenders do not have any reliable guarantee of return of their money.  
A perfect solution would be one that gives borrowers a degree of privacy while giving lenders a reliable
rate of return. 

## Goals
As a thought experiment, I started to think of what a credit system without banks would look like.  
While I am trying to imagine something that is technically feasible, this isn't meant to be a 
technical specification.  

* Decentralized - Eliminate any central points of failure in the system.  "Too big to fail" shouldn't be an option. 
* Anonymous - A person's credit history should be private and only be accessible if they want it to be.  
Lenders should not need access to this detailed credit history. Instead, they should only provided with anonymized data
on potential borrowers.  
* Distributed Risk - One of the functions of modern banks is to distribute the risk associated with non-payment across 
a larger pool of borrowers.  This system should also ...
* Diverse - 
* Antifragile?

## Architecture of Distributed Credit Market
With these goals in mind, I set out to create an architecture that supports these goals.  
I am glossing over a lot of the technical implementation details for the time being in order to focus on the 
core concepts.  I am hoping to get to some of the technical details at a later time as time allows.  

*Lenders*
provide money to a *credit pool* for a fixed amount of time.  After the end of this period of time, the 
credit pool pays the lender a sum of money according to some previously agreed upon *repayment algorithm*. 
The credit pool contains multiple *credit brokers*.  
Credit brokers take money from lenders and 
distribute it to borrowers according to an agreed upon an 
*anonymous credit score* computed by a *scoring service* using 
*scoring algorithm* that depend upon the borrower's *distributed credit history*.  
The credit pools then use one or more of these scores to make their lending decision.

A PRETTY DIAGRAM WOULD BE NICE HERE

A credit pool is a collection of credit brokers, borrowers, and lenders that agree upon repayment
and borrowing algorithms. The algorithms themselves are openly available in order that lenders, brokers, 
and borrowers all have the proper information to make their decisions. It is expected each broker will 
collect a small transaction fee for processing the credit transaction. 

The credit pool uses the escrow account concept described in 
the [contracts section](https://bitcoin.org/en/developer-guide#contracts) on the Bitcoin developer page. 
In order to prevent any one credit broker from committing fraudulent transactions, all transactions must have 
*m* out of *n* brokers agreeing to the transaction.  In a small credit pool this could be 2 out of 3.  Requiring 
more brokers to agree increases the costs of processing transactions, while decreasing the risk of fraudulent 
activity.  

The credit brokers share common lending algorithms.  In order for a transaction to take place, *m* out of *n*
brokers must compute the same value.  This makes the entire system deterministic, amenable to analysis,
and reduces the risk of someone gaming the system.  

The repayment and borrowing algorithms determine the amount of money owed lenders at the end of the maturation period
and the amount of money a borrower is allowed to borrow.  The amount available to borrow and the amount available
for repayment is limited by the total amount of money in the credit pool and avoids the need for 
[fractional reserve banking](http://en.wikipedia.org/wiki/Fractional_reserve_banking).  The borrowing algorithm 
is dependent upon the funds currently available in the credit pool and the credit history of the borrower.  

The distributed credit history is a collection of all transactions within a particular credit market.  It captures
the requests for credit, amounts borrowed, the amounts repaid, and any non-payment actions.  The distributed 
credit history, along with the current amount of money in the credit pool are the determining factors of how much,
if anything, to lend to a potential borrower.  

While an indivdual's 
*m* brokers sign each credit event. This credit event is encrypted using the public key of the user provided.  

## Bootstrapping the Market
One of the biggest issues with this type of credit market is that no one would lend any money because no one would
have any credit history.  

Maybe could use a network-of-trust approach similar to GPG. 

## How would this look? 
More than anything, this post is a thought experiment.  
Instead of three credit rating companies with a triopoly on the credit rating market, there would be a number of 
credit markets to choose from.  


## Digital Insurance
? 

## Issues
There is a lot of hand-waving going on here. 
Bootstraping
One of the big issues with this proposal is the volatility of the Bitcoin prices. 
http://en.wikipedia.org/wiki/Sticky_%28economics%29

A potential solution to this issue is to start with short term loans of m
http://www.coindesk.com/price/#2013-07-21,2014-07-21,close,bpi,USD

## Proof of concept
This needs some more technical details


